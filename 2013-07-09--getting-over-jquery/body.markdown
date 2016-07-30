![jquery.jpg][jquery]

If we look at their [API documentation](http://api.jquery.com/ "jQuery API Documentation"), we can _quickly categorize_ the features we use most frequently.

- AJAX
- Attributes, CSS, `.data`
- Effects, Animations
- Events
- DOM Querying, Selectors
- DOM Manipulation
- **Plugins**

That certainly _looks like_ a lot. Lets break it down, and attempt to arrive at the same functionality, but _without jQuery_. Our aim in doing so, isn't just getting rid of jQuery for the sake of doing so, but thinking about why we'd want it _in the first place_. Furthermore, we will be gaining insight into how jQuery operates, what is needed, what is not, and maybe even more importantly, understanding and becoming capable of performing these operations on our own.

## Scope

I previously mentioned the [micro library movement](http://microjs.com "Fantastic Micro Frameworks and Libraries"), which is awesome, too. Here, though, we will _pick a few battles_ of our own, and have a shot at resolving them without resorting to external dependencies. Other than _what browsers provide_, that is.

Keep browser support in mind. In each of my solutions, I'll tell you what the browser support is for that particular approach. I will mostly speak about _future-proof solutions_, but most of what I'll be talking about _probably won't work in IE 6_. So keep an eye on that.

> Even if you are working in a project that must support older browsers, for whatever reason, I think you'll still find value in these excerpts. Maybe they aren't that useful to you _today_, maybe they are. One thing is certain though, _the benefit of learning the underlying browser API won't be going away anytime soon_.

## AJAX

I wanted to give you an update on AJAX. We've already _[somewhat covered](/2013/06/10/uncovering-the-native-dom-api "Uncovering the Native DOM API")_ how to write native requests, but lets take it up a notch.

At this point, I think I should introduce you to [XHR2](http://www.html5rocks.com/en/tutorials/file/xhr2/ "New Tricks in XMLHttpRequest2"). Lets start by talking about [browser support](http://caniuse.com/xhr2 "Can I Use XHR2?"). As you can see, XHR2 support includes anything that's not `IE < 10 || Android < 3.0`. That's _not very encouraging_, but it's workable.

The fun in XHR2 comes from being able to set a `responseType`. Here is a table of possible values, adapted from what can be [found on MDN][xhr].

> | Value           | `response` data type                  |
> |-----------------|---------------------------------------|
> | `'text'`        | `String` (this is the _default_, too) |
> | `'arraybuffer'` | `ArrayBuffer`                         |
> | `'blob'`        | `Blob`                                |
> | `'document'`    | `Document`                            |
> | `'json'`        | JSON `object`                         |
>
> **Note** that the `'json'` value is currently _only supported by Firefox and Opera_. If you want to fetch JSON data in a cross-browser manner, your best bet is setting `responseType = 'text'`, and then parsing the response like so: `JSON.parse(xhr.response)`.

From the resources listed above, we can gather that `Blob` is a great representation if we want to fetch _images, or any other binary file_. `'document'` should be used for XML. `json` of parsed `'text'` for JSON, and `'text'` for _pretty much everything else_.

As far as sending data to the server goes, there are a few options. we could stick to using a simple `String` value.

```js
var xhr = new XMLHttpRequest();
xhr.open('POST', '/api', true);
xhr.onload = function(e){
    if(this.status === 200){
        console.log(this.response);
    }
};
xhr.send('data!');
```

But we're already used to doing _that_. What's new is we can send _form-like_ data using `FormData`.

```js
var formData = new FormData();
formData.append('username', 'carlos');
formData.append('email', 'cslim@geocities.com');
formData.append('dob', 1940);

var xhr = new XMLHttpRequest();
xhr.open('POST', '/register', true);
xhr.onload = function(e){
    if(this.status === 200){
        console.log(this.response);
    }
};
xhr.send(formData);
```

We don't _necessarily_ have to create the `FormData` from scratch, either. Suppose we had a form.

```html
<form id='registration' name='registration' action='/register'>
    <input type='text' name='username' value='carlos'>
    <input type='email' name='email' value='cslim@geocities.com'>
    <input type='number' name='dob' value='1940'>
    <input type='submit' onclick='return sendForm(this.form);'>
</form>
```

Then we could derive our AJAX request data _off of it_.

```js
function sendForm(form) {
    var formData = new FormData(form);
    formData.append('csrf', 'e69a18d7db1286040586e6da1950128c');

    var xhr = new XMLHttpRequest();
    xhr.open('POST', form.action, true);
    xhr.onload = function(e) {
        // ...
    };
    xhr.send(formData);

    return false; // we're already submitting the form through AJAX.
}

var form = document.querySelector('#registration');
sendForm(form);
```

Similarly to responses, `.send()` supports passing `Blob` data, if we need to perform _asynchronous file uploads_.

In older browsers, lots of different methods are used to upload files asynchronously. Flash, `iframe`s, anything goes. Other than file uploads or otherwise _using form data directly_, though, we are just fine doing AJAX in older browsers, as long as we don't pretend to use the **XHR2 API**. This API mostly improves our asynchronous file upload capabilities, but we are otherwise fine without it.

## Attributes, CSS, and `.data`

Not everything has to be as complicated as AJAX is, and `Element` attributes are never _reason enough_ to warrant the inclusion of a heavy-weight library such as jQuery.

Lets look at all of these in turn.

`.attr(name, val)` is just sugar. Once we have an element, presumably obtained using something similar to `document.querySelector('main')`, we can use `.setAttribute(name, val)` to set the attribute, or `getAttribute(name)` to retrieve its value.

`.prop(name, val)` does pretty much the same thing, except there are some parse hooks in place to return booleans or numbers, rather than always returning strings. Which is nice, but _that doesn't justify an enormous footprint_ either.

When it comes to CSS, it pains me to read the source code of jQuery plugins and find out that they set up tens of different styles directly in their JavaScript code, why not use classes instead? That's what they are for! Unless you are writing CSS that depends on the dynamic calculations you are performing in your JS code, there is _no reason not to use a class_, instead.

Once that's out of the picture, we're left with two applications for manipulating classes within JS code: logic to _hide or display_ DOM components, and logic to _add or remove classes_ from our nodes.

You should be ashamed to even think of using jQuery for the former. These would be all you need to type to get that working:

```js
// display
element.style.display = 'block';

// hide
element.style.display = 'none';
```

When it comes to the latter, we can use [classList](https://developer.mozilla.org/en-US/docs/Web/API/element.classList "element.classList - MDN"), which doesn't have [great support](http://caniuse.com/classlist "Can I Use classList?"), or we can simply use `className`. If we find ourselves in need to add or remove classes, then we will have to resort to using [regular expressions](/2013/05/27/learn-regular-expressions "Learn Regular Expressions") to figure out how to remove classes from our elements.

```js
!function(exports){
    var class_list = !!document.body.classList;
    var s = '(\\s|^)'; // space or start
    var e = '(\\s|$)'; // space or end

    function getRegex(className){
        return new RegExp(s + className + e, 'g');
    }

    exports.addClass = function(element, className){
        if(class_list){
            element.classList.add(className);
        }else{
            element.className += ' ' + className;
        }
    };

    exports.removeClass = function(element, className){
        if(class_list){
            element.classList.remove(className);
        }else{
            var rclass = getRegex(className);
            element.className = element.className.replace(rclass, '');
        }
    };

    exports.hasClass = function(element, className){
        if(class_list){
            return element.classList.contains(className);
        }else{
            var rclass = getRegex(className);
            return element.className.match(rclass);
        }
    };
}(window);
```

That wasn't that hard, either. As we are on the subject, let me give you some _added value_, and talk about [getComputedStyle](https://developer.mozilla.org/en-US/docs/Web/API/window.getComputedStyle "window.getComputedStyle - MDN"). Supported in every browser [except](http://caniuse.com/getcomputedstyle "Can I Use getComputedStyle?") for `IE < 9`, `getComputedStyle` returns the resulting value of _applying every style on an element_. The coolest feature of this method, though, is that it enables us to grab the computed _pseudo-element styles. For example, we could grab the `::after` styles on a `<blockquote>` element.

Here you have an example taken from **MDN**:

```html
<style>
    h3:after {
        content: ' rocks!';
    }
</style>

<h3>generated content</h3> 

<script>
    var h3 = document.querySelector('h3');
    var result = getComputedStyle(h3, ':after').content;

    // > ' rocks!'
    console.log('the generated content is: ', result);
</script>
```

Before we move forward, there's _one more attribute accessor_ we might want to talk about. The `.data` API. Similarly to `.prop`, it works by probing the value in your `data-*` attributes, parsing `true`, `false`, numbers, and JSON `object`s, or just returning a `String`. One important difference here, is that _jQuery sets up a cache_ for these values. This helps prevent querying the DOM time and again for stuff that _isn't going to change_. Under the assumption that _we are manipulating data attributes solely through their API_, that is.

Other than that, a simplified data API might look like:

```js
function data(element, name, value){
    if (value === undefined){
        value = element.getAttribute('data-' + name);
        return JSON.parse(value);
    }else{
        element.setAttribute('data-' + name, JSON.stringify(value));
    }
}
```

Keep in mind you might also want to use the [dataset API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement.dataset "Element.dataset - MDN"), but `IE < 11` doesn't support it.

If you were to add a little cache to reduce DOM querying, you'd have your own little awesome `.data` API!

## Effects, Animations

In this category, I'll get straight to the point. We'll want to **use CSS for any kind of animations**. If it's _fading effects_ you are after, then you can resort to [transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/transition "CSS Transitions - MDN"), instead.

When it comes to animations, there is one more option, though. We could use `setInterval` to set up a loop where we animate something, for example, if we want to move an element with absolute positioning all around our viewport.

```js
setInterval(function(){
    // move it a bit
}, delay);
```

I always had problems with `setInterval`. **Personal problems**. You see, the delay you apply as the second argument counts from the moment the function triggers, not the moment the execution ends. As a result, if your function takes `400`, and you've set a delay of `600`, The calls will eventually overlap so much, making a mess of everything. For that reason, I prefer doing _a bit of extra work_.

```js
function loop(fn, interval){
    return setTimeout(function(){
        fn(function(){
            loop(fn, interval);
        });
    }, interval);
}

loop(function(done){
    // do our trick

    done(); // continue our loop
}, 600);
```

The difference is subtle, but now, we can invoke `done` whenever we are done, so our loop will run sequentially, not in parallel, which** makes no sense**. It's supposed to be an _interval_, right?

### Using `requestAnimationFrame`

Enough with the `setInterval` rant. We shouldn't have to use either of these when it comes to animations. A _better option_ is available. Yes, I'm talking about [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window.requestAnimationFrame "requestAnimationFrame - MDN"). No, it has pretty [bad browser support](http://caniuse.com/requestanimationframe "Can I Use requestAnimationFrame?"). Android doesn't support it _at all_. `IE < 10` doesn't care about it either.

`requestAnimationFrame` allows us to perform a `setInterval`-like operation just before every repaint. This method takes a callback, our operation, and passes our callback an argument, with a `timestamp`, so we don't have to make assumptions about the time elapsed.

Here is an usage example, forked from the example on **MDN**.

```js
!function(w, raf) {
    w[raf] = w[raf] ||
             w.mozRequestAnimationFrame ||
             w.webkitRequestAnimationFrame ||
             w.msRequestAnimationFrame;           
}(window, 'requestAnimationFrame');

var start = Date.now();

function step(timestamp) {
    var progress = timestamp - start;
    d.style.left = Math.min(progress / 10, 200) + 'px';
    if (progress < 2000) {
        requestAnimationFrame(step);
    }
}

requestAnimationFrame(step);
```

**Chris Coyier** also provides a few, [nice usage examples](http://css-tricks.com/using-requestanimationframe/ "Using requestAnimationFrame"), on his blog.

## Events

_A lot has improved_ in the jQuery **event API** over time. It used to be all over the place, nowadays we mostly have the `.on` and `.off` methods, and those handle _pretty much everything_ we need.

So what are the strong points for jQuery in event handling? Well, they make it really easy to perform _event delegation_.

```js
$('ul').on('click', 'li', function(){
    console.log('li clicked!', this);
});
```

This seemingly innocent handler will be triggered whenever we click on _any_ `<li>`, yet the event handler will be on the parent `<ul>`. The way this works is that whenever an `<li>` is clicked, the event will [bubble up](http://www.quirksmode.org/js/events_order.html "Event Order in JavaScript") to the `<ul>`. The `<ul>` will have an special handler, provided by jQuery, which will trigger _our_ handler `.apply`ing the `<li>` as `this`.

If you are just realizing _how complicated this is_ to grasp, that's probably because how powerful the abstraction is. The implications of this might not be obvious at a glance, but the end result is that you get a much more performant experience. Rather than setting up an event handler for each `<li>`, which could _potentially be thousands_, you are setting a single `<ul>` event listener instead.

Other than event delegation, their API is once again really easy to implement by hand, and you might want to check out my [previous post](/2013/06/10/uncovering-the-native-dom-api "Uncovering the Native DOM API") on the subject to wrap your head around that.

If you want to try _going native_, a suggested approach consists of **barely two lines of code**.

```js
var $ = document.querySelectorAll.bind(document);
Element.prototype.on = Element.prototype.addEventListener;
```

Once our _ridiculously small library_ is in place, we can attach event handlers using our new API.

```js
$('#featured')[0].on('keyup', handleKeyUp);
```

Concise enough.

## DOM Querying, Selectors

> One of the most important mechanisms in browsers is querying the DOM to obtain a reference to HTML nodes. Yet, `querySelector`, by far the best option to perform such requests, is relatively unknown to the average developer. It's as if they're stuck with either `getElementById`, or _using jQuery_.

Truth is, `querySelector` and `querySelectorAll` are [broadly](http://caniuse.com/queryselector "Can I Use querySelector?") supported in all major browsers, with the exception of `IE < 8`. That is _really good_ browser support. That is, in fact, one of the major reasons jQuery decided to _drop support_ for `IE < 9` in [their v2 branch](http://blog.jquery.com/2013/04/18/jquery-2-0-released/ "jQuery 2.0 Released").

With `querySelector` being implemented across all browsers, the novelty in jQuery is reduced to _the ability to extend the selector engine_ by adding your own, _custom selectors_. This just adds to the confusion and isn't really necessary. _I'd recommend staying away from [that](http://james.padolsey.com/javascript/extending-jquerys-selector-capabilities/ "Extending jQuery's Selector Capabilities")_.

## DOM Manipulation

There isn't a lot left to cover about DOM manipulation that wasn't covered in the other topics we've been discussing. If we look at the [API documentation](http://api.jquery.com/category/manipulation/ "Manipulation - jQuery API Documentation") once again, you'll notice we've accounted for most of the methods in the category. The ones we didn't mention are mostly measure computations, DOM altering methods, or methods such as `.val()`, `.text()` and `.html()`, which _don't really abstract any cross-browser limitations away_.

When it comes to altering the DOM, the native methods can be [found on MDN][dom]. Once we know about those, all jQuery really does is _build on top_ of the `Node` API, providing us with some [syntactic sugar](http://en.wikipedia.org/wiki/Syntactic_sugar "Syntactic Sugar"), such as `insertAfter` does.

## **Plugins**

![plugins.jpg][plugins]

Ah, plugins! Do we really need _everything_ to be a [jQuery plugin](http://net.tutsplus.com/tutorials/javascript-ajax/14-reason-why-nobody-used-your-jquery-plugin/ "14 Reasons Why Nobody Used Your jQuery Plugin")? I get _ecstatic_ whenever I find a small library, which performs its intended objectives really well, has a _succint API_, and **doesn't freaking depend on jQuery for _absolutely no reason_**.

I guess my point is, make it a _conscious decision_. Don't mindlessly turn your _ten line miracle worker_ into a jQuery plugin just because you want to use `.hide()` and `.show()`. Write native code instead. You'll probably learn to _write better code_ while at it, and more people will be able to use it, _to boot_.

> Oh, and **stay the hell away from [jQuery UI](http://jqueryui.com "jQuery User Interface")**, too. _Thank you_.

Unless you are really _using it extensively_. If you only need the dialogs, you can get away with [just a few lines](http://raventools.com/blog/create-a-modal-dialog-using-css-and-javascript/ "Create a Modal Dialog Using CSS and JavaScript") of CSS code!

##### Need a Talk?

Below is an excellent talk on jQuery, by [Remy Sharp](http://remysharp.com/ "Remy Sharp's Blog"). He addresses a lot of important points, and raises some very good questions. He also presents a minimal library called [min.js](https://github.com/remy/min.js "min.js on GitHub"), which I think shows _a lot_ of promise. In this half hour _ish_ talk, you'll learn how you can actually write native BOM pretty effortlessly, without having to resort to a jQuery-like library.

[![remy-on-jquery](https://i.imgur.com/nORxT86.jpg)](http://vimeo.com/68910118 "So you know jQuery. Now what?")

##### In Conclusion

> I don't expect you to _shelf_ jQuery right away. I'm just attempting to enlighten you, _there is another way to do things_. jQuery is great and all, but it's been around for _almost ten years_, so it's _understandable_ that it lost some value along the way. It is good if you are actually using many of its features, but _you should ponder_ about whether this is a fact for you, or if you are simply using it because, _hey, it's already there_.

And it's not _jQuery's fault_, but rather, we should be _complimenting the browsers_ for this change. Going forward, IE11 is finally [putting an end](http://www.nczonline.net/blog/2013/07/02/internet-explorer-11-dont-call-me-ie/ "Internet Explorer 11: Don't call me IE") to all the non-sense set forth by it's predecessors. They're really trying hard this time to set it apart from "old IE" distributions.

Now that all major browsers offer automatic updates, jQuery will _steadily decline in value_. The ultimate purpose of the library, dealing with the **multitude of cross browser issues** present in older browsers, is _subsiding_. In its current state, jQuery will eventually become a library that just provides a somewhat nicer API than native browser JavaScript does.

> If you think there is a topic I didn't uncover, please _let me know_, and I'll consider it for a future blog post.

**Happy experimenting!**

  [jquery]: https://i.imgur.com/8wWcU19.jpg "jQuery"
  [plugins]: https://i.imgur.com/rl2URLW.jpg "Sad, sad plugins"
  [xhr]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest#responseType "responseType values - MDN"
  [dom]: https://developer.mozilla.org/en-US/docs/Web/API/Node#Methods "Node Methods - MDN"
