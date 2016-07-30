# Using `react-router`

The Express `app.js` module we [built in the last article][1] looks something like this.

```js
import express from 'express';
import hbs from 'express-handlebars';
import React from 'react/addons';
<mark>import App from './components/app';</mark>

var app = express();
app.engine('html', hbs({ extname: 'html' }));
app.set('view engine', 'html');
app.locals.settings['x-powered-by'] = false;
<mark>app.get('/', function home (req, res, next) {
  res.render('layout', {
    reactHtml: React.renderToString(<App />)
  });
});</mark>
app.listen(process.env.PORT || 3000);
```

If we wanted to add more routes, we'd have to add more statements like `app.get('/', function...)` above, as well as load each component necessary to render every one fo those routes. As your application grows, complexity would grow linearly as well. A better alternative is using [`react-router`][2], which allows you to remove Express from the equation and leave routing to the React application itself. Let's install `react-router` via `npm`.

```shell
npm i react-router -S
```

Then, you'll need to import the module into your `app.js` file.

```js
import Router from 'react-router';
```

We'll now defer routing to `react-router` instead of _routing at the Express level_, using `app.get` and the like. If you're following along in code, get rid of the `app.get('/', function...)` piece, and also slash the `import` statement for `App`. We're going to implement [a middleware method][3] for Express that will handle routing on its behalf.

```js
app.use(router);
```

The `router` method is displayed below. It creates a routing context using `react-router` and the request's `req.url`. `react-router` will figure out the component that should be rendered for that particular `location`, and we'll get that back as a `Handler`. We can then leverage JSX to render the `<Handler />` using [`React.renderToString`][4].

```js
function router (req, res, next) {
  var context = {
    <mark>routes: routes</mark>, location: req.url
  };
  Router.create(context).run(function ran (Handler, state) {
    res.render('layout', {
      reactHtml: React.renderToString(<Handler />)
    });
  });
}
```

<sub><mark>*</mark> _More on these in a moment!_</sub>

Note how this method is component-agnostic, as it should be able to figure out what component to render based on all the `routes` that we have and the `location` the human is trying to visit. This helps us decouple the web application from our React components for good.

## Defining Your Routes

How does the `routes` object look like? It should be a module, because you'll also be leveraging the exact same module in your client-side code so that routing stays consistent. Fair enough, let's add the `import` statement.

```js
import routes from './routes';
```

How should the module actually look like? Something like this, maybe? As you can see, the route definitions for `react-router` leverage JSX to declare **a nested route hierarchy**. We only have one route, though.

```js
import React from 'react';
import {Route} from 'react-router';
import HomeIndex from './components/home/index';

export default (
  <Route path='/' handler={HomeIndex}>
  </Route>
);
```

Except that, _most of the time_, you'll have pieces of your app **outside of routing** -- your navigation, sidebars and whatnot. In order to _future-proof_, you're better off defining an `<App />` component as well. Consider the following mockup from the `react-router` documentation, which illustrates the point well enough.

![A mockup of an application with a top navigation bar and a dashboard][5]

It'll be useful to wire up an `<App />` component where we can at a later point add some navigation elements. Let's start with the changes to the `router.js` module. You just need to also import the `components/app` component, and change the routing definition.

```js
import React from 'react';
import {Route<mark>, DefaultRoute</mark>} from 'react-router';
<mark>import App from './components/app';</mark>
import HomeIndex from './components/home/index';

export default (
  <Route path='/' handler={App}>
    <<mark>DefaultRoute</mark> handler={HomeIndex} />
  </Route>
);
```

The `<DefaultRoute />` is used when the parent route's `path` is matched exactly. So when the human navigates to `/`, `<HomeIndex />` will be rendered.

Now that our routing is wrapped within the `<App />` component, you can place shared functionality and markup in that component _(like we said earlier -- navigation and whatnot)_. For the time being though, our `components/app.js` file is almost an empty component. The relevant code is highlighted.

```js
import React from 'react'
import {RouteHandler} from 'react-router';

export default class App extends React.Component {
  render () {
    return <mark><RouteHandler /></mark>
  }
};
```

You can think of `<RouteHandler />` as _"nesting continues here"_. In other words, whenever you have a React component rendered using `react-router`, it'll be rendered at `<RouteHandler />` in its _parent route's_ component. See [the documentation][6] if I lost you on that one.

Continuing with the example about a human visiting `/`, the `react-router` will render `{HomeIndex}`, and then jump to the parent route. The parent has an `{App}` handler, so it'll render that and place the result of rendering `{HomeIndex}` inside `{App}`'s `<RouteHandler />`. It's very straightforward and subtly powerful.

React's `react-router` was modeled after the [often-praised Ember router][7]. You can nest your routes as deep as you need to, and you can mix in as many components as needed too. All your routing needs are now covered by `routes.js`.

> But, wait! What about client-side routing to match?

There isn't much else that needs to be done on the client-side to match your server-side `react-router` routes. Let's go back to what we used to have. As you probably guessed, references to your old `<App />` will now be removed in favor of our newfound routing capabilities.

```js
import React from 'react/addons';
<mark>import App from '../../components/app';</mark>
var main = document.getElementsByTagName('main')[0];

React.render(<mark><App /></mark>, main);
```

We'll be once again pulling in the `react-router` as well as reusing the `routes.js` module we've defined for the server-side, one directory up. Instead of rendering `<App />` directly like we used to, we're going to defer to the wisdom of [`react-router`][2] to tell us what component should be rendered. You can specify whether you want the router to work through hashes like `#/foo/bar` _(the default)_, or via the [`history` API][8] -- by specifying the use of `Router.HistoryLocation` explicitly.

```js
import React from 'react/addons';
import Router from 'react-router';
import routes from '../routes';
var main = document.getElementsByTagName('main')[0];

Router.run(routes, <mark>Router.HistoryLocation</mark>, function ran (Handler, state) {
  React.render(<mark><Handler /></mark>, main);
});
```

We're done. You should now understand how to handle routing in your brand new React app, how to get that working on both the server-side and the client-side without having to make changes in multiple places, and how to leverage nesting so that you can add some navigation or layout to your app from within a React component.

> This is fun stuff!

  [1]: /articles/universal-react-babel "Universal React with Babel, Browserify on Pony Foo"
  [2]: https://github.com/rackt/react-router "rackt/react-router on GitHub"
  [3]: http://expressjs.com/guide/using-middleware.html "Using Express middleware"
  [4]: https://facebook.github.io/react/docs/top-level-api.html#react.rendertostring "React.renderToString() documentation"
  [5]: https://i.imgur.com/5Vkx3Tm.png
  [6]: https://github.com/rackt/react-router/blob/0.13.x/docs/guides/overview.md "Architecture Overview for react-router"
  [7]: http://guides.emberjs.com/v1.10.0/routing/defining-your-routes/ "Defining Your Routes in Ember.js"
  [8]: https://developer.mozilla.org/en-US/docs/Web/API/History "History API documentation on MDN"
