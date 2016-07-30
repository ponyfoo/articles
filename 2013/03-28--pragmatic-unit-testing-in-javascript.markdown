<div><blockquote>
  <h1>Pragmatic Unit Testing in JavaScript</h1>
  <div><p>More often than not, companies completely (and <em>irresponsibly</em>) disregard JavaScript as code that <em>should be unit tested</em>. They might test their back-end code, it may be in &#x2026;</p></div>
</blockquote></div>

<div><p>More often than not, companies completely (and <em>irresponsibly</em>) disregard JavaScript as code that <em>should be unit tested</em>. They might test their back-end code, it may be in C#, Ruby, Java, or even PHP, or <em>just about any other language</em>. But there&#x2019;s a good chance that the front-end code is <strong>thoroughly untested</strong>.</p></div>

<div></div>

<div><p><em>Integration level testing</em> with tools such as <a href="http://docs.seleniumhq.org/" target="_blank">Selenium</a> is nice in theory, but way too impractical (you have to set up a server), and particularly slow (loading browsers and computing the recorded actions takes its toll). As such it&#x2019;s rarely part of build processes, and it&#x2019;s run manually (with a single command, but manually nonetheless).</p> <p>So why is that JavaScript gets <em>treated so differently</em> from the rest of languages?</p></div>

<div><p><img alt="js-discrimination.jpg" title="JavaScript statements are people too!" class="" src="https://i.imgur.com/WUHDGPw.jpg"></p> <h1 id="arguments-against-testing-javascript-in-the-client-side">Arguments against testing JavaScript in the client-side</h1> <ul> <li>JavaScript, and <em>the web</em> in general are <a href="http://www.codinghorror.com/blog/2007/04/javascript-and-html-forgiveness-by-default.html" target="_blank" aria-label="JavaScript and HTML: Forgiveness by Default"><em>tremendously</em> fault tolerant</a></li> <li>Errors in front-end code are <em>not perceived to be as <strong>impactful</strong> as back-end errors</em>. Since this is client-side code, no sensitive data will get lost or <em>accidentally deleted</em>. As long as the back-end is safe, data is safe</li> <li>JavaScript is <em>challenging to test</em></li> </ul> <p>While it&#x2019;s true that errors in the front-end are not <em>as likely</em> as permeate a robust back-end layer and cause trouble, there are real threats out there, such as <a href="http://en.wikipedia.org/wiki/Cross-site_scripting" target="_blank" aria-label="Cross-site scripting">XSS attacks</a>, which are enabled by front and back-end alike.</p> <h2 id="why-is-testing-javascript-hard">Why is testing JavaScript <em>hard</em>?</h2> <p>I could give a list of reasons why testing JavaScript is hard, but <em>it all boils down to it being a dynamic language</em>. There&#x2019;s no compiler. Sometimes that it great, we&#x2019;ve come to <em>love</em> the language for its dynamic nature. However, it makes testing <em>harder</em>.</p> <p>As such, our first line of defense should be <a href="https://ponyfoo.com/2013/03/22/managing-code-quality-in-nodejs" aria-label="Managing Code Quality in NodeJS">linting</a>. This is the closest we have to a compiler, in terms of assurance that <em>our code won&#x2019;t break</em>.</p> <p>But that obviously isn&#x2019;t <em>enough</em>. Linting is <em>just the first step</em> in the right direction. Our code should be tested.</p> <h2 id="back-to-the-dynamic-nature-of-javascript">Back to the dynamic nature of JavaScript</h2> <p>I think another important factor in testing is <em>visbility</em>. In statically typed languages such as C#, variables can be <code class="md-code md-code-inline">private</code>, <code class="md-code md-code-inline">public</code>, <code class="md-code md-code-inline">protected</code>, <code class="md-code md-code-inline">internal</code>, or some combination of those. In JS it&#x2019;s either <code class="md-code md-code-inline">private</code> or <code class="md-code md-code-inline">public</code>.</p> <p>There are no <em>statically defined interfaces</em>. You might be used to interfaces such as:</p> <pre class="md-code-block"><code class="md-code md-lang-cs"><span class="md-code-keyword">public</span> <span class="md-code-keyword">interface</span> <span class="md-code-title">ITrackable</span>
{
    <span class="md-code-keyword">int</span> TrackingNumber { <span class="md-code-keyword">get</span>; }
    <span class="md-code-function"><span class="md-code-keyword">void</span> <span class="md-code-title">Track</span><span class="md-code-params">()</span></span>;
    <span class="md-code-function"><span class="md-code-keyword">bool</span> <span class="md-code-title">Untrack</span><span class="md-code-params">()</span></span>;
    <span class="md-code-keyword">bool</span> IsBeingTracked { <span class="md-code-keyword">get</span>; }
}
</code></pre> <p>Bear with me for this small example of a testable class, written in C#:</p> <pre class="md-code-block"><code class="md-code md-lang-cs"><span class="md-code-keyword">public</span> <span class="md-code-keyword">class</span> <span class="md-code-title">Testable</span>
{
    <span class="md-code-keyword">private</span> <span class="md-code-keyword">readonly</span> ITrackable _trackable;
    
    <span class="md-code-function"><span class="md-code-keyword">public</span> <span class="md-code-title">Testable</span><span class="md-code-params">(ITrackable trackable)</span>
    </span>{
        _trackable = trackable;
    }
    
    <span class="md-code-function"><span class="md-code-keyword">public</span> <span class="md-code-keyword">bool</span> <span class="md-code-title">CallMeTracy</span><span class="md-code-params">()</span>
    </span>{
        _trackable.Track();
        <span class="md-code-keyword">return</span> <span class="md-code-keyword">true</span>;
    }
}
</code></pre> <p>The code doesn&#x2019;t make any sense. I know. The case in point is that, using <a href="http://www.amazon.com/dp/1935182501" target="_blank" aria-label="Dependency Injection in .NET">Dependency Injection</a>, <code class="md-code md-code-inline">Testable</code> becomes very easily testable. Here&#x2019;s a sample test:</p> <pre class="md-code-block"><code class="md-code md-lang-cs">[TestCase]
<span class="md-code-keyword">public</span> <span class="md-code-keyword">class</span> <span class="md-code-title">TestableTests</span>
{
    <span class="md-code-keyword">private</span> Testable testable;
    
    [SetUp]
    <span class="md-code-function"><span class="md-code-keyword">public</span> <span class="md-code-keyword">void</span> <span class="md-code-title">Setup</span><span class="md-code-params">()</span>
    </span>{
        <span class="md-code-keyword">var</span> mock = <span class="md-code-keyword">new</span> FakeTrackable();
        testable = <span class="md-code-keyword">new</span> Testable(mock);
    }
    
    [Test]
    <span class="md-code-function"><span class="md-code-keyword">public</span> <span class="md-code-keyword">void</span> <span class="md-code-title">should_call_me_tracy_and_return_true</span><span class="md-code-params">()</span>
    </span>{
        <span class="md-code-keyword">bool</span> result = testable.CallMeTracy();
        
        Assert.IsTrue(result, <span class="md-code-string">&quot;Expected CallMeTracy to return true.&quot;</span>);
    }
}

