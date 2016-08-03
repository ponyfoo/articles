<div></div>

<h1>Uncovering the Native DOM API</h1>

<p><kbd>js</kbd> <kbd>js-native</kbd> <kbd>dom</kbd> <kbd>best-practices</kbd></p>

<blockquote><p>JavaScript libraries such as <strong>jQuery</strong> serve a great purpose in enabling and <em>normalizing cross-browser behaviors</em> of the <a href="https://developer.mozilla.org/en/docs/DOM" target="_blank">DOM</a> in such a way that it&#x2019;s possible to use &#x2026;</p></blockquote>

<div><p>JavaScript libraries such as <strong>jQuery</strong> serve a great purpose in enabling and <em>normalizing cross-browser behaviors</em> of the <a href="https://developer.mozilla.org/en/docs/DOM" target="_blank">DOM</a> in such a way that it&#x2019;s possible to use the same interface to interact with many different browsers.</p></div>

<div></div>

<div><p>But they do so at a price. And that price, in the case of some developers, is having no idea what the heck the library is actually doing when we use it.</p> <blockquote> <p>Heck, it works! Right? Well, <em>no</em>. You should know what happens behind the scenes, in order to better <em>understand what you are doing</em>. Otherwise, you would be just <a href="http://pragprog.com/the-pragmatic-programmer/extracts/coincidence" target="_blank">programming by coincidence</a>.</p> </blockquote> <p>I&#x2019;ll help you explore some of the parts of the <strong>DOM API</strong> that are usually abstracted away behind a little neat interface in your library of choice. Lets kick off with AJAX.</p></div>

