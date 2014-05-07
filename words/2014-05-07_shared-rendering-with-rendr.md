# Shared Rendering with Rendr

[Rendr][2] boosts the perceived performance of Backbone applications by rendering them in the server-side. This allows us to display the rendered page before JavaScript code is executed in the browser and Backbone kicks in. In other words, it means that, the first time the page gets loaded, the human will get to see the content earlier. After that first load, Backbone will have taken over and handle routing in the client-side. However, the first load is extremely important, and having the ability to render the application on the server before the human gets any content is better than having them wait for Backbone to pull your data, fill your views, and render your templates. Search engine optimization is another reason why you might want to render your views in the server-side as well. That's why server-side rendering is still vital to the web application development process.

[![rendr.png][1]][2]

Let's take a quick dive into the world of [Rendr][2].

  [1]: http://i.imgur.com/88hPdca.png
  [2]: https://github.com/rendrjs/rendr

# Swimming Into Rendr

Rendr uses a conventional approach to application building. That means it expects you to name your modules in a certain way, and place them in certain directories. Rendr also has some opinions as to the kinds of templates you should use, and how your application is to access its data. By default, this means Rendr expects you to have a REST API to access the application data.

Rendr runs on Node, acting as a middleware in your HTTP stack. It works by intercepting requests and rendering views server-side, before handing the pre-rendered results to the client. In its conventional approach, it helps you to separate concerns by defining controllers, where you can fetch data, render views, or perform redirects. Rather than having to reference your templates in your views, Rendr uses well-defined naming policies that abstract away dependencies, which are mostly managed by the Rendr engine. This will become clearer once we start looking at code.

### Problems in Paradise

Not all is peaches and cream. Rendr includes some "peculiar" design choices that ultimately make me wonder if it's worth pursuing. However, it's worth it to play around with it and get your own feel for how well it works for you. For instance, Rendr uses Browserify to bring the modules you write into the browser, but it has three distinct hacks in the way it compiles your Common.JS modules using Browserify.

- jQuery needs to be shimmed through `browserify-shim`. This is problematic because the server-side version of Rendr uses its own version of jQuery, and there could be versioning discrepancies. If you try to use the Common.JS version, obtained through `npm`, it won't work.
- It requires aliases for some of its `require` calls to work as expected, which is also an issue because it translates into the next deficiency, as well.
- You won't be able to use the `brfs` transform to read files at build time because Rendr doesn't play nice with it.

### Diving In

To show-case Rendr, I've settled for a slightly modified version of an example AirBnB (the company behind Rendr) uses to teach how Rendr works. You can find the code at `ch07/11_entourage` in the accompanying code samples.

First off, let me talk about the templates. Rendr encourages us to use a superset of Mustache, called Handlebars. Handlebars provides extra features, mostly in the form of helper methods you can use, such as an `if` convenience method. Rendr expects us to compile the Handlebars templates and place the bundled result in `app/templates/compiledTemplates.js`. To do that, we'll start by installing the Grunt plugin for Handlebars.

```
npm install --save-dev grunt-contrib-handlebars
```

To configure the Handlebars Grunt plugin, we'll have to add the following snippet to the `Gruntfile`. The `options` passed to the `handlebars:compile` task target are needed by Rendr, which expects the templates to be named in a certain way.

```js
handlebars: {
  compile: {
    options: {
      namespace: false,
      commonjs: true,
      processName: function (filename) {
        return filename.replace('app/templates/', '').replace('.hbs', '');
      }
    },
    src: 'app/templates/**/*.hbs',
    dest: 'app/templates/compiledTemplates.js'
  }
}
```

The Browserify configuration is, at the moment, also tied to Rendr's expectations. You'll need to shim jQuery, rather than just installing it from `npm`. You are expected to provide an alias so that Rendr can access `rendr-handlebars`, the Handlebars adapter used by Rendr. Lastly, Rendr needs you to provide a few mappings to be able to access your application's modules. The code to configure Browserify to play nice with Rendr can be found below.

```js
browserify: {
  options: {
    debug: true,
    alias: ['node_modules/rendr-handlebars/index.js:rendr-handlebars'],
    aliasMappings: [{
      cwd: 'app/',
      src: ['**/*.js'],
      dest: 'app/'
    }],
    shim: {
      jquery: {
        path: 'assets/vendor/jquery-1.9.1.min.js',
        exports: '$'
      }
    }
  },
  app: {
    src: ['app/**/*.js'],
    dest: 'public/bundle.js'
  }
}
```