<span class="md-code-keyword">public</span> <span class="md-code-keyword">class</span> <span class="md-code-title">FakeTrackable</span> : <span class="md-code-title">ITrackable</span>
{
    <span class="md-code-function"><span class="md-code-keyword">public</span> <span class="md-code-keyword">void</span> <span class="md-code-title">Track</span><span class="md-code-params">()</span>
    </span>{
    }

    <span class="md-code-comment">// ... other implementation stubs ...</span>
}
</code></pre> <p>How do we reach a similar state of affairs in JavaScript? We simply can&#x2019;t. We must <em>adapt to the dynamism</em>, embrace it.</p> <h1 id="how-to-test-then">How to test, then?</h1> <p>It&#x2019;s not as bad as you might be thinking right now. It&#x2019;s just a matter of changing the way you think about testing.</p> <p>JavaScript testing needs to be <em>even more thorough</em>. Your code can&#x2019;t statically provide the interface you desire? Then <em>write tests</em> to ensure it exposes that interface. Welcome to <strong>TDD</strong>!</p> <p>Lets refer to a similar example in JavaScript</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">Testable</span><span class="md-code-params">(trackable)</span></span>{
    <span class="md-code-keyword">this</span>.callMeTracy = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
        trackable.track();
        <span class="md-code-keyword">return</span> <span class="md-code-literal">true</span>;
    };
}
</code></pre> <p>And the test, which might be something like this</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">describe(<span class="md-code-string">&apos;Testable&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
    it(<span class="md-code-string">&apos;should return true when calling him Tracy&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
        <span class="md-code-keyword">var</span> him = <span class="md-code-keyword">new</span> Testable({
            track: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
                <span class="md-code-comment">// this is just a mock</span>
            }
        });
        
        expect(him.callMeTracy).toBeDefined();
        expect(him.callMeTracy()).toBeTruthy();
    });
});
</code></pre> <p>Obviously, a clear difference here is that <code class="md-code md-code-inline">trackable</code> can be <em>anything</em>, unless it&#x2019;s constrained by <em>guard clauses</em>. But, that&#x2019;s generally something that JavaScript developers shy away from, given the <em>sheer power</em> it provides.</p> <h2 id="dependency-injection-in-javascript">Dependency Injection in JavaScript</h2> <p>There&#x2019;s a catch though, injecting dependencies in JavaScript is <em>kind of a mess</em>.</p> <p>If you are writing tests for <strong>Node.JS</strong> code, you might be in luck. I have been using <a href="https://github.com/thlorenz/proxyquire" target="_blank" aria-label="thlorenz/proxyquire on GitHub">proxyquire</a>, which basically allows you to, <em>without modifying a single line of source code</em>, test modules and mock dependencies on other modules loaded with <code class="md-code md-code-inline">require</code>. It requires some setting up, but it made me pretty happy thus far.</p> <p><strong>Browser code</strong> is <em>a different story</em>, it often follows patterns similar to this:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">!<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(window, $)</span></span>{
    <span class="md-code-built_in">window</span>.myThing = {
        annoy: <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
            alert(<span class="md-code-string">&apos;about to become very annoying!&apos;</span>);
            $(<span class="md-code-string">&apos;a, span, b, em&apos;</span>).wrap(<span class="md-code-string">&apos;&lt;marquee/&gt;&apos;</span>);
        }
    };
}(<span class="md-code-built_in">window</span>, jQuery);
</code></pre> <p>This kind of code is indeed hard to test, but you could always just load the affected JS file in isolation, after creating stubs for the global objects you need. An example would be:</p> <pre class="md-code-block"><code class="md-code">var window = {},
    jQuery = function(){
        return {
            wrap: function(){
            }
        };
    };