<div><h1 id="meet-xmlhttprequest">Meet: XMLHttpRequest</h1> <p>Surely you know how to write AJAX requests, right? Probably something like&#x2026;</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">$.ajax({
    url: <span class="md-code-string">&apos;/endpoint&apos;</span>
}).done(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(data)</span></span>{
    <span class="md-code-comment">// do something awesome</span>
}).fail(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(xhr)</span></span>{
    <span class="md-code-comment">// sad little dance</span>
});
</code></pre> <p>How do we write that with <em>native browser-level toothless JavaScript</em>?</p> <p>We could start by looking it up on <a href="https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest" target="_blank" aria-label="XMLHttpRequest - MDN">MDN</a>. XMLHttpRequest is right on <em>one count</em>. It&#x2019;s for performing requests. But they can <em>manipulate any data</em>, not just XML. They also aren&#x2019;t limited to just the <a href="http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol" target="_blank" aria-label="Hyper Text Transfer Protocol">HTTP protocol</a>.</p> <p><em>XMLHttpRequest</em> is what makes AJAX sprinkle magic all over rich internet applications nowadays. They are, admitedly, <strong>kind of hard to get right</strong> without looking it up, or <em>having prepared to use them for an interview</em>.</p> <p>Lets give it <em>a first try</em>:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> xhr = <span class="md-code-keyword">new</span> XMLHttpRequest();
xhr.onreadystatechange = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
    <span class="md-code-keyword">var</span> completed = <span class="md-code-number">4</span>;
    <span class="md-code-keyword">if</span>(xhr.readyState === completed){
        <span class="md-code-keyword">if</span>(xhr.status === <span class="md-code-number">200</span>){
            <span class="md-code-comment">// do something with xhr.responseText</span>
        }<span class="md-code-keyword">else</span>{
            <span class="md-code-comment">// handle the error</span>
        }
    }
};
xhr.open(<span class="md-code-string">&apos;GET&apos;</span>, <span class="md-code-string">&apos;/endpoint&apos;</span>, <span class="md-code-literal">true</span>);
xhr.send(<span class="md-code-literal">null</span>);
</code></pre> <p>You can try this in a pen I made <a href="http://cdpn.io/ycgzo" target="_blank" aria-label="Bare XMLHttpRequest">here</a>. Before we get into what I actually did in the pen, we should go over the snippet I wrote here, making sure we didn&#x2019;t miss anything.</p> <p>The <code class="md-code md-code-inline">.onreadystatechange</code> handler will fire every time <code class="md-code md-code-inline">xhr.readyState</code> changes, but the only state that&#x2019;s really relevant is <code class="md-code md-code-inline">4</code>, a <em>magic number</em> that denotes an XHR request is <em>complete</em>, whatever the outcome was.</p> <p>Once the request is complete, the XHR object will have it&#x2019;s <code class="md-code md-code-inline">status</code> filled. If you try to access <code class="md-code md-code-inline">status</code> <em>before completion</em>, you might <a href="http://stackoverflow.com/a/15623060/389745" target="_blank" aria-label="Why does it throw?">get an exception</a>.</p> <p>Lastly, when you know the <code class="md-code md-code-inline">status</code> of your XHR request, you can do something about it, you should use <code class="md-code md-code-inline">xhr.responseText</code> to figure out <em>how to react</em> to the response, probably passing that to a callback.</p> <p>The request is prepared using <code class="md-code md-code-inline">xhr.open</code>, passing the <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html" target="_blank" aria-label="HTTP/1.1 Method Definitions">HTTP method</a> in the first parameter, the resource to query in the second parameter, and a third parameter to decide whether the request should be asynchronous (<code class="md-code md-code-inline">true</code>), or block the UI thread and make everyone cry (<code class="md-code md-code-inline">false</code>).</p> <p>If you also want to send some data, you should pass that to the <code class="md-code md-code-inline">xhr.send</code>. This function actually <a href="https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest#send()" target="_blank" aria-label="XMLHttpRequest send - MDN">sends the request</a> and it supports all the signatures below.</p> <pre class="md-code-block"><code class="md-code">void send();
void send(ArrayBuffer data);
void send(Blob data);
void send(Document data);
void send(DOMString? data);
void send(FormData data);
</code></pre> <p>I won&#x2019;t go into detail, but you&#x2019;d use those signatures to send data to the server.</p> <p>A sensible way to wrap our native XHR call in a reusable function might be the following:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">ajax</span><span class="md-code-params">(url, opts)</span></span>{
    <span class="md-code-keyword">var</span> xhr = <span class="md-code-keyword">new</span> XMLHttpRequest();
    xhr.onreadystatechange = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
        <span class="md-code-keyword">var</span> completed = <span class="md-code-number">4</span>;
        <span class="md-code-keyword">if</span>(xhr.readyState === completed){
            <span class="md-code-keyword">if</span>(xhr.status === <span class="md-code-number">200</span>){
                opts.success(xhr.responseText, xhr);
            }<span class="md-code-keyword">else</span>{
                opts.error(xhr.responseText, xhr);
            }
        }
    };
    xhr.open(opts.method, url, <span class="md-code-literal">true</span>);
    xhr.send(opts.data);
}

