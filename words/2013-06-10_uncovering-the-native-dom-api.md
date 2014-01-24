# Uncovering the Native DOM API #

JavaScript libraries such as **jQuery** serve a great purpose in enabling and _normalizing cross-browser behaviors_ of the [DOM](https://developer.mozilla.org/en/docs/DOM "Document Object Model") in such a way that it's possible to use the same interface to interact with many different browsers.

But they do so at a price. And that price, in the case of some developers, is having no idea what the heck the library is actually doing when we use it.

> Heck, it works! Right? Well, _no_. You should know what happens behind the scenes, in order to better _understand what you are doing_. Otherwise, you would be just [programming by coincidence](http://pragprog.com/the-pragmatic-programmer/extracts/coincidence "Programming by Coincidence - Pragmatic Bookshelf").

I'll help you explore some of the parts of the **DOM API** that are usually abstracted away behind a little neat interface in your library of choice. Lets kick off with AJAX.

# Meet: XMLHttpRequest #

Surely you know how to write AJAX requests, right? Probably something like...

```js
$.ajax({
    url: '/endpoint'
}).done(function(data){
    // do something awesome
}).fail(function(xhr){
    // sad little dance
});
```

How do we write that with _native browser-level toothless JavaScript_?

We could start by looking it up on [MDN](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest "XMLHttpRequest - MDN"). XMLHttpRequest is right on _one count_. It's for performing requests. But they can _manipulate any data_, not just XML. They also aren't limited to just the [HTTP protocol](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol "Hyper Text Transfer Protocol").

_XMLHttpRequest_ is what makes AJAX sprinkle magic all over rich internet applications nowadays. They are, admitedly, **kind of hard to get right** without looking it up, or _having prepared to use them for an interview_.

Lets give it _a first try_:

```js
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function(){
    var completed = 4;
    if(xhr.readyState === completed){
        if(xhr.status === 200){
            // do something with xhr.responseText
        }else{
            // handle the error
        }
    }
};
xhr.open('GET', '/endpoint', true);
xhr.send(null);
```

You can try this in a pen I made [here](http://cdpn.io/ycgzo "Bare XMLHttpRequest"). Before we get into what I actually did in the pen, we should go over the snippet I wrote here, making sure we didn't miss anything.

The `.onreadystatechange` handler will fire every time `xhr.readyState` changes, but the only state that's really relevant is `4`, a _magic number_ that denotes an XHR request is _complete_, whatever the outcome was.

Once the request is complete, the XHR object will have it's `status` filled. If you try to access `status` _before completion_, you might [get an exception](http://stackoverflow.com/a/15623060/389745 "Why does it throw?").

Lastly, when you know the `status` of your XHR request, you can do something about it, you should use `xhr.responseText` to figure out _how to react_ to the response, probably passing that to a callback.

The request is prepared using `xhr.open`, passing the [HTTP method](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html "HTTP/1.1 Method Definitions") in the first parameter, the resource to query in the second parameter, and a third parameter to decide whether the request should be asynchronous (`true`), or block the UI thread and make everyone cry (`false`).

If you also want to send some data, you should pass that to the `xhr.send`. This function actually [sends the request][xhrsend] and it supports all the signatures below.

```
void send();
void send(ArrayBuffer data);
void send(Blob data);
void send(Document data);
void send(DOMString? data);
void send(FormData data);
```

I won't go into detail, but you'd use those signatures to send data to the server.

A sensible way to wrap our native XHR call in a reusable function might be the following:

```js
function ajax(url, opts){
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function(){
        var completed = 4;
        if(xhr.readyState === completed){
            if(xhr.status === 200){
                opts.success(xhr.responseText, xhr);
            }else{
                opts.error(xhr.responseText, xhr);
            }
        }
    };
    xhr.open(opts.method, url, true);
    xhr.send(opts.data);
}

ajax('/foo', { // usage
    method: 'GET',
    success: function(response){
        console.log(response);
    },
    error: function(response){
        console.log(response);
    }
});
```

You might want to add _default values_ to the `method`, `success` and `error` options, maybe even use [promises](/2013/05/08/taming-asynchronous-javascript "Taming Asynchronous JavaScript"), but it should be enough to _get you going_.

Next up, events!

# Event Listeners #

Lets say you now want to attach that awesome AJAX call to one your DOM elements, that's ridiculously easy!

```js
$('button').on('click', function(){
    ajax( ... );
});
```

Sure, you could use jQuery like your life depended on it, but this one is pretty simple to do with 'pure' JS. Lets try a reusable function from the get-go.

```js
function add(element, type, handler){
    if (element.addEventListener){
        element.addEventListener(type, handler, false);
    }else if (element.attachEvent){
        element.attachEvent('on' + type, handler); 
    }else{
        // more on this later
    }
}

function remove(element, type, handler){
    if (element.removeEventListener){
        element.removeEventListener(type, handler);
    }else if (element.detachEvent){
        element.detachEvent(type, handler);
    }else{
        // more on this later
    }
}
```

This one is pretty straightforward, you just add events with either the [W3C event model](http://www.w3.org/TR/DOM-Level-2-Events/events.html "DOM Events - W3C"), or the [IE event model][iemodel].

The [last resort][understandingie] would be to use `element['on' + type] = handler`, but this would be very bad because we wouldn't be able to attach more than one event to each DOM element.

If corner cases are in your wheelhouse, we _could_ use a dictionary to keep the handlers in a way that they are easy to add and remove. Then it would be just a matter of calling all of these handlers when an event is fired. This brings a **whole host of complications**, though:

```js
!function(window){
    var events = {}, map = [];

    function add(element, type, handler){
        var key = 'on' + type,
            id = uid(element),
            e = events[id];

        element[key] = eventStorm(element, type);

        if(!e){
            e = events[id] = { handlers: {} };
        }

        if(!e.handlers[type]){
            e.handlers[type] = [];
            e.handlers[type].active = 0;
        }

        e.handlers[type].push(handler);
        e.handlers[type].active++;
    }

    function remove(element, type, handler){
        var key = 'on' + type,
            e = events[uid(element)];

        if(!e || !e.handlers[type]){
            return;
        }
        
        var handlers = e.handlers[type],
            index = handlers.indexOf(handler);

        // delete it in place to avoid ordering issues
        delete handlers[index];
        handlers.active--;

        if (handlers.active === 0){
            if (element[key]){
                element[key] = null;
                e.handlers[type] = [];
            }
        }
    }

    function eventStorm(element, type){
        return function(){
            var e = events[uid(element)];
            if(!e || !e.handlers[type]){
                return;
            }
            
            var handlers = e.handlers[type],
                len = handlers.length,
                i;

            for(i = 0; i < len; i++){
                // check the handler wasn't removed
                if (handlers[i]){
                    handlers[i].apply(this, arguments);
                }
            }
        };
    }

    // this is a fast way to identify our elements
    // .. at the expense of our memory, though.
    function uid(element){
        var index = map.indexOf(element);
        if (index === -1){
            map.push(element);
            index = map.length - 1;
        }
        return index;
    }

    window.events = {
        add: add,
        remove: remove
    };
}(window);
```

You can glance at how this can very quickly get out of hand. Remember this was _just in the case of **no W3C event model**, and **no IE event model**_. Fortunately, _this is **largely unnecessary** nowadays_. You can imagine how hacks of this kind are all over your favorite libraries.

They have to be, if they want to support the old, decrepit and outdated browsers. Some have been [taking steps back](http://blog.jquery.com/2012/06/28/jquery-core-version-1-9-and-beyond/ "jQuery Version 1.9 and Beyond - jQuery Blog") from the _support every single browser_ philosophy.

I encourage you to read your favorite library's [code](http://code.jquery.com/jquery.js "Latest Stable jQuery Source"), and learn how _they_ resolve these situations, or how they are written in general.

Moving along.

# Event Delegation #

> What the heck is **event delegation**?, how am I even supposed to know _what it is_?

This is the **ever ubiquitous interview question**. Yet, every single time I'm asked this question during the course of an interview, the interviewers look surprised that I actually know what event delegation is. Other [common interview questions](https://github.com/darcyclarke/Front-end-Developer-Interview-Questions "Front End Developer Interview Questions") include event bubbling, event capturing, event propagation.

Save the interview questions link in your [pocket](http://getpocket.com "Pocket App"), and read it later to treat yourself to a little evaluation of your front-end development skills. It's good to know where you're standing.

Now, onto the meat.

![raw-meat.jpg][1]

Event delegation is what you have to do when you have many elements which need the same event handler. It _doesn't matter_ if the handler depends on the _actual_ element, because event delegation accomodates for that.

Lets look at a use case. I'll use the [Jade](http://jade-lang.com/ "Jade template engine") syntax.

```jade
body
    ul.foo
        li.bar
        li.bar
        li.bar
        li.bar

    ul.foo
        li.bar
        li.bar
        li.bar
        li.bar
```

We want, for whatever reason, to attach an event handler to each `.foo` element. The problem is that event listening is resource consuming. It's _lighter_ to attach a single event than thousands. Yet, it's surprisingly common to work in codebases with _little to no event delegation_.

A _better performing_ approach is to add a _super event handler_ on a node which is a parent to every node that wants to listen to that event using this handler. And then:

- When the event is raised on one of the children, it [bubbles up the DOM chain](http://www.quirksmode.org/js/events_order.html "JavaScript Event Order")
- It reaches the parent node which has our _super handler_.
- That special handler will check whether the [event target](https://developer.mozilla.org/en-US/docs/Web/API/event.target "event.target and window.event - MDN") is one of the intended targets
- Finally the _actual handler_ will be invoked, passing it the appropriate event context.

This is what happens when you bind events using jQuery code such as:

```js
$('body').on('click', '.bar', function(){
    console.log('clicked bar!', $(this));
});
```

As opposed to more unfortunate code:

```js
$('.bar').on('click', function(){
    console.log('clicked bar!', $(this));
});
```

Which would work _pretty much the same way_, except it will create one event handler for each `.bar` element, hindering performance.

There is **one crucial difference**. Event handling done directly on a node works for just that node. Forever. Event delegation works on any children that meet the criteria provided, `.bar` in this case. If you were to add more `.bar` elements to your DOM, those would also match the criteria, and therefore be attached to the _super handler_ we created in the past.

I won't be providing an example on raw JavaScript event delegation, but at least you now understand how it works and what it is, and hopefully, you understood _why you **need** to use it_.

We've been mentioning selectors such as `.bar` this whole time, but how does _that_ work?

# Querying the DOM #

You might have heard of [Sizzle](http://sizzlejs.com/ "Sizzle Selector Library"), the internal library jQuery uses as a selector engine. I don't particularly understand the internals of Sizzle, but you might want to [take a look around](https://github.com/jquery/sizzle/blob/master/dist/sizzle.js "sizzle.js on GitHub") their codebase.

For the most part, it uses `c.querySelector` and `c.querySelectorAll`. These methods enjoy [very good support accross browsers](http://stackoverflow.com/questions/3856294/is-queryselector-supported-by-all-browsers "Is querySelector supported by all browsers?").

Sizzle performs optimizations such as picking whether to use `c.getElementById`, `c.getElementsByTagName`, `c.getElementsByClassName`, or one of the [querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Element.querySelector "Element.querySelector - MDN") functions. It also fixes inconsistencies in IE8, and some other cross-browser fixes.

Other than that, querying the DOM is _pretty much done natively_.

Lets turn to [manipulation](http://api.jquery.com/category/manipulation/ "DOM Manipulation - jQuery API docs").

# DOM Manipulation #

> Manipulating the DOM is one of those things that is _remarkably important_ to get right, and _strikingly easy_ to get wrong.

Everyone knows how to add nodes to the DOM, so I won't waste my time on that. Instead, I'll talk about [createDocumentFragment](https://developer.mozilla.org/en-US/docs/Web/API/document.createDocumentFragment "document.createDocumentFragment - MDN").

`document.createDocumentFragment` allows us to create a DOM structure that's not attached to the main DOM tree. This allows us to create nodes that only exist in memory, and helps us to [avoid DOM reflowing](http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/ "Reflows & Repaints"). 

Once our tree fragment is ready, we can attach it to the DOM. When we do, all the child nodes in the fragment are attached to the specified node.

```js
var somewhere = document.getElementById('here'),
    fragment = document.createDocumentFragment(),
    i, foo;

for(i = 0, i < 1000; i++){
    foo = document.createElement('div');
    foo.innerText = i;
    fragment.appendChild(foo);
}
somewhere.appendChild(fragment);
```

[Pen here](http://cdpn.io/sweoB "Document Fragment Usage")

There's a cute post on DocumentFragments, written by _John Resig_, you might want to [check out](http://ejohn.org/blog/dom-documentfragments/ "DOM DocumentFragments").

Given that we've been talking about the DOM for a while, let me introduce you to the dark side of the DOM.

# Shadow DOM #

A couple of years ago I got [introduced to the shadow DOM](http://glazkov.com/2011/01/14/what-the-heck-is-shadow-dom/ "What the Heck is Shadow DOM?"). I had no idea it existed. You probably don't, either.

In short, the shadow DOM is a part of the DOM that's inaccessible for the most part. JavaScript acts as if there's nothing there, and so does CSS. There are a few browser-specific shadow DOM elements you can style (on certain properties), but interaction with the shadow DOM is _very carefully limited_ in general.

If you've gotten this far, and happen to be looking for a job, [this link](https://github.com/bevacqua/frontend-job-listings "Front End Job Listings") might help you in your search.

  [1]: http://i.imgur.com/UDGhrLQ.jpg "Well, maybe not that"
  [xhrsend]: https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest#send() "XMLHttpRequest send - MDN"
  [iemodel]: http://msdn.microsoft.com/en-us/library/ie/ms536343(v=vs.85).aspx "IE Events - MSDN"
  [understandingie]: http://msdn.microsoft.com/en-us/library/ms533023(v=vs.85).aspx "'Understanding' the Event Model - MSDN"