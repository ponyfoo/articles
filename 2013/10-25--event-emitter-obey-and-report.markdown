<h1>Event Emitter: Obey and Report</h1>

<blockquote><p>The <em>event emitter pattern</em> was popularized by Node, and is made available in the browser by libraries such as <a href="https://github.com/hij1nx/EventEmitter2" target="_blank">EventEmitter2</a>. In practice, I haven&#x2019;t seen a lot of &#x2026;</p></blockquote>

<div><kbd>js</kbd> <kbd>pattern</kbd> <kbd>event-emitter</kbd></div>

<div><p>The <em>event emitter pattern</em> was popularized by Node, and is made available in the browser by libraries such as <a href="https://github.com/hij1nx/EventEmitter2" target="_blank">EventEmitter2</a>. In practice, I haven&#x2019;t seen a lot of use of event emitters in client-side applications, but lately I&#x2019;ve been applying it more and more in my code. Here, I&#x2019;ll introduce a modular approach to client-side development I didn&#x2019;t invent, but which I decided to name, anyways.</p></div>

<div></div>

<div></div>

<div><h1 id="obey-and-report">Obey and Report</h1> <p>Recently, I came up with this idea where I laid out a library that <em>doesn&#x2019;t necessarily depend on Angular</em>, but which plays very nicely with it. The pattern basically dictates that the components <strong>can only change their state through commands</strong>, which can come from either its UI directly, or be <em>programatically invoked</em>. This allows me to provide a default UI for the component, but is flexible enough to let an Angular application take over and replace that UI layer with its own, which then simply dictates the library to obey the commands it&#x2019;s sent. <strong>Reporting</strong> happens whenever a public state property is updated, and the UI could use said property to update its <em>representation</em> of the component&#x2019;s state.</p> <p><img alt="obey-report" title="Obey and Report Pattern" class="" src="https://i.imgur.com/64esjO6.png"></p> <p>One way to make this communication pattern effective is using an event emitter (some day <a href="http://updates.html5rocks.com/2012/11/Respond-to-change-with-Object-observe" target="_blank" aria-label="Respond to change with Object.observe">Object.observe</a> will be a <em>realistic</em> alternative), and emit events whenever our state changes. In code, we can make our library an event emitter like below. We&#x2019;ll keep state in a separate private variable so that it can&#x2019;t be accessed from outside of our library, making sure we&#x2019;re masters of the universe when it comes to one of our instance&#x2019;s state.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// always use closures, keep it to yourself</span>
(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(window, EventEmitter2)</span> </span>{
  <span class="md-code-comment">// see: http://stackoverflow.com/q/8651415/389745</span>
<span class="md-code-pi">  &apos;use strict&apos;</span>;

  <span class="md-code-comment">// we&apos;ll keep track of state in a private variable only we can access</span>
  <span class="md-code-keyword">var</span> state = {};

  <span class="md-code-comment">// we&apos;ll use this to give each instance a unique id</span>
  <span class="md-code-keyword">var</span> lastId = -<span class="md-code-number">1</span>;

  <span class="md-code-comment">// this is our Module&apos;s constructor</span>
  <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">Module</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-keyword">var</span> self = <span class="md-code-keyword">this</span>;

    <span class="md-code-comment">// assign a new unique id</span>
    self._id = ++lastId;
    
    <span class="md-code-comment">// initialize our state with some defaults</span>
    state[self._id] = {
      x: <span class="md-code-number">0</span>, y: <span class="md-code-number">0</span>
    };
    
    <span class="md-code-comment">// invoke the EventEmitter2 constructor</span>
    EventEmitter2.call(self, {
      wildcard: <span class="md-code-literal">true</span>
    });
  }

  <span class="md-code-comment">// inherit from EE2 prototype, &apos;subclassing&apos; it</span>
  Module.prototype = <span class="md-code-built_in">Object</span>.create(EventEmitter2.prototype);
  Module.prototype.constructor = Module;

  <span class="md-code-comment">// expose our Module as some Thing the global object has access to.</span>
  <span class="md-code-built_in">window</span>.Thing = Module;
})(<span class="md-code-built_in">window</span>, EventEmitter2);
</code></pre> <p>Then, applying this pattern is simply a matter of exposing commands implementors can interact with (our <em>public interface</em>), and emitting changes to our properties, avoiding to <em>expose these directly</em>. We will expose commands, such as <code class="md-code md-code-inline">move</code>, where users of our library can change the state of their components in a clearly defined way. They won&#x2019;t be able to access this state, however, except when we choose to expose it, so they need to be paying attention to our <em>emitted events</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// the move command</span>
Module.prototype.move = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(x, y)</span> </span>{
  <span class="md-code-comment">// get the old X and Y states, add some distance to them</span>
  set(<span class="md-code-keyword">this</span>, <span class="md-code-string">&apos;x&apos;</span>, get(<span class="md-code-keyword">this</span>, <span class="md-code-string">&apos;x&apos;</span>) + x);
  set(<span class="md-code-keyword">this</span>, <span class="md-code-string">&apos;y&apos;</span>, get(<span class="md-code-keyword">this</span>, <span class="md-code-string">&apos;y&apos;</span>) + y);
};

