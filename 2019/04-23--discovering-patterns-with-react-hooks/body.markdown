##  Pattern extinction level event

Discovering new patterns is relatively rare when working with established technologies, but every now and then a new idea comes up that stirs things up within some ecosystem. Enter React hooks. Hooks enable a drastically different way to build stateful components in React, which means that they also invalidate [^invalidated] a lot of the established patterns. However, being an exceptionally well thought-through and composition-friendly abstraction, hooks enabled new, elegant patterns to emerge.

[^invalidated]: Of course, hooks being backward-compatible, you can still use old patterns in React if you stick to the class component API.

Over the last few months, I've been building apps with hooks and I've been having a lot of fun discovering these patterns. I'd like to share one, that I've grown especially fond of. For the lack of a better name, and for sole the purpose of this blog post, let's call it the Choco🍫 pattern (from Context, HOok, COpomonent). The pattern comes useful when you need a UI widget which is top-level, centralised and unique across the application and needs to expose some functionality to the rest of the components in the app. It might sound vague at the moment, so let's try to go through solving a concrete problem where I noticed the Choco pattern manifest itself.

## Exhibit A: Feature toggles 

Let's try to build a simple feature toggle functionality for a React app. There should be UI where the user can toggle features on and off and we want to expose these settings to any component interested. We can identify parts we will need: 

1) a context provider exposing the feature toggles anywhere in the component tree,
2) a component rendering the UI for toggling features, 
3) a place to store the feature toggle state and the associated logic.

Our first attempt could look like this:

<iframe
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  src="https://glitch.com/embed/#!/embed/hooks-feature-toggles?path=script.jsx&previewSize=0"
  alt="hooks-feature-toggles on Glitch"
  height="417px"
  width="100%">
</iframe>

This code is fairly simple and works so we could stop here. However, there is a big opportunity for refactoring, by moving all the code related to the "feature toggle concern" to a separate module, instead of mixing it with the rest of our app. Let's do just that.

<iframe
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  src="https://glitch.com/embed/#!/embed/hooks-feature-toggles-2?path=feature-toggle.jsx&previewSize=0"
  alt="hooks-feature-toggles-2 on Glitch"
  width="100%"
  height="431px">
</iframe>

The biggest change here is extracting the state and its related logic into a custom hook. We can now clearly see the Choco pattern here, with the Context, HOok, and COpomonent exported from the new module. 

I made a choice here to keep the `initialFeatures` dictionary with the application code instead of moving it to the newly created module. One could argue either way, but by keeping that dictionary out of the feature toggle module and dependency-injecting it, we make that module completely generic and a candidate for possible extraction into a library.

Again, we could easily end the refactoring here. In fact, that's the level of decoupling I'd recommend for in this scenario (more on that later). We can, however, take it one step further and introduce an abstraction to hide some of the implementation details and reduce the API surface of the module. Here is one way to do this:

<iframe
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  src="https://glitch.com/embed/#!/embed/hooks-feature-toggles-3?path=feature-toggle.jsx&previewSize=0"
  alt="hooks-feature-toggles-3 on Glitch"
  width="100%"
  height="431px">
</iframe>

The biggest difference to notice is that the fact we're using the native React context is now an implementation detail, not exposed to the rest of the app. By adding our own abstraction on top of it we also gained a more fine-grain control over what properties of the context are exposed. In this case, we could expose only the feature toggle dictionary, while keeping the function to toggle them private.

Another thing to notice is that by wrapping the context provider in a component, we had an opportunity to render the `FeatureToggler` in that component and take that responsibility away from the module's consumer. This might be a viable option if we decide to e.g. render the toggler in a portal to make it overlay the page after the user presses some specific key combination. However, in this example I chose not to - this way is much more flexible as the consumer can choose where and how to render the toggler.

What's interesting is that we kept the same type of exports from the feature toggle module (Context, HOok, and COpomonent), but they are consumed in a different way. The hook is now taking care of exposing the features to the components. The context (or more precisely the wrapped context provider) now also encapsulates the state and related logic. 

