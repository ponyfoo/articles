# JSX is the new XHTML

Web-oriented template engines typically consist of an entirely new language _-- [Jade][1], [Dust.js][2], etc --_ or a few extensions on top of HTML _-- [Mustache][3], [Hogan][4], [Nunjucks][5], etc_. We rarely see hybrids like JSX, a language that's more or less aligned with XHTML but which also comes with some gross limitations in what you're able to do with it, alongside "cool" features like being able to mix it with JavaScript code. In that last aspect, it's sort of like [Jade][1], which you can also mix with JavaScript, but I'll admit that mixing JavaScript and JSX is way cleaner than mixing JavaScript with Jade.

React being one of the communities that push ES6 most aggressively, I would've liked to see them implement views largely based around ES6 string interpolation. Of course, JSX was created so that you don't have to use React's virtual DOM API, which can get very verbose with _a method call for each DOM element._ But still, it would've been nice if they took a more ES6*ish* approach. And boy is JSX weird. In fact, the point about DOM elements brings me to an unfortunate side effect of JSX.

## Unexpected and automatic DOM element insertion

As a newcomer to React, the fact that this:

```html
<span>foo {'bar'} {'baz'}</span>
```

Is turned into that:

```html
<span data-reactid=".1gm29bnrabk.1.0">
  <span data-reactid=".1gm29bnrabk.1.0.0">foo </span>
  <span data-reactid=".1gm29bnrabk.1.0.1">bar</span>
  <span data-reactid=".1gm29bnrabk.1.0.2"> </span>
  <span data-reactid=".1gm29bnrabk.1.0.3">baz</span>
</span>
```

Makes absolutely no sense to me. I'll just go ahead and assume that this makes the life of whoever maintains the virtual DOM implementation in React easier, but they should've considered [using text nodes][6]. You might think this is not a big deal, but it is if we're styling `<span>` elements in a certain way. In particular if you just expected the span you actually wrote to be the only `<span>`, and defined a style such as `font-size: 1.1em`, or even `padding: 5px`.

> Besides, isn't the whole point of React to avoid as many DOM operations as possible?

## Using conditionals in your view components

Whenever you're dealing with dynamic representations of data, particularly optional user-entered information such as the metadata about a product in an e-commerce application, you'll want some sort of way to conditionally render a component.

For some reason, React's JSX make this [unnecessarily complicated][9] by not being able to use `if` statements inside code blocks. The advertised solution is to place your conditionals outside of the template, in the component's `render` method. Here's a piece of code showing the coding style they recommend.

```js
var loginButton;
if (loggedIn) {
  loginButton = <LogoutButton />;
} else {
  loginButton = <LoginButton />;
}

return (
  <nav>
    <Home />
    {loginButton}
  </nav>
);
```

Note that, in case you just wanted the `if` leg _(no `else`)_, you could leave `loginButton` as `undefined` and nothing would be rendered. The problem here is that you end up having to hoist parts of your component for no reason other than what is, plainly put, **a limitation of the JSX language**.

A _(still terrible)_ workaround that allows you to place conditional logic into the template is to use binary operators. For some other reason, those _are_ supported by JSX. Of course, this is sad to look at, slightly confusing, and not as clear as plain `if` / `else` would have been.

```js
return (
  <nav>
    <Home />
    { loggedIn && <LogoutButton /> || <LoginButton /> }
  </nav>
);
```

## Declaring a doctype

Apparently declaring a `<doctype>` [is impossible][7] when it comes to React. I ended up with an unimpressive `'<!doctype html>' + layout` concatenation. This would be fine if we were just doing client-side rendering but we're trying to get to a shared rendering application here, kind of the whole point of using a library like React.

Concatenating the `<doctype>` on the server-side will have to do for now. Luckily, it doesn't break the diffing algorithm when the client-side code boots the application state.

## Declaring HTML comments

HTML comments are similarly hard to shove into a JSX template. Granted, this isn't something you want to add to a document very often, but it's still important to be able to declare some when you actually need to. If you want HTML comments in your JSX very badly, you can use the method below _(derived from [this blog post][8])_.

```js
var comments = `<!--[if lte IE 8]>
  <script src="/js/html5shim.js"></script>
<![endif]-->`
```
```html
<head dangerouslySetInnerHTML={{__html: comments}} />
```

Which, _OBVIOUSLY_ brings us to my next point.

## React's "smart" `dangerouslySetInnerHTML` API

I definitely can relate to the notion of attempting to help your consumers not to make mistakes, but this API borders on **pompous and presumptuous**. The [documentation rightly asserts][10] that misuse of unescaped HTML might result in **XSS vulnerabilities** and that you should sanitize user input. A public API method's name is not the place where to tell people everything that could go wrong with it, though.

