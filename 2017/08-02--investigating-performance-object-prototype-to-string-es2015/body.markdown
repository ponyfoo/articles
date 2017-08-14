# Motivation

The performance of the `Object.prototype.toString()` method in Chrome and Node.js has been [under investigation](http://crbug.com/v8/5175) in the past already, because it is used heavily by certain frameworks and libraries to perform type tests. For example, the [AngularJS](https://angularjs.org/) framework uses it to implement various helper functions like [`angular.isDate`](https://github.com/angular/angular.js/blob/464dde8bd12d9be8503678ac5752945661e006a5/src/Angular.js#L616-L630), [`angular.isArrayBuffer`](https://github.com/angular/angular.js/blob/464dde8bd12d9be8503678ac5752945661e006a5/src/Angular.js#L739-L741) and [`angular.isRegExp`](https://github.com/angular/angular.js/blob/464dde8bd12d9be8503678ac5752945661e006a5/src/Angular.js#L680-L689) (among others):

```javascript
/**
 * @ngdoc function
 * @name angular.isDate
 * @module ng
 * @kind function
 *
 * @description
 * Determines if a value is a date.
 *
 * @param {*} value Reference to check.
 * @returns {boolean} True if `value` is a `Date`.
 */
function isDate(value) {
  return toString.call(value) === '[object Date]';
}
```

Also popular libraries like [lodash](https://lodash.com/) and [underscore.js](http://underscorejs.org/) use `Object.prototype.toString()` to implement checks on values, like the [`_.isPlainObject`](https://github.com/lodash/lodash/blob/6cb3460fcefe66cb96e55b82c6febd2153c992cc/isPlainObject.js#L13-L50) or [`_.isDate`](https://github.com/lodash/lodash/blob/6cb3460fcefe66cb96e55b82c6febd2153c992cc/isDate.js#L8-L25) predicates provided by lodash:

```javascript
/**
 * Checks if `value` is classified as a `Date` object.
 *
 * @since 0.1.0
 * @category Lang
 * @param {*} value The value to check.
 * @returns {boolean} Returns `true` if `value` is a date object, else `false`.
 * @example
 *
 * isDate(new Date)
 * // => true
 *
 * isDate('Mon April 23 2012')
 * // => false
 */
function isDate(value) {
  return isObjectLike(value) && baseGetTag(value) == '[object Date]'
}
```

The Mozilla engineers working on the [SpiderMonkey JavaScript engine](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey) also [identified](https://bugzilla.mozilla.org/show_bug.cgi?id=1369042) the `Symbol.toStringTag` lookup in `Object.prototype.toString()` as bottleneck for real-world performance, as part of their [Speedometer investigation](https://bugzilla.mozilla.org/show_bug.cgi?id=1245279). Running just the AngularJS subtest from the [Speedometer](http://browserbench.org/Speedometer) benchmark suite using the internal V8 profiler (enabled by passing `--no-sandbox --js-flags=--prof` as command line flags to Chrome) we can see that a significant portion of the overall time is spent performing the `@@toStringTag` lookup (inside the `GetPropertyStub`) and the `ObjectProtoToString` code, which implements the `Object.prototype.toString()` built-in method:

![Speedometer AngularJS performance profile][2]

[Jan de Mooij](https://twitter.com/jandemooij) from the SpiderMonkey team crafted a simple micro-benchmark to specifically test the performance of `Object.prototype.toString()` on Arrays:

```javascript
function f() {
    var res = "";
    var a = [1, 2, 3];
    var toString = Object.prototype.toString;
    var t = new Date;
    for (var i = 0; i < 5000000; i++) res = toString.call(a);
    print(new Date - t);
    return res;
}
f();
```

In fact, running this simple micro-benchmark using the internal profiler built into V8  (enabled in the `d8` shell via the `--prof` command line flag) already demonstrates the underlying problem: It is completely dominated by the `Symbol.toStringTag` lookup on the `[1,2,3]` array instance. Roughly 73% of the overall execution time is consumed by the negative property lookup (in the `GetPropertyStub` that implements the generic property lookup), and another 3% are wasted in the `ToObject` built-in, which is a no-op in case of arrays (since an Array is already an Object in the JavaScript sense).

![Mozilla micro-benchmark performance profile (before)][3]

# Interesting symbols

The [proposed solution for SpiderMonkey](https://bugzilla.mozilla.org/show_bug.cgi?id=1369042#c0) was to add the notion of an *interesting symbol*, which is a bit on every [hidden class](https://github.com/v8/v8/wiki/Design%20Elements) that says whether instances with this hidden class may have a property whose name is `@@toStringTag` or `@@toPrimitive`. This way the expensive search for `Symbol.toStringTag` can be avoided in the common case, where the lookup is negative anyways, which resulted in a **2x** improvement on the simple micro-benchmark for SpiderMonkey.

Since I was looking specifically into some [AngularJS](https://angularjs.org/) use cases, I was happy to find this idea and see that it works out well. So I started thinking about the [design](https://docs.google.com/document/d/1q_Y2YM8S055RF1R6qvDe65kOEVO99tdviI1vaDcbnmc/edit#) and eventually [ported](https://chromium-review.googlesource.com/c/593620) it to V8, although limited to just `Symbol.toStringTag` and `Object.prototype.toString()` for now, as I haven’t found evidence (yet) that `Symbol.toPrimitive` is a major pain point in Chrome or Node.js. The fundamental idea is that by default we assume that instances don’t have *interesting symbols*, and every time we add a new property to an instance, we check whether that property’s name is an *interesting symbol*, and if so we set the bit on the instances hidden classes.

```javascript
const obj = {};
Object.prototype.toString.call(obj);  // fast-path
obj[Symbol.toStringTag] = 'a';
Object.prototype.toString.call(obj);  // slow-path
```

Check this simple example: Here `obj` starts life as an instance with definitely no *interesting symbols* on it. So the first call to `Object.prototype.toString()` takes the new fast-path, where the `Symbol.toStringTag` lookup can be skipped (also because the `Object.prototype` doesn’t have any *interesting symbols* on it), whereas the second call takes the generic slow-path because `obj` now has an *interesting symbol*.

# Performance

Implementing this mechanism in V8 improves the performance on the above mentioned micro-benchmark by roughly **5.8x** on a Z620 Linux workstation. And checking the performance profile again, we can see that we no longer spend time in the `GetPropertyStub`, but the micro-benchmark is now dominated by the `Object.prototype.toString()` built-in as expected:

![Mozilla micro-benchmark performance profile (after)][4]

Running this on a [slightly more realistic benchmark](https://gist.github.com/bmeurer/cc4a6c97d244eb4c8c0738bd4b8c3319), which passes different values to `Object.prototype.toString()`, including primitives and objects which have a custom `Symbol.toStringTag` property, shows up to **6.5x** improvements in the latest V8 compared to V8 6.1.

![Micro-benchmark results][5]

Measuring the impact on the [Speedometer](http://browserbench.org/Speedometer) browser benchmark, specifically the AngularJS subtest in the benchmark suite, it seems to yield a 1% overall improvement on the full suite and a solid 3% on the AngularJS subtest.

![Speedometer results][6]

# Conclusion

Even a highly optimized built-in like `Object.prototype.toString()` still provides some potential for further optimization - leading up to **6.5x improvements** in throughput - if you dig deep enough into appropriate performance tests (like the Speedometer AngularJS benchmark in this case). Kudos to [Jan de Mooij](https://twitter.com/jandemooij) and [Tom Schuster](http://twitter.com/evilpies) from Mozilla for doing the investigation in this case, and coming up with the cool idea of *interesting symbols*!

It’s worth noting that JavaScriptCore, the JavaScript engine used by [WebKit](https://webkit.org/), caches the result of subsequent `Object.prototype.toString()` calls on the hidden class of the receiver instance (that cache was [introduced in early 2012](https://bugs.webkit.org/show_bug.cgi?id=84781), so it predates ES2015). It's a very interesting strategy, but it has limited applicability (i.e. it doesn’t help with other well-known symbols like [`Symbol.toPrimitive`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive) or [`Symbol.hasInstance`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance)) and requires pretty complex [invalidation logic](https://github.com/WebKit/webkit/blob/29330a72e9d9e8a0fff4ec77c65eb18020695a96/Source/JavaScriptCore/runtime/StructureRareData.cpp#L113-L169) to react to changes in the prototype chain, which is why I decided against a caching based solution in V8 (for now).


  [2]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/angular-before-66fc535bf2b6413889537b23b30dde89.png "angular-before.png"
  [3]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/mozilla-before-a2de4ffc5dbc4c4db5e67b6670afeab0.png "mozilla-before.png"
  [4]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/mozilla-after-9f219c80c8ee4a418d0ff3da0ac91603.png "mozilla-after.png"
  [5]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/results-microbenchmark-7a8e974c49d24e7c9c5b5e22ebfa784f.png "results-microbenchmark.png"
  [6]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/results-speedometer-8d819bdc516c4e71bdf049bd1910369f.png "results-speedometer.png"
