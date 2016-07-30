# The Plan

The first thing I did was installing some dependencies. I decided I would use `browserify` because that way I can easily leverage any of the many CommonJS modules on `npm`. I'd use the [`babelify`][1] transform to turn ES6 code into something the browser understands, and [`babel-node`][2] _(for now -- it's not meant for use in production)_ to run that code on the server-side. I'll be using `nodemon` and `watchify` to speed up my development cycle, and the latest version of **io.js** _(`3.0.0` at the time of this writing)_.

## io.js and `nvm`

First off, if you don't use **io.js** or `nvm`, it's time. Installing `nvm` let's you easily switch around different versions of Node.js _(and io.js -- whatever)_ without any friction. Installing [`nvm`][3] is easy, and it makes installing different versions of `node` just as easy. Here's how to install `nvm`:

```shell
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.26.0/install.sh | bash
```

Once you have `nvm`, you can easily install any version of `io.js`. We'll install `3.0.0`.

```shell
nvm install iojs-v3.0.0
```

When you want to switch to that version of the `node` binary, just run the following.

```shell
nvm use iojs-v3.0.0
```

## Getting Started

Next up let's create a directory for our application and install some dependencies.

```shell
mkdir my-app
cd my-app
npm init
npm i react -S
npm i babel babelify browserify watchify nodemon -D
```

### Setting up Babel

The next order of business is to set up Babel, so that we can leverage ES6 everywhere. When it comes to the server-side, we can use [`babel-node`][4] as a drop-in replacement for `node`. There's a big warning sign against using this in production flows, but it works very well during development.

First off, we'll need a `scripts` entry for `babel-node` in our `package.json`. This is necessary because `npm run` understands how to find the `babel-node` executable, so now you can do `npm run babel-node app.js` and run things using `babel-node` without installing it globally and [messing up versioning][5].

```json
{
  "scripts": {
    "babel-node": "babel-node --stage 0"
  }
}
```

When it comes to client-side ES6, we were already going to compile our code into a bundle via `browserify`, so it makes a lot of sense to throw in the `babelify` transform in there. Babelify leverages Babel to automatically transform our ES6 code into ES5 when bundling the code in our package. We can easily set `babelify` up by adding the following entry to our `package.json`.

```json
{
  "browserify": {
    "transform": [
      ["babelify", { "stage": [0] }]
    ]
  }
}
```

You also definitely want a continuous development script. These save you precious time during development. Whenever your source code changes, the Browserify bundle will be rebuilt. Whenever the server-side code changes, the application should be restarted. For now, we can stick to just those two things. You'll need a small change to `package.json` and a build script.

```json
{
  "scripts": {
    "start": "build/build-development"
  }
}
```

```shell
#!/bin/bash

watchify client/main.js -o public/bundle.js -dv &
nodemon --exec npm run babel-node -- app.js
```

<sub>_Don't forget to run `chmod +x build/build-development` so that you can execute that script!_</sub>

The `-dv` flags on `watchify` mean debug mode _(source maps)_ and verbose output _(a single line written to the terminal whenever the bundle gets recompiled)_. We pass `--exec npm run babel-node` to `nodemon` so that it runs our app through `babel-node` instead of using the regular `node` executable.

### Using JSX? Built-into Babel

Given that Babel now has built-in support for JSX, -- _not very modular of Babel, but we'll play along_ -- I figured this is _a great opportunity_ to try out JSX. In case you haven't ever tried React before, you've probably heard the saying: **"Everyone hates JSX until they try it"**. Thus far I'm on the "what is this non-sense" camp, but I'll probably end up accepting it.

In case you have no idea what I'm talking about, JSX is a templating engine from Facebook that allows you to embed XML into your JavaScript files, enabling seemingly awful lines of code such as `React.renderToString(<mark><App /></mark>)`, as we'll explore in a minute.

## Onto the server-side

When it comes to the server-side, I'll stick to what I'm comfortable with. I'll be using `express`. We need something to render the layout surrounding our React application, and I chose `express-handlebars` for that, but really any kind of templating language would do -- and to be fair, we probably could get away with just [using ES6 template strings][6].

```shell
npm i express express-handlebars -S
```

Let's put together our `app.js` file using some light ES6 code. First off, some `import` statements. These are equivalent to doing `var express = require('express');` and so forth. We'll get to the `App` later on. For now, all you need to know is that this will be the root entry point of our application. The server-side and the client-side will leverage `App` slightly differently in order to render the application on both the server-side and the client-side.

```js
import express from 'express';
import hbs from 'express-handlebars';
import React from 'react/addons';
import App from './components/app';
```

