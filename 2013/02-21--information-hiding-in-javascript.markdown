<h1>Information Hiding in JavaScript</h1>

<blockquote><p>Even though it&#x2019;s tricky at first, if you are used to <em>classical</em> object-oriented languages, it&#x2019;s easy (and <em>highly encouraged</em>) to perform <strong>information hiding</strong> &#x2026;</p></blockquote>

<div><kbd>js</kbd> <kbd>js-native</kbd> <kbd>best-practices</kbd></div>

<div><p>Even though it&#x2019;s tricky at first, if you are used to <em>classical</em> object-oriented languages, it&#x2019;s easy (and <em>highly encouraged</em>) to perform <strong>information hiding</strong> in JavaScript. There are more than a few methods by which you can hide information within <em>privileged scopes</em>. <strong>Closures, properties (getters and setters), and factories</strong>, are all ways in which you can improve your designs by <strong>hiding away the complexity</strong> behind the <em>public interfaces</em> your code exposes.</p></div>

<div></div>

<div><p>Information hiding results in <em>cleaner, conciser code, that&#x2019;s easier to read</em>, and therefore, <strong>more maintainable</strong> (and easily <em>testable</em>).</p></div>

<div><h1 id="closures">Closures</h1> <p>JavaScript, <em>being the stupendous functional language that it is</em>, closures let us write clean code without cluttering up <a href="https://developer.mozilla.org/en-US/docs/DOM/window" target="_blank" aria-label="The Global Object in client-side JavaScript">the dreaded global object</a> with properties and functions that have no business being there. A better approach is to <em>self-contain</em> the knowledge required by a particular functionality. This can be accomplished by wrapping your code in a function:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
	<span class="md-code-comment">// your code goes here</span>
}
</code></pre> <p>That&#x2019;s a closure right there, but this closure is just a function, you want it to self-execute. You can accomplish this by using any of the following operators: <code class="md-code md-code-inline">-</code>, <code class="md-code md-code-inline">+</code>, <code class="md-code md-code-inline">!</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">!<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
	<span class="md-code-comment">// your code goes here</span>
}();
</code></pre> <p>I like prepending <code class="md-code md-code-inline">!</code> to my self-executing functions, some prefer <em>wrapping it in parenthesis</em> instead. It&#x2019;s just a matter of personal preference, but try and stick to a convention. The most common pattern, though, is wrapping the function in parenthesis. Like this:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
	<span class="md-code-comment">// your code goes here</span>
})();
</code></pre> <p>Note that none of this will be exposed to the global namespace <em>unless you explicitly do so</em>, the pattern for that is usually to add a <code class="md-code md-code-inline">window</code> argument to your closure (or an argument with the value of your library namespace), and expose your properties there.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">!<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(window)</span></span>{
	<span class="md-code-built_in">window</span>.myObject = {
		foo: <span class="md-code-string">&apos;bar&apos;</span>
	};
}(<span class="md-code-built_in">window</span>);
</code></pre> <p>Now that you have a closure, anything you define within your closure will be <strong>private</strong> to that closure&#x2019;s scope. Due to the cascading nature of closures, every closure has access to a <strong>superset</strong> of it&#x2019;s outer closure&#x2019;s scope, meaning it has access to the entire outer closure, and to whatever gets defined within itself.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">!<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(window)</span></span>{
	<span class="md-code-keyword">var</span> builder = <span class="md-code-string">&apos;Built with amazing closure awesomeness&apos;</span>,
		myCar;
			
	<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">decorate</span><span class="md-code-params">(car)</span></span>{
		car.builder = builder;
		<span class="md-code-keyword">return</span> car;
	}
	
	!<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
		<span class="md-code-comment">// well, not that much of a secret</span>
		<span class="md-code-keyword">var</span> secret = <span class="md-code-string">&apos;They were pioneers!&apos;</span>;
		
		myCar = decorate({
			make: <span class="md-code-string">&apos;Ford&apos;</span>,
			model: <span class="md-code-string">&apos;T&apos;</span>,
			getSecret: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
				<span class="md-code-keyword">return</span> secret;
			}
		});
	}();
	
	<span class="md-code-built_in">window</span>.car = myCar;
}(<span class="md-code-built_in">window</span>);
</code></pre> <p>Since the inner scope of <code class="md-code md-code-inline">getSecret</code> is a superset of the outer scope, it can access <code class="md-code md-code-inline">secret</code>.</p> <p>There are <strong>two exceptions</strong> to the cascading rule of scoping, and those are <code class="md-code md-code-inline">this</code> and <code class="md-code md-code-inline">arguments</code>. Those two variables are re-defined at every scope. A common work-around is to keep a reference around for future use:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">String</span>.prototype.isInArray = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(array)</span></span>{
	<span class="md-code-keyword">var</span> self = <span class="md-code-keyword">this</span>.toString();

	<span class="md-code-keyword">return</span> array.some(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(value)</span></span>{
		<span class="md-code-keyword">return</span> value === self;
	});
}

