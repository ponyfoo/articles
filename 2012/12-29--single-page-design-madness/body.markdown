# _Not so progressively_ enhanced #

> Hey, this is a technical blog, I don't need no stinkin' IE.

I'll support **IE 10**, but that's as far as I'm willing to go **right off the bat**. Things like `localStorage` and `history` are supported by [_IE8+_](http://caniuse.com/#search=localStorage "Can I Use localStorage?") and [_IE10+_](http://caniuse.com/#search=history "Can I Use history?") respectively, and I didn't want to spend my time patching behavior in earlier versions of _IE_ just yet.

When I first though about this project I wanted it to be [mobile first](http://www.amazon.com/dp/1937557022 "Mobile First"), but in order to publish a working version early in 2013 rather than later, I'll be dedicating myself to the [mobile aspect](http://phonegap.com/ "Phone Gap") later on, though I'm build the site with mobile in mind.

As I went along this past couple of days, I focused on two things.

- Front-end **design**, which got to a point where I'm pretty comfortable at
- The way in which I'm going to handle the fact that the page is going to behave (given that the idea is to use a **single-page** approach)

## Front-end design ##

On the HTML side of things, I'm really enjoying [jade](http://jade-lang.com/ "Jade Template Language"), it definitely sped up my productivity, and thirty minutes into development I felt really comfortable about it. I feel it's the perfect fit for _Node_. The single thing I like the most is how I write my classes and IDs in the same way I write CSS selectors. If you haven't yet, don't be as skeptical as I was about giving a try. It's _awesome_.

I thought about trying out [SASS](http://sass-lang.com/ "SASS CSS"), but I'm not really a fan of their propietary syntax, and [LESS](http://lesscss.org/ "LESS CSS") was so much easier to install. So [**LESS**](http://lesscss.org/ "LESS CSS") it is.

Other than that, _I'm not really a fan of using CSS frameworks_, so I did most of it by hand, or copying from older projects, which always comes in handy.

I tried to keep my CSS as reusable as posible, so I implemented common styles for my buttons, headings, and inputs, which will definitely help in any upcoming additions to the site.

Since I suck so much at picking color schemes, and the [MongoDB site](http://www.mongodb.org/ "MongoDB") is pretty, I took their color scheme for simplicity.

I didn't pick a code [prettify](http://code.google.com/p/google-code-prettify/ "prettify syntax highlighting") scheme yet for `code` blocks, mostly because I couldn't find a decent one.

> I'll settle on implementing a color scheme for code blocks by myself, once I get around to it though. It's not an immediate concern anyways.

# Client-side templates #

In the beginning I was certain I wanted some sort of lightweight templating engine to handle my single-page design requirements. I'm liking the solution at which I've arrived a lot, so I figured I'd write about it.

The entire site is laid out in a single page, which contains the layout and includes every _view template_ that can be rendered, but initially hidden.

For example, lets talk about the template to post a new blog entry. **entry.jade**

In the page, I include the template as such:

```jade
include templates/entry

script(src='/js/entry.js')
```

This is of course, not as scalable as I'd like it to be, but I didn't take the time, _yet_, to search for and implement a proper _asset management system_, which would be the appropriate solution to keep view templates _self contained_.

I'm not exactly sure going forward how I'm going to handle templates that actually require a view model, but for now, a **jade** template will do just fine.

This is, as of right now, **entry.jade**:

```jade
section#entry-template.template(data-class='entry-writing')
  form#entry-editor(method='POST',action='/write-entry')
    div#entry-title-editor
      label(for='entry-title')='Title'
      input#entry-title(type='text',name='entry.title')

    div#wmd-button-bar-brief.wmd-button-bar
    div
      textarea#wmd-input-brief.wmd-input.entry-brief(name='entry.brief')

    div#wmd-button-bar-text.wmd-button-bar
    div
      textarea#wmd-input-text.wmd-input.entry-text(name='entry.text')

    article.blog-entry
      header.blog-entry-title
      section#wmd-preview-brief.blog-entry-brief
      section#wmd-preview-text.blog-entry-text

    div#entry-editor-buttons
      input(type='submit',value='Post')
```

My templating engine will have to deal with _displaying_ or _hiding_ the template as required, through a simple yet powerful system I defined.

The client-side **markdown** implementation is not the topic in discussion, so I'll refrain from posting the entire **entry.js** for now, this is the snippet that registers the template with the templating engine.

```js
nbrut.tt.add({
    key: 'entry-editor',
    alias: '/write-entry',
    trigger: '#write-entry',
    source: '#entry-template',
    title: { value: 'New Post', formatted: true },
    onAfterActivate: onAfterActivate
});
```

This seemingly innocent function call does a few things. The most important of them are parsing the **DOM** for the template and storing it in a key value dictionary, alongside the provided settings.

```js
function read(template) {
    var s = $(template.source);
    if (s.length !== 1){
        throw new Error('template source not unique.');
    }
    var css = s.data('class');
    var html = s.remove().html();

    template.dom = {
        html: html,
        css: css
    };
}
```

Then it's simply stored in the dictionary:

```js
templates[settings.key] = settings;
```

And now we can _activate_ the template at any time, making it visible:

```js
nbrut.tt.activate('entry-editor');
```

Forcing people to use the _Javascript_ console isn't what I had in mind, so I naturally added a click handler to _activate_ the template:

```js
trigger.on('click', function(e){
    if (e.which === 1){ // left-click
        activate(settings.key);
        return false;
    }
});
```

The point of using the `which` property is that I'm a _huge_ fan of opening new tabs for just about anything using my mouse wheel, and it infuriates me when I can't do that.

Ultimately, when you click on the button, and a template is to be _activated_, a few things happen.

```js
function activate(key) {
    var template = templates[key];
    if (template === undefined) {
        template = templates['404']; // fall back to 404.
    }

    if(!template.initialized){
        template.initialized = true;
        template.initialize();
    }

    if(template.container in active) {
        if(active[template.container] === template && !template.selfCleanup) {
            return; // already active.
        } else {
            deactivateContainer(template.container); // clean-up.
        }
    }

    activateTemplate(template); // set-up.

    template.onAfterActivate();
}
```

We'll come back to the first if clause later. The template initialization is going to prove useful when some sort of one-time preparation for a particular template is required.

The third if clause introduces a new dictionary, the `active` dictionary. This one keeps track of which template is _active_ in each container managed by the templating engine. If the container is _active_, then it gets deactivated, which is a glorified way of saying the container is emptied.

If the template doesn't need to refresh itself when it's already active, we don't need to do anything.
    
Once the template is defined, initialized, and the container is ready, the template is set up in `activateTemplate`, and then any _post-activation callback_ that has been provided, executes.

```js
function activateTemplate(template){
    var c = $(template.container);
    if (c.length !== 1){
        throw new Error('template container not unique.');
    }
    c.html(template.dom.html);
    c.attr('class',template.dom.css);
    active[template.container] = template;

    if (template.container === defaults.container){
        var title = setTitle(template.title);
    }
}
```

There isn't a lot to mention about `activateTemplate`, except perhaps that the document title is only updated if `template.container === defaults.container`, which means a full _view render_ took place, instead of just a _partial render_.

## What now? History navigation ##

Now we have a _templating system_ to go back and forth between our different view templates, which is awesome (and incredibly _fast_), but this wouldn't be as fancy if it didn't let users navigate like they're used to, through the back and forward buttons in their browsers, so we need to implement **history navigation**.

There are a couple of aspects to this. First of all, since we are already _switching views_ on the _client side_, rather on the _server side_, we might as well do the routing on the client side:

```js
server.get('/*', function(req,res){
    res.render('index.jade');
});
```

> It's not like I'm going to get to [20 million monthly page views](http://theoatmeal.com/ "The Oatmeal") anytime soon, but that's **no excuse** for not improving the experience of the few adventuring ones that _dare navigate_ my site.

So now every single `GET` request that hits my site receives the same response. A single page containing templates that enable me to render any of my views. We have three things to take care of now:

- Dealing with **404** errors appropriately
- Keeping track of the user's **navigation history**
- **Landing page** that corresponds to the navigation history

To keep track of a user's navigation history, lets go back to the way in which we registered a view template:

```js
nbrut.tt.add({
    key: 'entry-editor',
    alias: '/write-entry',
    trigger: '#write-entry',
    source: '#entry-template',
    title: { value: 'New Post', formatted: true },
    onAfterActivate: onAfterActivate
});
```

I explained all of those properties, except for the _alias_. The alias would be the _route_ that we register with the HTML 5 [history](https://developer.mozilla.org/en-US/docs/DOM/Manipulating_the_browser_history "Manipulating the browser history") API.

So now we'll need to make a couple of subtle changes to the `activateTemplate` function. The first is adding an optional parameter I called `soft`. The second is:
   
```js
if (template.container === defaults.container){
    var title = setTitle(template.title);

    if(!soft){
        history.pushState(template.key, title, template.alias);
    }
}
```

We didn't change anything else, thus `!soft` will always be _truthy_. Each template activation will change the _navigation history_ with the _route sugar_ defined in `template.alias`.

In order to **preserve navigation**, since we're manipulating `history` with `history.pushState` we need to handle the `popstate` event, this encompasses two steps. The first is to handle `popstate`, so we'll go ahead and do that:

```js
$(function(){
    $(window).on('popstate', function(e){
        if (e.originalEvent === undefined || e.originalEvent.state === null){
            key = keys[document.location.pathname];
        } else {
            key = e.originalEvent.state;
        }
        activate(key, true);
    });

    $(window).trigger('popstate'); // manual trigger fixes an issue in Firefox.
});
```

Here we simply take the key from data passed to the `state`, or from the `document.location`, and pass that along to `activate`, with `soft = true`, meaning we won't be _pushing_ a new `state` this time.

The manual trigger fixes an [issue in Firefox](http://hacks.mozilla.org/2011/03/history-api-changes-in-firefox-4/ "history API in Firefox") where it wouldn't `popstate` on document load.

The second and last step was to add the `soft` parameter to `activate` as well, and to _prevent  identical_ `state` objects to be pushed.

```js
// ...

if(template.container in active) {
    if(active[template.container] === template) {
        if(!template.selfCleanup){
            return;
        }
        soft = true;
    } else {

// ...
```

And regarding the **HTTP 404** status error code won't really be ever happening in our application. That's fine, but we do need to _alert our users_ of the fact that a particular endpoint doesn't match any of the view templates that we can render. For this purpose, I created a very minimal **404** template:

    section#not-found-template.template
        h1='Not Found'
        div='Sorry, the page you are looking for does not exist in the Matrix.'

Now I just need to _register_ this template in my engine, like so:

```js
nbrut.tt.add({
    key: '404',
    source: '#not-found-template'
});
```

And _that's it_. Remember there was a _fallback_ to `template = templates['404'];`, that's all we'll ever need to deal with **404** issues.

# Coming Up #

In the [next post](/articles/javascript-javascript-javascript "Javascript Javascript Javascript") I'll delve into **MongoDB**, how to pair it with **Node**, and figuring out _how to communicate_ across all the application layers.