Now that we have our dependencies in place, we can configure basic stuff about Express. This sets up `express-handlebars` so that we can place our layout in `views/layout.html`. I've also turned off the `x-powered-by` header because it makes no sense to advertise your technology stack like that.

```js
var app = express();
app.engine('html', hbs({ extname: 'html' }));
app.set('view engine', 'html');
app.locals.settings['x-powered-by'] = false;
```

The view route is where things get a tad more interesting. This route will be hit once, whenever the page is first loaded. After that, the client-side rendering engine in React will take over. We'll get worried about routing and whatnot another day, for now our focus is on figuring out <mark>the correct way to render React apps</mark>.

```js
app.get('/', function home (req, res, next) {
  res.render('layout', {
    reactHtml: React.renderToString(<mark><App /></mark>)
  });
});
```

The `<App />` expression is just JSX for `React.createFactory(App)({})`. Good thing JSX is built into Babel! The `app` should listen on a port, so that you can actually visit the site.

```js
app.listen(process.env.PORT || 3000);
```

Oh, and the `layout.html` we've been discussing should be placed in `views/layout.html`. For the moment, we'll get away with this measly piece of code. This will guarantee that if we can actually see some HTML, it'll be because it was server-side rendered. Here is `layout.html` in all its glory.

```html
<main>{{{reactHtml}}}</main>
```

Now let's turn into what you came here for.

## Some Actual React Code

The `App` component looks like the piece of code below. It was taken from [`react-starter-es6-babel`][7] _(as the app itself is not important)_ with some minor modifications, so that **it works as a _universal_ script**. If you head over to their repository, you'll notice here I'm exporting the component via `export default`, instead of just calling `React.render(<App />, el)`.

That simple change will allow us to render the component on both the server-side and the client-side alike. The `App` component `extends` the `React.Component` `class`. It has a `state` property with an `n` value -- that can be incremented using a button. It gets rendered through a `render` method which returns the JSX that makes up the component. It also binds a click handler to another method on the component, which changes the state by calling `setState` on the component.

```js
import React from 'react'

<mark>export default</mark> class App extends React.Component {
  constructor () {
    super()
    this.state = { n: 0 }
  }
  render () {
    return <div>
      <h1>clicked {this.state.n} times</h1>
      <button onClick={this.handleClick.bind(this)}>click me!</button>
    </div>
  }
  handleClick () {
    this.setState({ n: this.state.n + 1 })
  }
}
```

## Trying it out

If you run `npm start`, it'll execute the `app.js` script via `babel-node`, and if you visit the application at `http://localhost:3000`, you should see something like the screenshot below.

![The application running on Google Chrome][8]

You might also notice that the button doesn't do anything, even though our component has a click handler and everything. If we retrace our steps, you'll remember that our layout _only_ consists of the rendered React HTML inside a `<main>` tag -- not very dynamic.

## Onto the client-side

We've already set up our build to `browserify` a bundle earlier. Let's put that bundle together at `client/main.js`. When you run `npm start` the next time, a bundle should be created at `public/bundle.js`.

```js
import React from 'react/addons';
import App from '../components/app';
var main = document.getElementsByTagName('main')[0];

React.render(<App />, main);
```

You'll need the `serve-static` package in order to serve the bundle.

```shell
npm i serve-static -S
```

It should be mounted on the `public` directory.

```js
import serveStatic from 'serve-static';

app.use(serveStatic('public'));
```

Also, don't forget to add the `<script>` tag to your `layout.html`!

```html
<main>{{{reactHtml}}}</main>
<script src='/bundle.js'></script>
```

The bundle takes over what we had already rendered on the server-side, and sets up the click handler. You can now click on the button and things will happen!

![Clicking the button increases the counter][9]

Next up we'll figure out how routing is set up, and we'll adjust our code as necessary. Think this should turn into an extensive series? It's kind of fun to write about.

  [1]: https://github.com/babel/babelify "babel/babelify on GitHub"
  [2]: https://babeljs.io/docs/usage/cli/ "Babel CLI Documentation"
  [3]: https://github.com/creationix/nvm "creationix/nvm on GitHub"
  [4]: https://babeljs.io/docs/usage/cli/ "Babel CLI Documentation"
  [5]: /articles/semver "Pragmatic Semantic Versioning on Pony Foo"
  [6]: http://www.2ality.com/2015/01/template-strings-html.html "HTML templating with ES6 template strings"
  [7]: https://github.com/substack/react-starter-es6-babel "substack/react-starter-es6-babel on GitHub"
  [8]: https://i.imgur.com/bKa0V5A.png
  [9]: https://i.imgur.com/BR10Hvh.png
