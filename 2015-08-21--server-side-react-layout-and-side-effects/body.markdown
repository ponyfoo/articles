## Modularizing the router

The first order of business will be turning our router into an individual module. So far it was just another method in `app.js`, but it'd be a bit neater to place it in a file of its own, reducing complexity and finally decoupling `app.js` from the knowledge of how views are rendered, _or even routed_.

You should add this line to `app.js`, after cutting the `router` method from that module.

```js
import router from './router';
```

Now we can move the `router` to a module of its own, the only addition would be a `export default` prefix so that the module _exports an API_ -- that's exactly the `router` method.

```js
import React from 'react';
import Router from 'react-router';
import routes from './routes';

export default function router (req, res, next) {
  var context = {
    routes: routes, location: req.url
  };
  Router.create(context).run(function ran (Handler, state) {
    res.render('layout', {
      reactHtml: React.renderToString(<Handler />)
    });
  });
}
```

What was bugging me at this point was rendering the layout in Handlebars while every view component was using JSX.

## Rendering the Layout with JSX

I decided to give a shot to a React component for the layout as well. That way I'd get more cohesiveness across the codebase _(by sticking to a single template engine)_, as well as . This component would only be used on the server-side, and we would still render the contents of each individual view separately, as we did before. The distinction is useful for the client-side rendering part, as the layout itself won't be changing -- for the most part.

In the snippet below I introduced an `import` statement that'll get us the `<Layout />` component, and changed the rendering mechanism to render the layout via React instead of Handlebars. Note how I'm passing the `main` HTML for the partial to the layout.

```js
import React from 'react';
import Router from 'react-router';
<mark>import Layout from './components/Layout';</mark>
import routes from './routes';

export default function router (req, res, next) {
  var context = {
    routes: routes, location: req.url
  };
  Router.create(context).run(ran);
  function ran (Handler, state) {
    var doctype = <mark>'<!doctype html>'</mark>;
    var main = React.renderToString(<Handler />);
    var full = React.renderToString(<mark><Layout main={main} /></mark>);
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.write(doctype + full);
    res.end();
  }
};
```

> **JSX is Awkward**
>
> Keeping the `<!doctype>` out of the JSX template is kind of awkward, but I couldn't figure out a way to include it in the JSX. The problem is that **JSX has an XML style** to its templates -- meaning you have to use self-closing `<meta />` and `<link />` tags. This also makes it hard for you to render HTML comments `<!-- foo -->` -- I haven't figured out a way how to render those either! Of course, most of the time, you don't want to declare a `<!doctype>` nor to add HTML comments (as opposed to comments in the template itself, which are easy to add).

An important portion of the equation is how the `Layout` file should look like. Here's a first version of the layout based on what we used to do in Handlebars. It leverages the `main` partial, which you can access as `this.props.main` in the React component.

```js
import React from 'react';

export default class Layout extends React.Component {
  render () {
    const { main } = this.props;

    return (
      <html>
        <body>
          <main <mark>dangerouslySetInnerHTML</mark>={{__html: main}} />
          <script src='/bundle.js'></script>
        </body>
      </html>
    );
  }
};
```

The `dangerouslySetInnerHTML` property has the noble intention to warn you against injecting unsanitized user input into your site, it's [documentation is very pompous][1] in a _"you'll probably get it wrong, so we made it a very weird API"_ way. Imagine if everything you could possibly get wrong had a shitty API just so you double check. Wouldn't it be better to _just educate developers_ about [XSS][2] and [user-input sanitization][3]? `</rant>`

## Setting the `<title>`

When it comes to universal rendering sometimes even the simplest things can become annoying to deal with. One of those things is setting the title of your web pages. Your engine of choice has to be able to set the title on the layout on the server-side directly and then be also able to change `document.title` dynamically on the client-side whenever the view changes. In the case of React I came across a small module that helps me achieve universal view titles with little effort.

First off, we'll be installing [`react-document-title`][4]. This is a React component you can include in your JSX templates to set the title.

```bash
npm i react-document-title -S
```

A nice aspect of how it works is that you can add as many `<DocumentTitle title='foo' />` tags in your component, and nest them as deep as you want. The `react-side-effect` library underlying `react-document-title` then figures out what tag was added last, and returns that when `DocumentTitle.rewind()` gets called.

Given that this works across all components, we are able to pull the `title` that was defined within one of the partial views, and then we can apply it to the document directly on the server-side.

```js
import React from 'react';
<mark>import DocumentTitle from 'react-document-title';</mark>

export default class Layout extends React.Component {
  render () {
    const { main } = this.props;

    return (
      <html>
        <head>
          <title><mark>{DocumentTitle.rewind()}</mark></title>
        </head>
        <body>
          <main dangerouslySetInnerHTML={{__html: main}} />
          <script src='/bundle.js'></script>
        </body>
      </html>
    );
  }
};
```

When it comes to client-side rendering, [`react-document-title`][4] will just change `document.title`. If you wanted to add a default title to the pages in case no other component defines a title, you could do it at the `<App />` level, changing that component as follows:

```js
import React from 'react';
import {RouteHandler} from 'react-router';
import DocumentTitle from 'react-document-title';

export default class App extends React.Component {
  render () {
    return <mark><DocumentTitle title='Pony Foo'></mark>
      <RouteHandler />
    </DocumentTitle>
  }
}
```

_Ta-da!_ That was pretty easy. You could also use [`react-doc-meta`][5] to do the same about `<meta>` tags.

[1]: https://facebook.github.io/react/tips/dangerously-set-inner-html.html "'Dangerously Set innerHTML' documentation on React's website"
[2]: https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet "XSS Prevention Cheat Sheet"
[3]: http://www.smashingmagazine.com/2011/01/keeping-web-users-safe-by-sanitizing-input-data/ "Keeping Web Users Safe By Sanitizing Input Data"
[4]: https://github.com/gaearon/react-document-title "gaearon/react-document-title"
[5]: https://github.com/geekyme/react-doc-meta "geekyme/react-doc-meta"