Did we improve this code in comparison to the previous iteration? I think the answer depends on how we plan to use it. If we decided to extract the feature toggle module into a small library, then that last version of the code is most likely a better choice. It would be easier to document and use the API with a smaller exposed surface. Hiding the implementation details might also enable adding more functionality later on, without introducing breaking changes. 

However, if we keep it as a module within the codebase the abstraction creates an unnecessary indirection that the next developer would need to unpack to understand. We created a black box that would need to be explored as opposed to the previous version, which while verbose, used the core React APIs most likely already familiar. Since we don't extract and version the module, breaking changes are not a big deal as we could quickly make necessary adjustments in the places where the module is consumed.

## Exhibit B: Toast notifications

It wouldn't be much of a pattern if it didn't repeat. To see how it manifests again and again in this kind of problems, let's look at one more example. We are going to build a simplified toast notification system. We want to be able to ask for a notification anywhere in our component tree and then show it in a global bar. Again, let's identify what pieces we are going to need:

1) a context provider exposing a function that can be used to ask for a toast,
2) a component rendering the toast bar,
3) a place to store the notifications state and related logic

We already know what to expect so let's skip the first draft and go directly to the version with a separate module:

<iframe
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  src="https://glitch.com/embed/#!/embed/hooks-toasts?path=feature-toggle.jsx&previewSize=0"
  alt="hooks-toasts on Glitch"
  width="100%"
  height="431px">
</iframe>

It's our Choco pattern on full display! And as in the previous example, we can refactor this to make the API cleaner and more controlled. It could look like this:

<iframe
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  src="https://glitch.com/embed/#!/embed/hooks-toasts-2?path=toasts.jsx&previewSize=0"
  alt="hooks-toasts-2 on Glitch"
  width="100%"
  height="431px">
</iframe>

As in the feature toggle example, I chose to only expose some part of the context value to the component tree through the custom hook. In this case, the array of toasts remains private, while the functions to add and remove toasts are exposed.

## Exhibit C: User guide
To cement the pattern further, let's look at one last example. We will build user guide functionality. We want to add a help button in some places in our app, which when clicked, opens a user guide widget that explains the part of the interface in question. You know the drill by now, let's see what we're going to need:

1) a context provider exposing the help icon button component
2) a component rendering the user guide UI
3) a place to store state (i.e. which piece of the interface the user asked for) and related logic

Our first iteration could look like this:

<iframe
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  src="https://glitch.com/embed/#!/embed/hooks-user-guide?path=user-guide.jsx&previewSize=0"
  alt="hooks-user-guide on Glitch"
  width="100%"
  height="431px">
</iframe>

Hopefully, the structure is intuitive and familiar by now. The only significant difference is the type of functionality exposed to the other components. In the case of the guide, we choose to expose a component, while in the previous examples it was a method or state data. Anything goes! 

Following the previously established procedure, the refactored version of the same code could look like this:

<iframe
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  src="https://glitch.com/embed/#!/embed/hooks-user-guide-2?path=user-guide.jsx&previewSize=0"
  alt="hooks-user-guide-2 on Glitch"
  width="100%"
  height="431px">
</iframe>

## Spot the pattern

Of course, I slightly cheated with the way I presented the refactoring evolution here. I already knew the pattern and made the examples follow it from the very beginning.  In fact, when I first got to implement these features using hooks, I didn't immediately see the Choco pattern and my initial implementation attempts were chaotic and inconsistent. But with time, I started to see the same code paths and elements showing up repeatedly and I was able to streamline and clean up my implementations.
 
Discovering and embracing patterns can be a powerful technique that can make penetrating and understanding a codebase easier. I'm always on a lookout for concerns and features that share similar requirements and characteristics and try to implement them in a consistent way, without introducing abstractions prematurely. 

I'm really eager to find out what other concerns are candidates for using this pattern and what other patterns using hooks will surface. Let me know what you think!