</code></pre> <p>Once you get tired of helplessly mocking your way out of trouble, you should use a real stubbing and mocking framework, such as <a href="http://sinonjs.org/" target="_blank" aria-label="Standalone test spies, stubs and mocks">Sinon.JS</a>. Also, remember that using spies to verify that callbacks (such as <code class="md-code md-code-inline">callMeTracy</code>, and <code class="md-code md-code-inline">wrap</code>) are invoked, and passed <em>the correct parameters</em>!</p> <h2 id="unit-testing-frameworks">Unit Testing Frameworks</h2> <p>I like <a href="http://pivotal.github.com/jasmine" target="_blank" aria-label="Jasmine BDD framework for testing">Jasmine</a> for my unit testing, but there are plenty of frameworks to choose from. <a href="http://visionmedia.github.com/mocha" target="_blank" aria-label="Mocha test framework">Mocha</a> is another popular one.</p> <p>Once you realize that testing in JavaScript is <em>not that bad</em>, and look at it from <em>another perspective</em>, you can start exploiting the very dynamic nature you feared to help you build <em>even better tests</em>!</p> <p>A pattern I commonly use when defining unit tests in Jasmine, is to prepare a list of test cases (expected input and output), and then run them all at once. Here&#x2019;s an example taken directly from <a href="https://github.com/bevacqua/jsn/blob/29384246d28d688475669375423568c54439feed/test/spec/text.js" target="_blank" aria-label="jsn GitHub repository">one of my GitHub repositories</a>:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">describe(<span class="md-code-string">&apos;test cases&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
    <span class="md-code-keyword">var</span> cases = [],
        context = {
            plain: <span class="md-code-string">&apos;plain&apos;</span>,
            foo: {
                bar: <span class="md-code-string">&apos;baz&apos;</span>,
                undef: <span class="md-code-literal">undefined</span>,
                nil: <span class="md-code-literal">null</span>,
                num: <span class="md-code-number">12</span>
            },
            color: <span class="md-code-string">&apos;red&apos;</span>,
            how: { awesome: <span class="md-code-string">&apos;very&apos;</span> }
        };

    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">include</span><span class="md-code-params">(input,output)</span></span>{ cases.push({ input: input, output: output }); }

    include(<span class="md-code-string">&apos;@plain&apos;</span>,<span class="md-code-string">&apos;plain&apos;</span>);
    include(<span class="md-code-string">&apos;@foo.bar&apos;</span>,<span class="md-code-string">&apos;baz&apos;</span>);
    include(<span class="md-code-string">&apos;@foo.undef&apos;</span>,<span class="md-code-literal">undefined</span>);
    include(<span class="md-code-string">&apos;@foo.nil&apos;</span>,<span class="md-code-literal">null</span>);
    include(<span class="md-code-string">&apos;@foo.num&apos;</span>,<span class="md-code-number">12</span>);
    include(<span class="md-code-string">&apos;@@foo.bar&apos;</span>,<span class="md-code-string">&apos;@foo.bar&apos;</span>);

    cases.forEach(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(testCase,i)</span></span>{
        <span class="md-code-keyword">var</span> replace = text.replace(<span class="md-code-built_in">Object</span>.create(context));

        it(<span class="md-code-string">&apos;should return expected output for case #&apos;</span> + (i+<span class="md-code-number">1</span>), <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
            expect(replace(testCase.input)).toEqual(testCase.output);
        });
    });
});
</code></pre> <p>This is something that <em>simply cannot be accomplished statically</em>. You could get here with reflection, but it&#x2019;s just unnatural to static languages.</p></div>