<span class="md-code-comment">// internal state setter</span>
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">set</span> <span class="md-code-params">(instance, key, value)</span> </span>{
  state[instance._id][key] = value;

  <span class="md-code-comment">// emit events when properties change</span>
  instance.emit([<span class="md-code-string">&apos;report&apos;</span>, key], value);  
}

<span class="md-code-comment">// internal state getter</span>
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">get</span> <span class="md-code-params">(instance, key)</span> </span>{
  <span class="md-code-keyword">return</span> state[instance._id][key];
}
</code></pre> <p>Say we were using <a href="https://github.com/madrobby/keymaster" target="_blank" aria-label="keymaster on GitHub">keymaster</a> to handle keyboard input, our module could then <em>consume input</em> like below. Note how we don&#x2019;t really even know what we&#x2019;re doing, we are just telling the component:</p> <blockquote> <p>Hey, <code class="md-code md-code-inline">Thing</code>!, Obey this command, <code class="md-code md-code-inline">.move</code>!</p> </blockquote> <p>The code would look something like this.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// get a Thing instance, obviously we&apos;ll always want the same one</span>
<span class="md-code-keyword">var</span> component = <span class="md-code-keyword">new</span> Thing();

<span class="md-code-keyword">var</span> map = {
  left: -<span class="md-code-number">1</span>,
  right: <span class="md-code-number">1</span>,
  up: -<span class="md-code-number">1</span>,
  down: <span class="md-code-number">1</span>
};

<span class="md-code-comment">// on left or right, move in the X axis</span>
key(<span class="md-code-string">&apos;left, right&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(e, h)</span> </span>{
  <span class="md-code-comment">// tell the component to move</span>
  component.move( map[h.shortcut], <span class="md-code-number">0</span> );
});

<span class="md-code-comment">// on up or down, move in the Y axis</span>
key(<span class="md-code-string">&apos;up, down&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(e, h)</span> </span>{
  <span class="md-code-comment">// tell the component to move</span>
  component.move( <span class="md-code-number">0</span>, map[h.shortcut] );
});
</code></pre> <p>Lastly, updating our UI would <em>also</em> be completely decloupled from the rest of the code. Whenever the state changes, we update the position of the UI representation for our <code class="md-code md-code-inline">Thing</code> instance. We don&#x2019;t care <em>how</em> it changed, we only care <em>that it did</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// get the element that represents our state in the UI</span>
<span class="md-code-keyword">var</span> element = <span class="md-code-built_in">document</span>.getElementById(<span class="md-code-string">&apos;square&apos;</span>);

<span class="md-code-comment">// whenever we get a report on X...</span>
component.on(<span class="md-code-string">&apos;report.x&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value)</span> </span>{
  <span class="md-code-comment">// update our left position</span>
  element.style.left = value + <span class="md-code-string">&apos;px&apos;</span>;
});

<span class="md-code-comment">// whenever we get a report on Y...</span>
component.on(<span class="md-code-string">&apos;report.y&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value)</span> </span>{
  <span class="md-code-comment">// update our top position</span>
  element.style.top = value + <span class="md-code-string">&apos;px&apos;</span>;
});

<span class="md-code-comment">// initialize at X=50, Y=50</span>
component.move(<span class="md-code-number">50</span>, <span class="md-code-number">50</span>);
</code></pre> <p>A <a href="http://cdpn.io/ejBvu" target="_blank" aria-label="View in CodePen">working example</a> can be found clicking on the image below.</p> <p><a href="http://cdpn.io/ejBvu" target="_blank" aria-label="View in CodePen"><img alt="thing.png" class="" src="https://i.imgur.com/1f66Pk6.png"></a></p> <p>This kind of pattern might be most useful when your components present <em>complex interactions between their different properties</em>. In those cases, it might be even better to use <strong>Angular.js</strong>. Although sometimes a <em>combination of both</em> might also prove to be useful. Of course, the hiding of information is entirely optional, and it might make sense not to hide anything but instead expose the properties in our components directly, however the event emitter pattern still makes sense to &#x201C;report&#x201D; <em>when</em> these events take place.</p></div>