That's it, as far as build configuration goes. It might not be ideal, but once it's in there you can forget about it. Let's go into the sample application code, and how it works.

# Understanding Boilerplate in Rendr

The first step we'll take in putting together our Rendr application is creating the entry point for the Node program. We'll name this file `app.js` and place it in our application root. As I mentioned earlier, Rendr works as a middleware in our HTTP stack, sitting inside Express.

### Express Middleware for Rendr

Express is a popular Node.js framework that wraps the native `http` module providing a bit more of functionality, allowing you to perform routing, and a few more things. Past this section, most of what we'll discuss is inherent to Rendr, and not part of Express. Rendr enhances Express to make its conventions work, though.

```shell
npm install express --save
```

Have a look at the piece of code below. Here I'm using the `express` package to set up an HTTP server in Node. Calling `express()` will create a new Express application instance, and we can add middleware to that instance with `app.use`. Invoking `app.listen(port)` will keep the application running and react on incoming HTTP requests on the chosen port. Best practice dictates that the listening port for our application should be configurable as an environment variable, and have a sensible default value.

```js
var express = require('express');
var app = express();
var port = process.env.PORT || 3000;

app.use(express.static(__dirname + '/public'));
app.use(express.bodyParser());
app.listen(port);

console.log('listening on port %s', port);
```

The `static` middleware tells Express to serve all of the content in the specified directory as static assets. This means that if a human requests `http://localhost:3000/js/foo.js`, and the `public/js/foo.js` file exists, that's what Express will respond with. The `bodyParser` middleware is a utility that will parse request bodies that are detected to be in JSON, or form data format.

The following piece of code configures Rendr for our example. The middleware will take care of everything else, as we'll see next. The data adapter configuration tells Rendr what API it should be querying. The beauty of Rendr lies in that, both in the client-side as well as the server-side, it'll just query the API whenever it needs to fetch data.

```js
var rendr = require('rendr');
var rendrServer = rendr.createServer({
  dataAdapterConfig: {
    default: {
      host: 'api.github.com',
      protocol: 'https'
    }
  }
});

app.use(rendrServer);
```

### Setting Up Rendr

Rendr provides a series of base objects you are expected to extend when building your application. The `BaseApp` object, which extends from `BaseView` should be extended, and placed in `app/app.js` in order to create a Rendr app. In this file you could add app initialization code that runs in both the client and the server, and it's used to maintain the application's global state. The snippet of code below will suffice.

```js
var BaseApp = require('rendr/shared/app');

module.exports = BaseApp.extend({
});
```

We also need to create a router module, which we could be using to track page views whenever there's a route change, although for now we'll merely create an instance of the base router. The router module should be placed at `app/router.js`, and it should look like the piece of code shown below.

```js
var BaseClientRouter = require('rendr/client/router');

var Router = module.exports = function Router (options) {
  BaseClientRouter.call(this, options);
};

Router.prototype = Object.create(BaseClientRouter.prototype);
Router.prototype.constructor = BaseClientRouter;
```

Let's turn our attention to how the meat of your Rendr application should look like.

# A Simple Rendr Application

So far, we've configured Grunt and Express to comply with Rendr's needs. Now it's time to develop the application itself. To make this example easier to grok, I'll show you the code in the logical order Rendr uses to serve its responses.

In order to keep our example self-contained, yet interesting, we'll create three different views.

- Home is the welcome screen for our app
- Users keeps a list of GitHub users
- User contains the details of an specific user

These views will have a 1-to-1 relationship with routes. The home view will sit at the application root, `/`; the user list will be at `/users`; and the user details view will be at `/users/:login`, where `:login` is the user login on GitHub ([`bevacqua`][1] in my case). Views are rendered by controllers. Let's start with routing, and then learn how controllers operate.

#### Routes and Controllers

The code below matches routes to controller actions. The controller actions should be defined as the controller name, followed by a hash, and then the action name. This module goes into `app/routes.js`, and you'll find the code below.

```js
module.exports = function (match) {
  match('',                   'home#index');
  match('users'       ,       'users#index');
  match('users/:login',       'users#show');
};
```

Controllers are used to fetch any data that's required to render a view. You'll have to define each action that is expected by the routes. Let's put the two controllers together. By convention, controllers should be placed in `app/controllers/:name_controller.js`. The snippet below is our Home controller, and should be placed at `app/controllers/home_controller.js`. It should expose an `index` function, matching the `index` route. This function takes a parameters object and a callback, that once called will render the view.