ajax(<span class="md-code-string">&apos;/foo&apos;</span>, { <span class="md-code-comment">// usage</span>
    method: <span class="md-code-string">&apos;GET&apos;</span>,
    success: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(response)</span></span>{
        <span class="md-code-built_in">console</span>.log(response);
    },
    error: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(response)</span></span>{
        <span class="md-code-built_in">console</span>.log(response);
    }
});
</code></pre> <p>You might want to add <em>default values</em> to the <code class="md-code md-code-inline">method</code>, <code class="md-code md-code-inline">success</code> and <code class="md-code md-code-inline">error</code> options, maybe even use <a href="https://ponyfoo.com/2013/05/08/taming-asynchronous-javascript" aria-label="Taming Asynchronous JavaScript">promises</a>, but it should be enough to <em>get you going</em>.</p> <p>Next up, events!</p> <h1 id="event-listeners">Event Listeners</h1> <p>Lets say you now want to attach that awesome AJAX call to one your DOM elements, that&#x2019;s ridiculously easy!</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">$(<span class="md-code-string">&apos;button&apos;</span>).on(<span class="md-code-string">&apos;click&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
    ajax( ... );
});
</code></pre> <p>Sure, you could use jQuery like your life depended on it, but this one is pretty simple to do with &#x2018;pure&#x2019; JS. Lets try a reusable function from the get-go.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">add</span><span class="md-code-params">(element, type, handler)</span></span>{
    <span class="md-code-keyword">if</span> (element.addEventListener){
        element.addEventListener(type, handler, <span class="md-code-literal">false</span>);
    }<span class="md-code-keyword">else</span> <span class="md-code-keyword">if</span> (element.attachEvent){
        element.attachEvent(<span class="md-code-string">&apos;on&apos;</span> + type, handler); 
    }<span class="md-code-keyword">else</span>{
        <span class="md-code-comment">// more on this later</span>
    }
}

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">remove</span><span class="md-code-params">(element, type, handler)</span></span>{
    <span class="md-code-keyword">if</span> (element.removeEventListener){
        element.removeEventListener(type, handler);
    }<span class="md-code-keyword">else</span> <span class="md-code-keyword">if</span> (element.detachEvent){
        element.detachEvent(type, handler);
    }<span class="md-code-keyword">else</span>{
        <span class="md-code-comment">// more on this later</span>
    }
}
</code></pre> <p>This one is pretty straightforward, you just add events with either the <a href="http://www.w3.org/TR/DOM-Level-2-Events/events.html" target="_blank" aria-label="DOM Events - W3C">W3C event model</a>, or the <a href="http://msdn.microsoft.com/en-us/library/ie/ms536343(v=vs.85).aspx" target="_blank" aria-label="IE Events - MSDN">IE event model</a>.</p> <p>The <a href="http://msdn.microsoft.com/en-us/library/ms533023(v=vs.85).aspx" target="_blank" aria-label="&apos;Understanding&apos; the Event Model - MSDN">last resort</a> would be to use <code class="md-code md-code-inline">element[&apos;on&apos; + type] = handler</code>, but this would be very bad because we wouldn&#x2019;t be able to attach more than one event to each DOM element.</p> <p>If corner cases are in your wheelhouse, we <em>could</em> use a dictionary to keep the handlers in a way that they are easy to add and remove. Then it would be just a matter of calling all of these handlers when an event is fired. This brings a <strong>whole host of complications</strong>, though:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">!<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(window)</span></span>{
    <span class="md-code-keyword">var</span> events = {}, map = [];

    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">add</span><span class="md-code-params">(element, type, handler)</span></span>{
        <span class="md-code-keyword">var</span> key = <span class="md-code-string">&apos;on&apos;</span> + type,
            id = uid(element),
            e = events[id];

        element[key] = eventStorm(element, type);

        <span class="md-code-keyword">if</span>(!e){
            e = events[id] = { handlers: {} };
        }

        <span class="md-code-keyword">if</span>(!e.handlers[type]){
            e.handlers[type] = [];
            e.handlers[type].active = <span class="md-code-number">0</span>;
        }

        e.handlers[type].push(handler);
        e.handlers[type].active++;
    }

    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">remove</span><span class="md-code-params">(element, type, handler)</span></span>{
        <span class="md-code-keyword">var</span> key = <span class="md-code-string">&apos;on&apos;</span> + type,
            e = events[uid(element)];

        <span class="md-code-keyword">if</span>(!e || !e.handlers[type]){
            <span class="md-code-keyword">return</span>;
        }
        
        <span class="md-code-keyword">var</span> handlers = e.handlers[type],
            index = handlers.indexOf(handler);

        <span class="md-code-comment">// delete it in place to avoid ordering issues</span>
        <span class="md-code-keyword">delete</span> handlers[index];
        handlers.active--;

        <span class="md-code-keyword">if</span> (handlers.active === <span class="md-code-number">0</span>){
            <span class="md-code-keyword">if</span> (element[key]){
                element[key] = <span class="md-code-literal">null</span>;
                e.handlers[type] = [];
            }
        }
    }

    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">eventStorm</span><span class="md-code-params">(element, type)</span></span>{
        <span class="md-code-keyword">return</span> <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
            <span class="md-code-keyword">var</span> e = events[uid(element)];
            <span class="md-code-keyword">if</span>(!e || !e.handlers[type]){
                <span class="md-code-keyword">return</span>;
            }
            
            <span class="md-code-keyword">var</span> handlers = e.handlers[type],
                len = handlers.length,
                i;

            <span class="md-code-keyword">for</span>(i = <span class="md-code-number">0</span>; i &lt; len; i++){
                <span class="md-code-comment">// check the handler wasn&apos;t removed</span>
                <span class="md-code-keyword">if</span> (handlers[i]){
                    handlers[i].apply(<span class="md-code-keyword">this</span>, <span class="md-code-built_in">arguments</span>);
                }
            }
        };
    }

    <span class="md-code-comment">// this is a fast way to identify our elements</span>
    <span class="md-code-comment">// .. at the expense of our memory, though.</span>
    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">uid</span><span class="md-code-params">(element)</span></span>{
        <span class="md-code-keyword">var</span> index = map.indexOf(element);
        <span class="md-code-keyword">if</span> (index === -<span class="md-code-number">1</span>){
            map.push(element);
            index = map.length - <span class="md-code-number">1</span>;
        }
        <span class="md-code-keyword">return</span> index;
    }

    <span class="md-code-built_in">window</span>.events = {
        add: add,
        remove: remove
    };
}(<span class="md-code-built_in">window</span>);
</code></pre> <p>You can glance at how this can very quickly get out of hand. Remember this was <em>just in the case of <strong>no W3C event model</strong>, and <strong>no IE event model</strong></em>. Fortunately, <em>this is <strong>largely unnecessary</strong> nowadays</em>. You can imagine how hacks of this kind are all over your favorite libraries.</p> <p>They have to be, if they want to support the old, decrepit and outdated browsers. Some have been <a href="http://blog.jquery.com/2012/06/28/jquery-core-version-1-9-and-beyond/" target="_blank" aria-label="jQuery Version 1.9 and Beyond - jQuery Blog">taking steps back</a> from the <em>support every single browser</em> philosophy.</p> <p>I encourage you to read your favorite library&#x2019;s <a href="http://code.jquery.com/jquery.js" target="_blank" aria-label="Latest Stable jQuery Source">code</a>, and learn how <em>they</em> resolve these situations, or how they are written in general.</p> <p>Moving along.</p> <h1 id="event-delegation">Event Delegation</h1> <blockquote> <p>What the heck is <strong>event delegation</strong>?, how am I even supposed to know <em>what it is</em>?</p> </blockquote> <p>This is the <strong>ever ubiquitous interview question</strong>. Yet, every single time I&#x2019;m asked this question during the course of an interview, the interviewers look surprised that I actually know what event delegation is. Other <a href="https://github.com/darcyclarke/Front-end-Developer-Interview-Questions" target="_blank" aria-label="Front End Developer Interview Questions">common interview questions</a> include event bubbling, event capturing, event propagation.</p> <p>Save the interview questions link in your <a href="http://getpocket.com/" target="_blank" aria-label="Pocket App">pocket</a>, and read it later to treat yourself to a little evaluation of your front-end development skills. It&#x2019;s good to know where you&#x2019;re standing.</p> <p>Now, onto the meat.</p> <p><img alt="raw-meat.jpg" title="Well, maybe not that" class="" src="https://i.imgur.com/UDGhrLQ.jpg"></p> <p>Event delegation is what you have to do when you have many elements which need the same event handler. It <em>doesn&#x2019;t matter</em> if the handler depends on the <em>actual</em> element, because event delegation accomodates for that.</p> <p>Lets look at a use case. I&#x2019;ll use the <a href="http://jade-lang.com/" target="_blank" aria-label="Jade template engine">Jade</a> syntax.</p> <pre class="md-code-block"><code class="md-code md-lang-css"><span class="md-code-tag">body</span>
    <span class="md-code-tag">ul</span><span class="md-code-class">.foo</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>

    <span class="md-code-tag">ul</span><span class="md-code-class">.foo</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>
        <span class="md-code-tag">li</span><span class="md-code-class">.bar</span>
