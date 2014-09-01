# The Conventional Front-End

Conventions are a great thing. Frameworks such as Ruby on Rails and ASP.NET MVC are good examples of conventional MVC implementations. Conventions are essentially sensible defaults. For example, when you call `return View()` without any arguments in ASP.NET MVC, the framework does the reasonable thing: render the default view that maps to the controller action. This type of behavior can be observed throughout the framework.

When it comes to the front-end, conventions aren't as popular as I'd like. In this article we'll take a look at [`measly`][1], a conventional layer on top of `XMLHttpRequest` you can use to get started in the conventional world.

  [1]: https://github.com/bevacqua/measly

> **Everything is about context.**

How many times have you seen or worked on web applications where dozens of `XMLHttpRequest` objects were used to make AJAX requests? Pretty much any web application, right? What about doing the right thing when those requests fail? Not so many perhaps, but I'd be willing to bet you still worked on quite a few where at least `404` or `500` status codes were handled. That's still not good enough if you are handling them every time, whenever you made a request.

Maybe something like the piece of pseudo-code below.

```js
$.get('/api/cats').then(meow).fail(MEOWWW);
```

A much better alternative is to centralize this kind of error checking, so that the rest of the application can live happily ever after without a worry in the world about status codes other than `2xx`. That's where [`measly`][1] comes in. Measly allows you to easily define a layered hierarchy for your components. It's able to handle requests on any of these layers, or on the request itself.

To get you started, the first thing you'll need to do is create a layer whenever a view or partial view is rendered. If you were using [Taunus][3], then that would be pretty easy. [Taunus][3] let's you hook into the rendering engine and listen for views being rendered. You can use this hook to create a `measly` layer associated to each view's container.

```js
taunus.on('render', function (container, model) {
  model.measly = measly.layer({ context: container });
});
```

Once every view has it's own `measly` layer, you could create a global hook to render error messages in the context for the layer where the failed request originated. Note that if I created the hook on `model.measly` instead of just `measly`, it would've only affected requests on an specific view.

The `render` method shown below uses some code written in [`Dominus`][2], a jQuery-like library, to render the error messages. Since we're using the context of the partial where the request originated, the error will be displayed close to whatever the user was interacting with, at the top of that partial view. However, you could easily change the rendering logic if you wanted to.

```js
measly.on(404, render);

function render (err, body) {
  var context = $(this.context);
  var messages = $('<ul>').addClass('vw-validation');

  $(body.messages.map(dom)).appendTo(messages);

  context.find('.vw-validation').remove();
  context.prepend(messages);
}

function dom (message) {
  return $('<li>').text(message).addClass('vw-validation-message')[0];
}
```

Note that this approach to error reporting in the DOM forces you to have some sort of convention when it comes to JSON responses. That is a good thing, though!

You're now able to make requests through measly and never again worry about failure responses. The example below shows how you could do that with access to the model, presumably in a partial view controller.

```js
function (model) {
  $('.ca-save').on('click', function () {
    model.measly.put('/api/cats', model.cat);
  });
}
```

What other kinds of conventional tactics would you apply to your front-end architecture?

  [1]: https://github.com/bevacqua/measly
  [2]: https://github.com/bevacqua/dominus
  [3]: https://github.com/bevacqua/taunus