```js
module.exports = {
  index: function (params, callback) {
    callback();
  }
};
```

The `user_controller.js` module is a bit different. It has an `index` action as well, but it also has a `show` action. In both cases, we need to call `this.app.fetch` with some parameters, in order to get the model data, and then invoke the callback once we're done.

```js
module.exports = {
  index: function (params, callback) {
    var spec = {
      collection: {
        collection: 'Users',
        params: params
      }
    };
    this.app.fetch(spec, function (err, result) {
      callback(err, result);
    });
  },
  show: function (params, callback) {
    var spec = {
      model: {
        model: 'User',
        params: params
      },
      repos: {
        collection: 'Repos',
        params: { user: params.login }
      }
    };
    this.app.fetch(spec, function (err, result) {
      callback(err, result);
    });
  }
};
```

Fetching this data wouldn't be possible if we didn't have matching models and collections. Let's flesh those out next.

### Models and Collections

Models and collections need to extend the base objects provided by Rendr, so let's create those. Below is the code for our base model, placed at `app/models/base.js`.

```js
var RendrBase = require('rendr/shared/base/model');

module.exports = RendrBase.extend({});
```

The base collection is similarly thin. Having your own base objects, though, is necessary to easily share functionality across your models.

```js
var RendrBase = require('rendr/shared/base/collection');

module.exports = RendrBase.extend({});
```

We'll have to define our models using the endpoint we want to use to fetch the models, in this case from the GitHub API. Our models should also export a unique identifier that's what we used when calling `app.fetch` in our User controller. Below is how the User model looks like. This should be placed at `app/models/user.js`.

```js
var Base = require('./base');

module.exports = Base.extend({
  url: '/users/:login',
  idAttribute: 'login'
});
module.exports.id = 'User';
```

As long as your models don't have any validation or computed data functions, they'll look quite similar. A `url` endpoint, the unique identifier, and the name of the parameter that's used to look up a single model instance. Here's how the Repo model looks like.

```js
var Base = require('./base');

module.exports = Base.extend({
  url: '/repos/:owner/:name',
  idAttribute: 'name'
});
module.exports.id = 'Repo';
```

Backbone collections need to reference a model to learn what kind of data they're dealing with. Collections are similar to models, using a unique identifier to teach Rendr what kind of collection they are, and having a URL where we can fetch that data from. Below, you'll find the Users collection in code. It should be placed in `app/collections/users.js`.

```js
var User = require('../models/user');
var Base = require('./base');

module.exports = Base.extend({
  model: User,
  url: '/users'
});
module.exports.id = 'Users';
```

The Repos collection is almost identical, except it uses the Repo model, and it has a different URL for fetching the data from the REST API. The code is as below, and it should go in `app/collections/repos.js`.

```js
var Repo = require('../models/repo');
var Base = require('./base');

module.exports = Base.extend({
  model: Repo,
  url: '/users/:user/repos'
});
module.exports.id = 'Repos';
```

At this point, the user requested a URL, and the router decided which controller action that should direct them to. The action method probably fetched some data from the API, and then it invoked its callback. At last, let's learn how views behave to render the HTML.

### Views and Templates

As with most things Rendr, the first step in defining our views is creating our own base view, which is an extension of Rendr's base view. The base view should go in `app/views/base.js`, and look like the piece of code below.

```js
var RendrBase = require('rendr/shared/base/view');

module.exports = RendrBase.extend({});
```

Our first view is the Home view. It should be placed at `app/views/home/index.js`, and look like below. As you can see, views also need to export an identifier.

```js
var BaseView = require('../base');

module.exports = BaseView.extend({
});
module.exports.id = 'home/index';
```

Given that our views consist mostly of links to each other, but not a lot of functionality, they're mostly empty. The Users view is almost identical to the Home view. It goes in `app/views/users/index.js`, and its code is below.

```js
var BaseView = require('../base');

module.exports = BaseView.extend({
});
module.exports.id = 'users/index';
```

There is also the User Details view, which goes in `app/views/users/show.js`. This view has to tamper with the template data, which is what I've been referring to as view model, in order to make the `repos` object available to the template as well.

```js
var BaseView = require('../base');

module.exports = BaseView.extend({
  getTemplateData: function () {
    var data = BaseView.prototype.getTemplateData.call(this);
    data.repos = this.options.repos;
    return data;
  }
});
module.exports.id = 'users/show';
```

The last view we'll put together is a partial to render a list of repositories. It should be placed in `app/views/user_repos_view.js`, and as you can see, partials barely differ from other views, and they need a view controller just like any other view.