</code></pre> <p>We want, for whatever reason, to attach an event handler to each <code class="md-code md-code-inline">.foo</code> element. The problem is that event listening is resource consuming. It&#x2019;s <em>lighter</em> to attach a single event than thousands. Yet, it&#x2019;s surprisingly common to work in codebases with <em>little to no event delegation</em>.</p> <p>A <em>better performing</em> approach is to add a <em>super event handler</em> on a node which is a parent to every node that wants to listen to that event using this handler. And then:</p> <ul> <li>When the event is raised on one of the children, it <a href="http://www.quirksmode.org/js/events_order.html" target="_blank" aria-label="JavaScript Event Order">bubbles up the DOM chain</a></li> <li>It reaches the parent node which has our <em>super handler</em>.</li> <li>That special handler will check whether the <a href="https://developer.mozilla.org/en-US/docs/Web/API/event.target" target="_blank" aria-label="event.target and window.event - MDN">event target</a> is one of the intended targets</li> <li>Finally the <em>actual handler</em> will be invoked, passing it the appropriate event context.</li> </ul> <p>This is what happens when you bind events using jQuery code such as:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">$(<span class="md-code-string">&apos;body&apos;</span>).on(<span class="md-code-string">&apos;click&apos;</span>, <span class="md-code-string">&apos;.bar&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
    <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;clicked bar!&apos;</span>, $(<span class="md-code-keyword">this</span>));
});
</code></pre> <p>As opposed to more unfortunate code:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">$(<span class="md-code-string">&apos;.bar&apos;</span>).on(<span class="md-code-string">&apos;click&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
    <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;clicked bar!&apos;</span>, $(<span class="md-code-keyword">this</span>));
});
</code></pre> <p>Which would work <em>pretty much the same way</em>, except it will create one event handler for each <code class="md-code md-code-inline">.bar</code> element, hindering performance.</p> <p>There is <strong>one crucial difference</strong>. Event handling done directly on a node works for just that node. Forever. Event delegation works on any children that meet the criteria provided, <code class="md-code md-code-inline">.bar</code> in this case. If you were to add more <code class="md-code md-code-inline">.bar</code> elements to your DOM, those would also match the criteria, and therefore be attached to the <em>super handler</em> we created in the past.</p> <p>I won&#x2019;t be providing an example on raw JavaScript event delegation, but at least you now understand how it works and what it is, and hopefully, you understood <em>why you <strong>need</strong> to use it</em>.</p> <p>We&#x2019;ve been mentioning selectors such as <code class="md-code md-code-inline">.bar</code> this whole time, but how does <em>that</em> work?</p> <h1 id="querying-the-dom">Querying the DOM</h1> <p>You might have heard of <a href="http://sizzlejs.com/" target="_blank" aria-label="Sizzle Selector Library">Sizzle</a>, the internal library jQuery uses as a selector engine. I don&#x2019;t particularly understand the internals of Sizzle, but you might want to <a href="https://github.com/jquery/sizzle/blob/master/dist/sizzle.js" target="_blank" aria-label="sizzle.js on GitHub">take a look around</a> their codebase.</p> <p>For the most part, it uses <code class="md-code md-code-inline">c.querySelector</code> and <code class="md-code md-code-inline">c.querySelectorAll</code>. These methods enjoy <a href="http://stackoverflow.com/questions/3856294/is-queryselector-supported-by-all-browsers" target="_blank" aria-label="Is querySelector supported by all browsers?">very good support accross browsers</a>.</p> <p>Sizzle performs optimizations such as picking whether to use <code class="md-code md-code-inline">c.getElementById</code>, <code class="md-code md-code-inline">c.getElementsByTagName</code>, <code class="md-code md-code-inline">c.getElementsByClassName</code>, or one of the <a href="https://developer.mozilla.org/en-US/docs/Web/API/Element.querySelector" target="_blank" aria-label="Element.querySelector - MDN">querySelector</a> functions. It also fixes inconsistencies in IE8, and some other cross-browser fixes.</p> <p>Other than that, querying the DOM is <em>pretty much done natively</em>.</p> <p>Lets turn to <a href="http://api.jquery.com/category/manipulation/" target="_blank" aria-label="DOM Manipulation - jQuery API docs">manipulation</a>.</p> <h1 id="dom-manipulation">DOM Manipulation</h1> <blockquote> <p>Manipulating the DOM is one of those things that is <em>remarkably important</em> to get right, and <em>strikingly easy</em> to get wrong.</p> </blockquote> <p>Everyone knows how to add nodes to the DOM, so I won&#x2019;t waste my time on that. Instead, I&#x2019;ll talk about <a href="https://developer.mozilla.org/en-US/docs/Web/API/document.createDocumentFragment" target="_blank" aria-label="document.createDocumentFragment - MDN">createDocumentFragment</a>.</p> <p><code class="md-code md-code-inline">document.createDocumentFragment</code> allows us to create a DOM structure that&#x2019;s not attached to the main DOM tree. This allows us to create nodes that only exist in memory, and helps us to <a href="http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/" target="_blank" aria-label="Reflows &amp; Repaints">avoid DOM reflowing</a>.</p> <p>Once our tree fragment is ready, we can attach it to the DOM. When we do, all the child nodes in the fragment are attached to the specified node.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> somewhere = <span class="md-code-built_in">document</span>.getElementById(<span class="md-code-string">&apos;here&apos;</span>),
    fragment = <span class="md-code-built_in">document</span>.createDocumentFragment(),
    i, foo;