<span class="md-code-keyword">var</span> objects = [<span class="md-code-string">&apos;a&apos;</span>, <span class="md-code-string">&apos;b&apos;</span>, <span class="md-code-string">&apos;c&apos;</span>, <span class="md-code-string">&apos;d&apos;</span>, <span class="md-code-string">&apos;e&apos;</span>];
<span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;a&apos;</span>.isInArray(objects));
<span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;f&apos;</span>.isInArray(objects));
</code></pre> <h1 id="what-do-you-mean-properties">What do you mean, properties?</h1> <p>Bloggers seldom mention JavaScript properties <strong>(getters and setters)</strong>, to the point where they&#x2019;ve barely seen the light of day, and have not even remotely been adopted in widespread use.</p> <blockquote> <p>Properties are subtle yet powerful tools, in particular when trying to do <a href="http://en.wikipedia.org/wiki/Aspect-oriented_programming" target="_blank" aria-label="Aspect Oriented Programming">AOP</a>. Sometimes, it makes more sense to use a property rather than a field, or a function. The beauty of properties is that you can have behavior in your fields. Additionally, some libraries <em>require</em> you to use fields instead of functions, this helps you circumvent that kind of requirements.</p> </blockquote> <p>Allow me to demonstrate with a mini-object:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">!<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(window)</span></span>{
	<span class="md-code-keyword">var</span> workers = [],
		factor = <span class="md-code-number">1</span>;
	
	<span class="md-code-built_in">window</span>.timecard = {
		checkin: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(username)</span></span>{
			<span class="md-code-keyword">if</span> (workers.indexOf(username) === -<span class="md-code-number">1</span>){
				<span class="md-code-built_in">console</span>.log(username + <span class="md-code-string">&apos; checked in!&apos;</span>);
				workers.push(username);
			}
		},
		checkout: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(username)</span></span>{
			<span class="md-code-keyword">var</span> i = workers.indexOf(username);
			<span class="md-code-keyword">if</span> (i !== -<span class="md-code-number">1</span>){
				<span class="md-code-built_in">console</span>.log(username + <span class="md-code-string">&apos; checked out!&apos;</span>);
				workers.splice(i, <span class="md-code-number">1</span>);
			}
		},
		get workers() {
			<span class="md-code-keyword">return</span> workers.join(<span class="md-code-string">&apos; and &apos;</span>);
		},
		get productivity() {
			<span class="md-code-keyword">return</span> workers.length * factor;
		},
		get productivityFactor(){
			<span class="md-code-keyword">return</span> factor;
		},
		set productivityFactor(value){
			<span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;Productivity &apos;</span> + (value &gt; factor ? <span class="md-code-string">&apos;increased!&apos;</span> : <span class="md-code-string">&apos;decreased.&apos;</span>));
			factor = value;
		}
	};
}(<span class="md-code-built_in">window</span>);

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">p</span><span class="md-code-params">(test)</span></span>{ <span class="md-code-comment">// print</span>
	<span class="md-code-built_in">console</span>.log(test);
}