```js
var BaseView = require('./base');

module.exports = BaseView.extend({
});
module.exports.id = 'user_repos_view';
```

Lastly, there's the view templates. The first view template we'll take a look at is the `__layout.hbs` file. This is the HTML that will serve as a container for all of your templates. You can find the code below. Note how I'm bootstrapping the application data, and initializing it, using JavaScript. This is required by Rendr. The `{{{body}}}` expression will be replaced by the views dynamically as the route changes.

```html
<!doctype html>
<html>

  <head>
    <title>Entourage</title>
  </head>

  <body>
    <div>
      <a href='/'>GitHub Browser</a>
    </div>
    <ul>
      <li><a href='/'>Home</a></li>
      <li><a href='/users'>Users</a></li>
    </ul>

    <section id='content' class='container'>
      {{{body}}}
    </section>

    <script src='/bundle.js'></script>
    <script>
    (function() {
      var App = window.App = new (require('app/app'))({{json appData}});
      App.bootstrapData({{json bootstrappedData}});
      App.start();
    })();
    </script>
  </body>
</html>
```

Next up we have the Home view template. Here I just have a few links and there's no view model data access going on. This template goes in `app/templates/home/index.hbs`. Note that Backbone will capture navigation to any links in your application that match one of its routes, and behave as a single page application. Rather than reloading the entire page whenever a link is clicked, Backbone will just load the corresponding view.

```html
<h1>Entourage</h1>
<p>This little app demonstrates how to use Rendr by consuming GitHub's public API.</p>
<p>Check out <a href='/repos'>Repos</a> or <a href='/users'>Users</a>.</p>
<p>
  <i><a href='https://github.com/rendrjs/rendr'>Rendr</a> is built by AirBnB</i>
</p>
```

Now things get a bit more interesting. Here we are looping through the list of models that were fetched in the controller action, and render a list of users and links to their account details. This template goes in `app/templates/users/index.hbs`.

```html
<h1>Users</h1>

<ul>
{{#each models}}
  <li>
    <a href='/users/{{login}}'>{{login}}</a>
  </li>
{{/each}}
</ul>
```

Next up we have the user details template, which goes in `app/templates/users/show.hbs`. You can find the template code below. Take into account how I'm telling Handlebars to load the `user_repos_view` partial, and how that name matches exactly the identifier that was defined in its view.

```html
<img src='{{avatar_url}}' width='80' height='80'> {{login}} ({{public_repos}} public repos)

<br>

<div>
  <div>
    {{view 'user_repos_view' collection=repos}}
  </div>

  <div>
    <h3>Info</h3>
    <br>
    <table>
      <tr>
        <th>Location</th>
        <td>{{location}}</td>
      </tr>
      <tr>
        <th>Blog</th>
        <td>{{blog}}</td>
      </tr>
    </table>
  </div>
</div>
```

The User Repos view is our last view template, a partial in this case. It has to be located at `app/templates/user_repos_view.hbs`, and it's used to iterate through a collection of repositories, displaying interesting metrics about each repository.

```html
<h3>Repos</h3>
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Watchers</th>
      <th>Forks</th>
    </tr>
  </thead>
  <tbody>
  {{#each models}}
    <tr>
      <td>{{name}}</td>
      <td>{{watchers_count}}</td>
      <td>{{forks_count}}</td>
    </tr>
  {{/each}}
  </tbody>
</table>
```

That's it! Phew. You can find all of this code, in a [working code sample here][2]. The resulting code looks sort of like the screenshot below, after some styling.

![rendr-entourage.png][3]

Note that there are alternatives to Rendr. If you are concerned about the issues I raised earlier, here are some resources you might want to take a look at.

- [Rendering on the Server and Client in Node.js][4]
- [Ezel.js][5]
- [Shared Rendering in Node and the Browser][6]

_Until the next time!_

  [1]: https://github.com/bevacqua
  [2]: https://github.com/bevacqua/buildfirst/tree/master/ch07/11_entourage
  [3]: http://i.imgur.com/ujaxhze.png
  [4]: http://artsy.github.io/blog/2013/11/30/rendering-on-the-server-and-client-in-node-dot-js/ "Rendering on the Server and Client in Node.js - Artsy"
  [5]: http://ezeljs.com/
  [6]: http://substack.net/shared_rendering_in_node_and_the_browser "Shared Rendering in Node and the Browser - substack"

[isomorphic backbone rendr mustache]
