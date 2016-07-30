![static.jpg][1]

# Why `assetify`?

There are quite a few reasons, actually. The [first reason I had](/2013/01/18/asset-management-in-node "Asset Management in Node") still holds true. I don't think there are any comprehensive, simple, and **sufficiently flexible** alternatives to `assetify` out there in the world of Node. You have to either deal with the _lame constraints_ [Require.JS](http://requirejs.org/ "Require.JS Asset Loader") enforces upon your code (and everyone else's), or with package managers such as Twitter's [bower](https://github.com/bower/bower "bower Package Manager on GitHub") which doesn't really do much more than what `npm` does for Node. Except the needs for front-end asset management are **way different** than those of the Node ecosystem.

We **need** ways to:

- Figure out which assets to serve
- Pre-process the multitude of languages that compile to JS and CSS
- Mash them together to prevent so many requests
- Minify their footprints. Both in code and using [GZip](http://en.wikipedia.org/wiki/Gzip "GZip Compression")
- Slap `Expires` headers onto as many static assets as possible
- Dynamically add snippets of CSS and JS to our responses
- Keep all of the above in the correct order, so dependencies work well
- Strictly separate the build process from executing the web server
- Avoid repeating ourselves when declaring the asset sources
- Do all of the above while being able to work with ease

That's a lot of stuff we should be doing, yet, countless projects still serve their static assets straight from their development sources. This shouldn't be the prevalent case.

## The `assetify` Way

There are two distinct stages when working with assetify to manage static assets. The first step is processing the sources you developed, parsing LESS stylesheets, CoffeeScript files, bundling, minifying, etc. The second step involves serving the resulting files.

## Asset Processing

With assetify, you work on your sources, and then generate output that is meant to be consumed by your users. `assetify` is **middleware-based**, and you can _extend it with plugins_ of your own.

Here is a sample folder structure for your public static assets:

![assetify-structure.png][2]

`assetify` will take your directory structure and produce a _build result_, which you'll then use to serve your static assets. Let's walk through a basic setup.

We start off by installing `assetify`

```bash
$ npm install assetify --save
```

### Asset Configuration

This is a simple configuration file:

```js
{
    assets: {
        source: __dirname + '/public',
        bin: __dirname + '/public/.bin',
        js: [
            '/js/app.js',
            '/js/foo.js'
        ],
        css: [
            '/css/reset.css'
            '/css/layout.less',
            '/css/design.less'
        ]
    }
}
```

Let me explain these properties.

#### # assets.source

The `source` directory is the base directory where your static assets are. Don't worry, assetify supports referencing assets outside of this directory, but it will be used as the base directory for relatively referenced assets.

#### # assets.bin

The `bin` directory is used to place the results of processing your static assets.

#### # assets.js [Array]

First of all: `assets.css` works in the same way as `assets.js`, some plugins run exclusively in the appropriate set of assets, though, so it's important to place them in the correct array. More on plugins later.

### What kind of elements can be in these arrays?

Strings. Strings are syntactic sugar for an object like this: `{ file: '/the/string' }`. So what kind of properties can these objects have?

```json
{
    "file": "/js/app.js",
    "glob": "/js/service/*.js",
    "inline": false,
    "src": "alert('foo');",
    "ext": "//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js",
    "test": "window.jQuery",
    "profile": "owner"
}
```

We obviously shouldn't be setting all these properties at once, let's go over them.

#### # Asset.file

The path to the asset, relative to `assets.bin`. This path can contain `..` jumps to parent directories. `assetify` concedes that the order you output your assets in is important, and it will respect the order you define for your assets to be exposed.

#### # Asset.glob

This property isn't actually in assetify, but it comes bundled with [**grunt-assetify**](https://github.com/bevacqua/grunt-assetify "grunt-assetify task"), which is a `grunt` task that simplifies the compilation step.

As you might have guessed, if you're familiar with `grunt`, the `glob` property is special in that it allows us to set a [globbing pattern](https://github.com/isaacs/node-glob "node-glob"). When it's expanded, resulting objects will have `file` set to each matching file, and all the other properties will be preserved.

#### # Asset.inline

Typically, files will be written to disk, and script tags will reference those files. However, if `inline === true`, the script will be inlined. Resulting in HTML such as:

```html
<script>
alert("foo");
</script>
```

Also, assets that are _dynamically generated_ will be inlined. More on these later.

#### # Asset.src

Usually, you'll be using `file` to set the source code of your assets. If you need to provide the asset source code directly, you can use this property instead.

#### # Asset.ext

Sometimes, we want to reference assets in a CDN. We can do this using `ext`. Generally, its recommended that you provide a fallback in case the CDN asset fails to load, you can do that by providing a test using the test property, for example, if `window.jQuery` isn't set after we try to load jQuery from Google's CDN, we fall back to the local copy of jQuery (which should be set with `file`).

#### # Asset.profile

Last but not least, `profile`. This property allows us to set up different asset groups. You can specify a String with a single profile, or an array with multiple profiles. e.g: `['anon', 'registered']`.

The purpose of this property is to save time for our users by not making them download assets they are not going to need.

### Build Step

Now that you know how to configure your assets hash, here's how you compile your assets.

This step can be simplified with the aid of [**grunt-assetify**](https://github.com/bevacqua/grunt-assetify "grunt-assetify task").

```js
var assetify = require('assetify').instance();

assetify.compile(options, function(err){
    console.log('assetify compilation done');
});
```

Compilation will generate all the required files that we will need to serve our static assets later on. Additionally, the assetify compiler will output an `assetify.json` file that will contain some metadata that will be used to let the middleware know how to behave.

Remember how I mentioned assetify is **middleware-based**? Well, if you don't add anything else, your "compilation" isn't doing anything, it will just be a glorified copy. The thing is, once you've set up the asset options, adding functionality is really easy. Let me teach you by example.

```js
var assetify = require('assetify').instance();

assetify.use(assetify.plugins.less);
assetify.use(assetify.plugins.minifyCSS);
assetify.use(assetify.plugins.minifyJS);
assetify.use(assetify.plugins.bundle);
assetify.compile(options, function(err){
    console.log('assetify compilation done');
});
```

Just like that, your assets will be bundled together, your LESS stylesheets will be properly pre-processed, and everything will be minified.

Here is a list of all the _compilation plugins_ that come bundled with assetify.

##### # plugins.bundle

Instead of resulting in one output file as a result for each input file, you'll be getting one file as a result for each profile. If no profiles were specified, then a default, `'all'` profile, will be used.

##### # plugins.less

Your LESS CSS stylesheet files will be pre-processed

##### # plugins.sass

Your SASS CSS (and SCSS) stylesheet files will be pre-processed

##### # plugins.coffee

Your CoffeeScript JS files will be pre-processed

##### # plugins.jsn

Your [jsn](https://github.com/bevacqua/jsn "jsn") JS files will be pre-processed

##### # plugins.minifyCSS

Your CSS will be minified.

##### # plugins.minifyJS

Your JS will be minified

##### # plugins.forward(opts, concat)

By default, images in the `source` directory won't be forwarded to the compilation folder. If you want to keep all your public-facing static assets in one place, then you can forward images to the output directory.

`plugins.forward` is a function you need to call to get the plugin. The first parameter takes an object and it's optional. It defaults to:

```json
{
    "extnames": [".ico", ".png", ".gif", ".jpg", ".jpeg"]
}
```

If you want to add more forwarded extensions, you could set `concat` to true. It doesn't have to be limited to image extensions, they can be anything.

```js
var fwd = assetify.plugins.forward({ extnames: ['.woff', '.otf', '.ttf'] }, true);
assetify.use(fwd);
```

Now you can very simply use `NODE_ENV` to figure out whether you want to minify and bundle or not. It gets better, using [**grunt-assetify**](https://github.com/bevacqua/grunt-assetify "grunt-assetify task") will make the compile step even easier to configure, by letting you tell it whether you're in production or not, and using a sensible defaults-based approach.

### Plugin Precedence

Plugins don't just run in whatever order you pick, they do so _within the event they are bound to_. Events will be executed in series, not asynchronously. Each plugin will also run in series. This is required due to the nature of the compiler, and the fact that a plugin, such as the bundling plugin, might alter the very list of assets that are being processed.

This is kind of an implementation detail, but you should know this if you want to roll your own plugins.

#### `'afterReadFile'`

After reading input files, this event will be raised. At this point, language pre-processor plugins will run. Such as LESS, CoffeeScript, and jsn pre-processors. If you are including a custom pre-processor, this is the best fit.

#### `'beforeBundle'`

The `beforeBundle` event is emitted once all plugins from the previous step have completed. The bundle plugin runs in this step.

#### `'afterBundle'`

Directly after the bundling step, `afterBundle` plugins will trigger. This step includes the packed plugins for minification, and the `'forward'` plugin.

#### `'afterOutput'`

The `'afterOutput'` step is here in case we want to run any plugins after the output has been generated and written to disk. None of the distributed plugins run in this step, but you might want to develop a plugin to perform some task in this step.

#### `'beforeRender'`

This step is special in that it runs whenever asset HTML tags are going to be emitted, typically before an incoming request is going to render them. This step is used by an special plugin I'll describe in short.

## Connect Middleware

> The second piece of the assetify puzzle is serving the assets, to facilitate this, we've provided a middleware you can tack onto `connect` (or `express`), very easily.

Here is an example of how to integrate with `express`.

```js
var express = require('express'),
    app = express(),
    bin = __dirname + '/public/.bin',
    assetify = require('assetify').instance();

assetify(app, express, bin);

// routing, etc
```

We're basically passing assetify three things: the `app`, so that we can tack a middleware onto it; `express`, so we know how to use a few more configuration options, and `bin`, which is the same `bin` as the one we used in the compilation step.

Your middleware won't do much. It will, however, set up a local variable in your `res` objects, called `assetify`. This object will have a **very small API**.

#### # assetify.css.emit(profile)

This will emit all the CSS style tags you need in your view, in the order you chose, and using the assets that have been previously compiled through `assetify`. The `profile` can be omitted.

In a Jade view:

```jade
html
  head
    !=assetify.css.emit()

  body
    p Awesome!
```

#### # assetify.css.add(code, before)

Dynamically adding assets to particular requests is supported. The code will be inlined in the appropriate asset tag in the response. If `before` is true, then the asset will be added before any statically compiled assets, rather than last.

This API is repeated for JavaScript assets, in `assetify.js`.

The crucial take-away here is that we can compile and serve in two completely separate steps, and it would still work. This enables us to compile using [**grunt-assetify**](https://github.com/bevacqua/grunt-assetify "grunt-assetify task") in a `grunt` task, and then run our app just doing `node app`.

Why are we passing `express` to create this middleware? Well, there are actually a few more options we can use with `assetify`. These should also be passed to the options we've described at the beginning.

```js
{
    compress: true,
    fingerprint: true,
    expires: /^\/img\//i,
    assets: {
        favicon: '/icon.ico',
        host: 'http://localhost:3000'
    }
}
```

Let's go over each of these, too!

##### # opts.compress

Whether to use the `connect.compress()` middleware.

##### # opts.fingerprint

If this is enabled, then we're going to use [static-asset](https://github.com/bminer/node-static-asset) to produce fingerprints for our assets, sending them far into the future with expires headers. These fingerprints will only be set for assets you declare explicitly, and not for forwarded assets, since image references aren't in control of assetify.

##### # opts.expires

If this is set, then the regular expression provided will be used to give an `Expires` header to any matching request. The `favicon` will also receive this treatment.

##### # opts.assets.favicon

A favicon to use with `connect.favicon(file)`.

##### # opts.assets.host

Sometimes, it's convenient to have asset reference the absolute URL rather than a relative URL, in these cases, we can provide a host name using this property.

That's about it when it comes to using `assetify`.

# Extending `assetify`

You can include your own plugins using the following API:

```js
assetify.use(key, eventName, plugin);
```

#### # use.key

The `key` can be either omitted, or filter our plugin to only run for the `'js'` or `'css'` pipe.

#### # use.eventName

The `eventName` is one of the event steps mentioned in the **Plugin Precedence** section.

#### # use.plugin(assets, config, context, done)

Your plugin. It will receive four parameters. The `assets` will contain the list of assets that are currently being processed. You can manipulate them in any way. Keep in mind that if you modify the array, the next plugin will get your changes too.

The `config` object is the options hash that was passed to assetify when invoking the `compile` function. The `context` variable you'll receive will contain the `key`, and if you are attaching to the `beforeRender` event, you'll receive the `http` context property as well.

Once you are done doing your job, you should invoke the `done` callback, so processing can continue. If you pass an argument, it will be treated as an error.

## That's all folks

There's pretty much all you'll ever need to know about `assetify`. Let me know if you have any questions, issues, feature requests, concerns or suggestions!

  [1]: https://i.imgur.com/hpXE43e.jpg "Static Assets"
  [2]: https://i.imgur.com/huLcbuF.png "Directory structure of static assets"
