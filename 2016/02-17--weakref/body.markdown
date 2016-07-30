The [Background](#background) and [Intended Audience](#intended-audience) sections, found below, contain good indicators of why the proposal was put forward, as well as the intended audience. They were extracted from the proposal's repository. Afterwards we'll move onto use cases and its API.

# Background

In a simple garbage-collected language, memory is a graph of references between objects, anchored in the roots such as global data structures and the program stack. The garbage collector may reclaim any object that is not reachable through that graph from the roots. Some patterns of computation that are important to JavaScript community are difficult to implement in this simple model because their straightforward implementation causes no-longer used objects to still be reachable. This results in storage leaks or requires difficult manual cycle breaking. Examples include:

- MVC and data binding frameworks
- Reactive-style libraries and languages that compile to JS
- Caches that require cleanup after keys are no longer referenced
- Proxies/stubs for comm or object persistence frameworks
- Objects implemented using external or manually-managed resources

Two related enhancements to the language runtime will enable programmers to implement these patterns of computation: weak references and finalization.

> A *strong reference* is a reference that causes an object to be retained; in the simple model all references are strong. A _weak reference_ is a reference that allows access to an object that has not yet been garbage collected, but does not prevent that object from being garbage collected. _Finalization_ is the execution of code to clean up after an object that has become unreachable to program execution.

For example, MVC frameworks often use an observer pattern: the view points at the model, and also registers as an observer of the model. If the view is no longer referenced from the view hierarchy, it should be reclaimable. However in the observer pattern, the model points at its observers, so the model retains the view (even though the view is no longer displayed). By having the model point at its observers using a weak reference, the view can just be garbage collected normally with no complicated reference management code.

Similarly, a graphics widget might have a reference to a primitive external bitmap resource that requires manual cleanup (e.g., it must be returned to a buffer pool when done). With finalization, code can cleanup the bitmap resource when the graphics widget is reclaimed, avoiding the need for pervasive manual disposal discipline across the widget library.

# Intended Audience

The garbage collection challenges addressed here largely arise in the implementation of libraries and frameworks. The features proposed here are advanced features (e.g., like proxies) that are **primarily intended for use by library and framework creators**, not their clients.

> Thus, the priority is enabling library implementors to correctly, efficiently, and securely manage object lifetimes and finalization.

# The API for weak references

When using strongly-held references, cutting off one of them doesn't affect other references.

```js
var a, b;
a = b = document.querySelector('.ponyfoo')
a = undefined
// ... a GC pause later ...
// b still references the DOM element
```

In the diagram below _-- also retrieved from the proposal's repository --_ the target object is the DOM element itself, while the _Client_ and _Service Object_ are references to it. Here, the case is being made for the use case where we have a core "Client" component _(such as the DOM element)_ and one or many "Service Object" components that are only necessary as long as the DOM element is being referenced.

![Diagram without using WeakRef][1]

In the situation we've just described, we'd have to dereference `b` whenever we dereference `a`. In particular, we'd have to keep track of reference counting ourselves, so that we know when the DOM element is no longer being referenced by a relevant party, and then we can dereference the element in _"secondary" _-- that is, "Service" --_ references to it.

Under this proposal, there's a `makeWeakRef` global function and a `WeakRef` built-in class. Using `makeWeakRef` returns a `WeakRef` object that's pointed at `target`. If `target` is no longer referenced, then the `WeakRef` object will stop pointing at it after a full garbage collector sweep.

```js
makeWeakRef(target, executor?, holdings?)
```

The parameters to `makeWeakRef` are as follows:

- `target` is the object we wish to create a weak reference to
- `executor` is an optional argument that can be used as a _finalization callback_
- `holdings` is an optional argument provided to `executor` when it's invoked for `target`

We could create a new `WeakRef` using `makeWeakRef`, as shown below.

```js
var target = document.querySelector('.ponyfoo')
var weakRef = makeWeakRef(target)
```

The `WeakRef` object has two methods. There's `get()`, which retrieves the weakly held `target` object reference or `null` if the object has been garbage collected away. Then there's `.clear()` which nulls out the underlying weak reference _while preventing the `executor` from being invoked_, without the need for a full GC sweep.


The following diagram shows how `WeakRef` can act as an intermediary that deals with reference counting of strongly held references and retrieval of a weakly held reference on our behalf.

![Diagram when using WeakRef][2]

While there's at least one non-weak reference to the DOM element matching `.ponyfoo`, `weakRef.get()` target will return the element. In our case, this means we'll be able to retrieve the weakly held reference to the DOM element for as long as the `target` variable is pointing at it.

```js
weakRef.get() === <mark>target</mark>
```

If we de-referenced `target`, the `WeakRef` interface can no longer guarantee that it'll return the DOM element. After a full GC sweep and finalization, `weakRef.get` will start to return `null`.

```js
target = undefined // de-reference <mark>target</mark>
// after a full GC, weakRef no longer points at <mark>target</mark>
weakRef.get() === <mark>null</mark>
```

Instead of waiting for a GC sweep you could alternatively call `weakRef.clear` to break the weakly held reference. In that case, `weakRef.get` would immediately begin to return `null`.

# Use Cases

Whenever you previously wanted to use a [`WeakMap`][3] just so you could map a DOM element to an object that provided extra features related to it, you could now use `WeakRef`. Similarly, in every case where you want to provide extra features around an object you don't have control over, a `WeakRef` is a great way to do so without holding onto it with a strongly-held reference that prevents it from being garbage collected. Something that, under certain circumstances, could end up in a **hard-to-detect memory leak** situation.

[1]: https://i.imgur.com/ly0xhbt.png
[2]: https://i.imgur.com/PUv49zn.png
[3]: /articles/es6-weakmaps-sets-and-weaksets-in-depth "ES6 WeakMaps, Sets, and WeakSets in Depth"