Angular used to have a similar `ng-bind-html-unsafe`, which at least wasn't as presumptuous. It has since been deprecated in favor of [`ng-bind-html`][11], which is a saner version of React's implementation. When passed a string, it'll become sanitized, and if you want it to be unescaped you just tell it through their `$sce.trustAsHtml(string)` service. Of course, in that respect it might've been better to just let the user pass in something like `{ unescaped: 'some <strong>html</strong' }` instead of having to go through a service, but it's good enough -- at least it's not as condescending, and I don't feel like throwing up whenever I'm using the API.

## Components coupled to client-side code and ES6

Another big issue in my understanding of React is the way how components are loaded. Pretty much the entire React codebase is loaded on the server-side, and that means that a lot of libraries which are meant for the client-side of your application will be loaded on the server. This becomes an issue whenever you are loading a module that accesses the DOM before you even use it, -- usually happens when a library does feature testing to decide the API they'll export -- as that'll result in a thrown exception on the server-side _(as there is no DOM to be accessed)_.

You could work around this by using `require` statements instead of `import`, and requiring those client-side-only modules on a method like `componentDidMount`, which only get executed in the client-side. ES6 `import` statements need to be defined at the top level of your module definition, though, which means you can't work around this one using plain ES6.

This can be really problematic as even a test like `modern = !!window.document.addEventListener` would break on the server-side, and there's plenty of client-side libraries that do this before you call any methods on them.

## Wrapping Up

I've yet to play around with `redux` and client-side routing -- two big things that I'll be playing with next and which might change my mind about a few of the points I've made here. I've only played around with React lightly, so take the article with a grain of _"React onboarding experience areas of improvement"_ salt.

The new site is now up at [bevacqua.io][12], and while I definitely didn't need to use React to put it together, it's always interesting to try out new pieces of front-end technology. I don't _only have_ ranty things to say about JSX, and I'm sure it'll grow on me as I use it for more stateful applications and combined with `redux`.

ES6 has been fun so far, and I find myself gradually adopting some of the language features, as well as writing less semicolons. What's hard sometimes is to decide when to use an ES6 feature such as arrow functions over a named function declaration. In general I've been taking the approach of defaulting to the ES5 alternative, because I don't want to be littering my code with ES6 expressions just for the sake of it.

As a fun fact, here's [a snippet of code][13] I felt happy to write using lots of ES6 and the experimental `::` ES7 [function bind][14] syntax. Usually there's lots of named method involved in this sort of code, or `async.apply` _(or `contra.curry`)_, and ES6 definitely made my code cleaner on this one.

```js
concurrent({
  repo: next => query('/repos/' + repo, next),
  master: next => query('/repos/' + repo + '/branches/master', next)
}, ::this.pulledRepo)
```

<sub>_By the way concurrent is a method from [`contra`][15], equivalent to [`async.parallel`][16]._</sub>

Yay!

[1]: http://jade-lang.com/ "Jade Template Engine"
[2]: http://www.dustjs.com/ "Dust.js by LinkedIn"
[3]: https://mustache.github.io/ "{{ mustache }}"
[4]: http://twitter.github.io/hogan.js/ "JavaScript Templating from Twitter"
[5]: http://mozilla.github.io/nunjucks/ "Nunjucks from Mozilla"
[6]: https://developer.mozilla.org/en-US/docs/Web/API/Document/createTextNode "Document.createTextNode() – MDN"
[7]: https://github.com/facebook/react/issues/1035 "'Allow html conditional comments and doctype' on GitHub Issues"
[8]: https://nemisj.com/conditional-ie-comments-in-react-js/ "Conditional IE comments in React"
[9]: https://facebook.github.io/react/tips/if-else-in-JSX.html "If-Else in JSX"
[10]: https://facebook.github.io/react/tips/dangerously-set-inner-html.html "Dangerously Set innerHTML"
[11]: https://docs.angularjs.org/guide/migration#ngbindhtmlunsafe-has-been-removed-and-replaced-by-ngbindhtml "ngBindHtmlUnsafe has been removed and replaced by ngBindHtml – AngularJS Documentation"
[12]: http://bevacqua.io/ "My consulting site"
[13]: https://github.com/bevacqua/bevacqua.io/blob/a7b34539656a0d47f53971be38dcfff69ff690c9/components/opensource/project.js#L49-L52 "Source code for bevacqua.io site on GitHub"
[14]: http://babeljs.io/blog/2015/05/14/function-bind/ "Function Bind Syntax on Babel's Blog"
[15]: https://github.com/bevacqua/contra "bevacqua/contra on GitHub"
[16]: https://github.com/caolan/async "caolan/async on GitHub"