<span class="md-code-keyword">for</span>(i = <span class="md-code-number">0</span>, i &lt; <span class="md-code-number">1000</span>; i++){
    foo = <span class="md-code-built_in">document</span>.createElement(<span class="md-code-string">&apos;div&apos;</span>);
    foo.innerText = i;
    fragment.appendChild(foo);
}
somewhere.appendChild(fragment);
</code></pre> <p><a href="http://cdpn.io/sweoB" target="_blank" aria-label="Document Fragment Usage">Pen here</a></p> <p>There&#x2019;s a cute post on DocumentFragments, written by <em>John Resig</em>, you might want to <a href="http://ejohn.org/blog/dom-documentfragments/" target="_blank" aria-label="DOM DocumentFragments">check out</a>.</p> <p>Given that we&#x2019;ve been talking about the DOM for a while, let me introduce you to the dark side of the DOM.</p> <h1 id="shadow-dom">Shadow DOM</h1> <p>A couple of years ago I got <a href="http://glazkov.com/2011/01/14/what-the-heck-is-shadow-dom/" target="_blank" aria-label="What the Heck is Shadow DOM?">introduced to the shadow DOM</a>. I had no idea it existed. You probably don&#x2019;t, either.</p> <p>In short, the shadow DOM is a part of the DOM that&#x2019;s inaccessible for the most part. JavaScript acts as if there&#x2019;s nothing there, and so does CSS. There are a few browser-specific shadow DOM elements you can style (on certain properties), but interaction with the shadow DOM is <em>very carefully limited</em> in general.</p> <p>If you&#x2019;ve gotten this far, and happen to be looking for a job, <a href="https://github.com/bevacqua/frontend-job-listings" target="_blank" aria-label="Front End Job Listings">this link</a> might help you in your search.</p> <p>A follow-up to this article can be found here: <a href="http://blog.ponyfoo.com/2013/07/09/getting-over-jquery" target="_blank" aria-label="Getting Over jQuery on Pony Foo">Getting Over jQuery</a></p></div>