timecard.checkin(<span class="md-code-string">&apos;Joe&apos;</span>);
timecard.checkin(<span class="md-code-string">&apos;Steven&apos;</span>);
p(timecard.workers === <span class="md-code-string">&apos;Joe and Steven&apos;</span>); <span class="md-code-comment">// true</span>
p(timecard.productivity); <span class="md-code-comment">// 2</span>
|timecard.productivity = <span class="md-code-number">3</span>; <span class="md-code-comment">// productivity is read-only</span>
p(timecard.productivity === <span class="md-code-number">2</span>); <span class="md-code-comment">// thus, setting it is a no-op</span>
timecard.productivityFactor = <span class="md-code-number">4</span>;
timecard.checkout(<span class="md-code-string">&apos;Joe&apos;</span>);
p(timecard.productivity); <span class="md-code-comment">// 4. The getter function is evaluated every time.</span>
</code></pre> <p>Note that IE versions up to <strong>IE 8</strong> <em>don&#x2019;t support <code class="md-code md-code-inline">get</code> and <code class="md-code md-code-inline">set</code> operators</em>. To work around that, you&#x2019;ll have to use the <code class="md-code md-code-inline">__defineGetter__</code> and <code class="md-code md-code-inline">__defineSetter__</code> methods that can be found in <code class="md-code md-code-inline">Object.prototype</code>. These are a bit more flexible because it&#x2019;s impossible to add a getter or setter to an existing object without using one of them.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> o = {
	name: <span class="md-code-string">&apos;Steven&apos;</span>,
	surname: <span class="md-code-string">&apos;Sundae&apos;</span>
};

o.__defineGetter__(<span class="md-code-string">&apos;displayName&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
	<span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>.name + <span class="md-code-string">&apos; &apos;</span> + <span class="md-code-keyword">this</span>.surname;
});
</code></pre> <h1 id="functional-factories">Functional Factories</h1> <p>Closely related to <a href="http://en.wikipedia.org/wiki/Currying" target="_blank" aria-label="Currying">currying</a>, are the <em>functional factories</em>. These factories are very powerful to the JavaScript hacker. They will allow you to reuse very similar pieces of functionality in cases where it would be otherwise very hard to do so.</p> <p>Suppose you are writing your <a href="http://nodejs.org/" target="_blank" aria-label="Node.JS">Node.JS</a> application, and you have a series of <em>router methods</em>, all of which take <code class="md-code md-code-inline">req, res</code> as parameters. All that changes in each of those, is the function you call in your application logic, but there&#x2019;s a few more tasks you have to do, such as <em>sending a response</em>, that are common to every request. Lets assume your module exports an object hash keyed with <em>routes</em>, whose values are the <em>middleware</em> for each of those routes.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> list = [<span class="md-code-number">1</span>,<span class="md-code-number">2</span>,<span class="md-code-number">3</span>,<span class="md-code-number">4</span>];

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">respond</span><span class="md-code-params">(res,value)</span></span>{
	<span class="md-code-keyword">var</span> json = <span class="md-code-built_in">JSON</span>.stringify(value);
	
	res.writeHead(<span class="md-code-number">200</span>, { <span class="md-code-string">&apos;Content-Type&apos;</span>: <span class="md-code-string">&apos;application/json&apos;</span> });
	res.end(json);
}

<span class="md-code-built_in">module</span>.exports = {
	list: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(req,res)</span></span>{
		respond(res,list);
	},
	add: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(req,res)</span></span>{
		list.push(req.body.item);
		respond(res,list);
	},
	remove: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(req,res)</span></span>{
		list.splice(req.body.index, <span class="md-code-number">1</span>);
		respond(res,list);
	}
};
</code></pre> <p>Generally speaking, when you see something along the lines of:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(arg1,arg2,arg3,arg4)</span></span>{
	<span class="md-code-keyword">do</span>(arg1,arg2,arg3,arg4);
}
</code></pre> <p>That code can be refactored to be a bit more <a href="http://en.wikipedia.org/wiki/Don%27t_repeat_yourself" target="_blank" aria-label="DRY principle">DRY</a>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">handle</span><span class="md-code-params">(before)</span></span>{
	<span class="md-code-keyword">var</span> noop = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{};
	
	<span class="md-code-keyword">return</span> <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(req, res)</span></span>{
		(before || noop)(req, res);
		respond(res, list);
	};
}

<span class="md-code-built_in">module</span>.exports = {
	list: handle(),
	add: handle(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(req)</span></span>{
		list.push(req.body.item);
	}),
	remove: handle(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(req)</span></span>{
		list.splice(req.body.index, <span class="md-code-number">1</span>);
	})
};
</code></pre> <p>Granted, this style is a tad more <em>verbose</em>, especially if <strong>unwarranted</strong>, but if your <code class="md-code md-code-inline">handle</code> function did <em>more stuff</em>, such as <em>request validation</em>, <em>logging</em>, or trigger web socket events, this would be a nice way to abstract those away from the specifics of each of those methods, that little have to do with validation itself.</p> <p>Using such factories also allows to abstract away complexity, which is always a very valuable thing to do when designing applications.</p></div>
