<h1>Architecture of ECMAScript 6 Modules</h1>

<blockquote><p>This blog post contains useful information if you&#x2019;re interested in the latest developments on ECMAScript 6 Harmony modules and they current state of their &#x2026;</p></blockquote>

<div><kbd>es6</kbd> <kbd>modules</kbd> <kbd>harmony</kbd></div>

<div><p>This blog post contains useful information if you&#x2019;re interested in the latest developments on ECMAScript 6 Harmony modules and they current state of their implementation. The original post <a href="https://gist.github.com/wycats/51c96e3adcdb3a68cbc3" target="_blank">can be found here</a>.</p></div>

<div></div>

<div><p><img src="https://i.imgur.com/bky9ghC.png" alt="es6.png"></p> <p>At the bottom you&#x2019;ll find a list of useful <em>related links</em>.</p></div>

<div><p>This document covers a number of use-cases covered by existing module implementations in JavaScript, and how those use-cases will be handled by ES6 modules.</p> <p>It will also cover some additional use-cases unique to the ES6 module system.</p> <h2 id="terminology">Terminology</h2> <p>For those unfamiliar with the current ES6 module proposal, here is some terminology you should understand:</p> <ol> <li><em>module</em>: a unit of source code with optional <em>imports</em> and <em>exports</em>.</li> <li><em>export</em>: a module can <code class="md-code md-code-inline">export</code> a value with a name.</li> <li><em>imports</em>: a module can <code class="md-code md-code-inline">import</code> a value exported by another module by its name.</li> <li><em>module instance object</em>: an instance of the <code class="md-code md-code-inline">Module</code> constructor that represents a <em>module</em>. Its property names and values come from the <em>module</em>&#x2019;s exports.</li> <li><em>Loader</em>: an object that defines how modules are fetched, translated, and compiled into a <em>module instance object</em>. Each JavaScript environment (the browser, node.js) defines a default Loader that defines the semantics for that environment.</li> </ol> <h2 id="imports-and-exports">Imports and Exports</h2> <p>Let&#x2019;s start with the basic API of ES6 modules:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// libs/string.js</span>

<span class="md-code-keyword">var</span> underscoreRegex1 = <span class="md-code-regexp">/([a-z\d])([A-Z]+)/g</span>,
    underscoreRegex2 = <span class="md-code-regexp">/\-|\s+/g</span>;

export <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">underscore</span><span class="md-code-params">(string)</span> </span>{
  <span class="md-code-keyword">return</span> string.replace(underscoreRegex1, <span class="md-code-string">&apos;$1_$2&apos;</span>)
               .replace(underscoreRegex2, <span class="md-code-string">&apos;_&apos;</span>)
               .toLowerCase();
}

export <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">capitalize</span><span class="md-code-params">(string)</span> </span>{
  <span class="md-code-keyword">return</span> string.charAt(<span class="md-code-number">0</span>).toUpperCase() + string.substr(<span class="md-code-number">1</span>);
}
</code></pre> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// app.js</span>

import { capitalize } from <span class="md-code-string">&quot;libs/string&quot;</span>;

<span class="md-code-keyword">var</span> app = {
  name: capitalize(<span class="md-code-built_in">document</span>.title)
};

export app;
</code></pre> <p>This illustrates the basic syntax of ES6 modules. A module can export named values, and other modules can import those values.</p> <h2 id="avoiding-scope-pollution">Avoiding Scope Pollution</h2> <p>When working with a module with a large number of exports, you may want to avoid adding each of them as top-level names of another module that wants to import it.</p> <p>For example, consider an API like <a href="http://nodejs.org/api/fs.html" target="_blank">Node.js fs module</a>. This module has a large number of exports, like <code class="md-code md-code-inline">rename</code>, <code class="md-code md-code-inline">chown</code>, <code class="md-code md-code-inline">chmod</code>, <code class="md-code md-code-inline">stat</code> and others. With the ES6 module API, it is possible to bring in the module as a single top-level name that contains all of the module&#x2019;s exports.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">import <span class="md-code-string">&quot;fs&quot;</span> as fs;

fs.rename(oldPath, newPath, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(err)</span> </span>{
  <span class="md-code-comment">// continue</span>
});
</code></pre> <h2 id="concatenation">Concatenation</h2> <p>In the example above, the modules were loaded based on their location on the file system. This is how the default Loader for the browser will work.</p> <p>For production applications, you will want to concatenate the files on the file system into a single file. ES6 modules handle this case by providing a literal way to define a module:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">module</span> <span class="md-code-string">&quot;libs/string&quot;</span> {
  <span class="md-code-keyword">var</span> underscoreRegex1 = <span class="md-code-regexp">/([a-z\d])([A-Z]+)/g</span>,
      underscoreRegex2 = <span class="md-code-regexp">/\-|\s+/g</span>;

  export <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">underscore</span><span class="md-code-params">(string)</span> </span>{
    <span class="md-code-keyword">return</span> string.replace(underscoreRegex1, <span class="md-code-string">&apos;$1_$2&apos;</span>)
                 .replace(underscoreRegex2, <span class="md-code-string">&apos;_&apos;</span>)
                 .toLowerCase();
  }

  export <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">capitalize</span><span class="md-code-params">(string)</span> </span>{
    <span class="md-code-keyword">return</span> string.charAt(<span class="md-code-number">0</span>).toUpperCase() + string.substr(<span class="md-code-number">1</span>);
  }
}

<span class="md-code-built_in">module</span> <span class="md-code-string">&quot;app&quot;</span> {
  import { capitalize } from <span class="md-code-string">&quot;libs/string&quot;</span>;

  <span class="md-code-keyword">var</span> app = {
    name: capitalize(<span class="md-code-built_in">document</span>.title)
  };

  export app;
}
</code></pre> <p>Modules defined using this syntax will be available to other modules, and will not needed to be fetched through the Loader.</p> <h2 id="modules-in-non-default-locations">Modules in Non-Default Locations</h2> <p>In web applications, while many modules may be concatenated into a single file for production use, some modules, like jQuery, may be loaded off of a CDN.</p> <p>It is possible to override the default Loader hooks to specify where to load a module from, but ES6 modules provide a simple API for mapping modules to their physical location.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">System.ondemand({
  <span class="md-code-string">&quot;https://ajax.googleapis.com/jquery/2.4/jquery.module.js&quot;</span>: <span class="md-code-string">&quot;jquery&quot;</span>,
  <span class="md-code-string">&quot;backbone.js&quot;</span>: [<span class="md-code-string">&quot;backbone/events&quot;</span>, <span class="md-code-string">&quot;backbone/model&quot;</span>]
});
</code></pre> <p>The first line in the example specifies that the <code class="md-code md-code-inline">jquery</code> module can be found at <code class="md-code md-code-inline">https://ajax.googleapis.com/jquery/2.4/jquery.module.js</code>.</p> <p>The second line specifies that <code class="md-code md-code-inline">backbone/events</code> and <code class="md-code md-code-inline">backbone/model</code> can both be found at <code class="md-code md-code-inline">backbone.js</code>.</p> <p>You can call <code class="md-code md-code-inline">System.ondemand</code> as many times as you want, so libraries can provide a snippet of code for people to use in order to <em>import</em> their libraries.</p> <h2 id="the-compilation-pipeline">The Compilation Pipeline</h2> <p>The next several sections deal with various use-cases involving the compilation pipeline.</p> <p>Here is a high-level overview of the process.</p> <img class="" src="https://i.imgur.com/6yAVeq2.png"> <p>The dotted line between <strong>fetch</strong> and <strong>translate</strong> reflects the fact that process of retrieving the source is asynchronous.</p> <h3 id="stricter-mode-linting">Stricter Mode (Linting)</h3> <p>Linting tools are a crucial part of a JavaScript developer&#x2019;s workflow, but they are currently used primarily via a compilation toolchain that presents errors in the terminal.</p> <p>Using the Module Loader&#x2019;s translate hook, it is possible to add additional static checks that are presented to the user as <code class="md-code md-code-inline">SyntaxError</code>s.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">import { JSHINT } from <span class="md-code-string">&quot;jshint&quot;</span>;
import { options } from <span class="md-code-string">&quot;app/jshintrc&quot;</span>

System.translate = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">var</span> errors = JSHINT(source, options), messages = [options.actualAddress];

  <span class="md-code-keyword">if</span> (errors) {
    errors.forEach(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(error)</span> </span>{
      <span class="md-code-keyword">var</span> message = <span class="md-code-string">&apos;&apos;</span>;
      message += error.line + <span class="md-code-string">&apos;:&apos;</span> + error.character + <span class="md-code-string">&apos;, &apos;</span>;
      message += error.reason;
      messages.push(message);
    });

    <span class="md-code-keyword">throw</span> <span class="md-code-keyword">new</span> <span class="md-code-built_in">SyntaxError</span>(messages.join(<span class="md-code-string">&quot;\n&quot;</span>));
  }

  <span class="md-code-keyword">return</span> source;
};
</code></pre> <p>If the linter returns errors, the translate hook raises a <code class="md-code md-code-inline">SyntaxError</code> and the Loader pipeline will stop, throwing the exception as if it was a true <code class="md-code md-code-inline">SyntaxError</code>.</p> <img class="" src="https://i.imgur.com/tbeB9TJ.png"> <h3 id="importing-compile-to-javascript-modules-coffeescript">Importing Compile-to-JavaScript Modules (CoffeeScript)</h3> <p>Increasingly, modules are written using languages that compile to JavaScript.</p> <p>The <code class="md-code md-code-inline">translate</code> hook provides a way to translate source code to JavaScript before it is loaded as a module.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">System.translate = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (!options.path.match(<span class="md-code-regexp">/\.coffee$/</span>)) { <span class="md-code-keyword">return</span>; }

  <span class="md-code-keyword">return</span> CoffeeScript.translate(source);
};
</code></pre> <p>In this example, any modules ending in <code class="md-code md-code-inline">.coffee</code> will be translated from CoffeeScript to JavaScript, and the rest of the pipeline will just see the compiled JavaScript.</p> <img class="" src="https://i.imgur.com/GIAQldl.png"> <h3 id="verification-and-translation">Verification and Translation</h3> <p>Some other compilers, like <a href="http://typescript.codeplex.com/" target="_blank">TypeScript</a> and <a href="http://restrictmode.org/" target="_blank">restrict mode</a> perform both compile-time verification and source translation.</p> <p>The above techniques could be combined to produce seamless in-browser support for such libraries.</p> <h3 id="using-existing-libraries-as-modules">Using Existing Libraries as Modules</h3> <p>The existing jQuery library is distributed as a library that &#x201C;exports&#x201D; the <code class="md-code md-code-inline">jQuery</code> name onto the global object.</p> <p>It should be possible to import existing libraries without having to modify the original source, like this:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">import { jQuery } from <span class="md-code-string">&quot;jquery&quot;</span>;

jQuery(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">($)</span> </span>{
  $(<span class="md-code-string">&quot;.ui-button&quot;</span>).button();
});
</code></pre> <p>The final hook in the process, <strong>link</strong> can be used to manually process a source file into a <em>Module</em> object.</p> <p>In this case, we could configure the Loader to extract all properties written to <code class="md-code md-code-inline">window</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">extractExports</span><span class="md-code-params">(loader, original)</span> </span>{
  source =
    `<span class="md-code-keyword">var</span> exports = {};
    (<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(window)</span> </span>{
      ${source};
    })(exports);
    exports;`

  <span class="md-code-keyword">return</span> loader.eval(source);
}

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (options.metadata.type === <span class="md-code-string">&apos;legacy&apos;</span>) {
    <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(extractExports(<span class="md-code-keyword">this</span>, source));
  }

  <span class="md-code-comment">// returning undefined will result in the normal</span>
  <span class="md-code-comment">// parsing and registration behavior</span>
}
</code></pre> <p>In order to make it easy for the <strong>link</strong> hook to decide whether it should use custom linking logic, the <code class="md-code md-code-inline">resolve</code> hook can provide metadata for the module that will be passed to the following hooks.</p> <p>In this case, you can keep a list of which modules are &#x201C;legacy&#x201D; and populate the metadata with that information in <code class="md-code md-code-inline">resolve</code>:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> legacy = [<span class="md-code-string">&quot;jquery&quot;</span>, <span class="md-code-string">&quot;backbone&quot;</span>, <span class="md-code-string">&quot;underscore&quot;</span>];

System.resolve = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(path, options)</span> </span>{
  <span class="md-code-keyword">if</span> (legacy.indexOf(path) &gt; -<span class="md-code-number">1</span>) {
    <span class="md-code-keyword">return</span> { name: path, metadata: { type: <span class="md-code-string">&apos;legacy&apos;</span> } };
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-keyword">return</span> { name: path, metadata: { type: <span class="md-code-string">&apos;es6&apos;</span> } };
  }
}
</code></pre> <img class="" src="https://i.imgur.com/y6Jjk0o.png"> <h3 id="importing-amd-modules-from-es6-modules">Importing AMD Modules from ES6 Modules</h3> <p>Similarly, you may want to import an AMD module&#x2019;s exports in an ES6 module.</p> <p>Consider a simple AMD module for the string formatting example above:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// libs/string.js</span>

define([<span class="md-code-string">&apos;exports&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(exports)</span> </span>{
  <span class="md-code-keyword">var</span> underscoreRegex1 = <span class="md-code-regexp">/([a-z\d])([A-Z]+)/g</span>,
      underscoreRegex2 = <span class="md-code-regexp">/\-|\s+/g</span>;

  exports.underscore = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(string)</span> </span>{
    <span class="md-code-keyword">return</span> string.replace(underscoreRegex1, <span class="md-code-string">&apos;$1_$2&apos;</span>)
                 .replace(underscoreRegex2, <span class="md-code-string">&apos;_&apos;</span>)
                 .toLowerCase();
  }

  exports.capitalize = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(string)</span> </span>{
    <span class="md-code-keyword">return</span> string.charAt(<span class="md-code-number">0</span>).toUpperCase() + string.substr(<span class="md-code-number">1</span>);
  }
});
</code></pre> <p>To assimilate this module, you could use a similar technique to the one we used above for jQuery:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> amd = [<span class="md-code-string">&quot;string-utils&quot;</span>];

<span class="md-code-comment">// Resolve </span>
System.resolve = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(path, options)</span> </span>{
  <span class="md-code-keyword">if</span> (amd.indexOf(path) &gt; -<span class="md-code-number">1</span>) {
    <span class="md-code-keyword">return</span> { name: path, metadata: { type: <span class="md-code-string">&apos;amd&apos;</span> } };
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-keyword">return</span> { name: path, metadata: { type: <span class="md-code-string">&apos;es6&apos;</span> } };
  }
};

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">extractAMDExports</span><span class="md-code-params">(loader, source)</span> </span>{
  <span class="md-code-keyword">var</span> loader = <span class="md-code-keyword">new</span> Loader();
  loader.eval(`
    <span class="md-code-keyword">var</span> <span class="md-code-built_in">module</span>;
    <span class="md-code-keyword">var</span> define = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(deps, callback)</span> </span>{
      <span class="md-code-built_in">module</span> = { deps: deps, callback: callback };
    };
    ${source};
    <span class="md-code-built_in">module</span>;
  `);

  <span class="md-code-comment">// Assume synchronously available dependencies. See below</span>
  <span class="md-code-comment">// for a discussion of async dependencies.</span>
  <span class="md-code-keyword">var</span> exports = {};
  <span class="md-code-keyword">var</span> deps = <span class="md-code-built_in">module</span>.deps.map(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(name)</span> </span>{
    <span class="md-code-comment">// AMD uses a special dependency named `exports` to</span>
    <span class="md-code-comment">// collect exports.</span>
    <span class="md-code-keyword">if</span> (name === <span class="md-code-string">&apos;exports&apos;</span>) { <span class="md-code-keyword">return</span> exports; }
    <span class="md-code-keyword">else</span> { <span class="md-code-keyword">return</span> loader.get(name); }
  });

  callback(deps);
  <span class="md-code-keyword">return</span> exports;
}

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (options.metadata.type === <span class="md-code-string">&apos;amd&apos;</span>) {
    <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(extractAMDExports(<span class="md-code-keyword">this</span>, source));
  }
}
</code></pre> <p>To be clear, the particular implementation here is simple, and a real approach to AMD assimilation would be more complicated. This should provide some idea of what such an approach would look like.</p> <img class="" src="https://i.imgur.com/gl9TMbU.png"> <h3 id="importing-node-modules-from-es6-modules">Importing Node Modules from ES6 Modules</h3> <p>The approach to importing node modules from ES6 modules is similar. Consider a node version of the above module:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> underscoreRegex1 = <span class="md-code-regexp">/([a-z\d])([A-Z]+)/g</span>,
    underscoreRegex2 = <span class="md-code-regexp">/\-|\s+/g</span>;

exports.underscore = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(string)</span> </span>{
  <span class="md-code-keyword">return</span> string.replace(underscoreRegex1, <span class="md-code-string">&apos;$1_$2&apos;</span>)
               .replace(underscoreRegex2, <span class="md-code-string">&apos;_&apos;</span>)
               .toLowerCase();
}

exports.capitalize = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(string)</span> </span>{
  <span class="md-code-keyword">return</span> string.charAt(<span class="md-code-number">0</span>).toUpperCase() + string.substr(<span class="md-code-number">1</span>);
}
</code></pre> <p>You&#x2019;d override the hooks in a similar way:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> node = [<span class="md-code-string">&quot;string-utils&quot;</span>];

<span class="md-code-comment">// Resolve </span>
System.resolve = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(path, options)</span> </span>{
  <span class="md-code-keyword">if</span> (node.indexOf(path) &gt; -<span class="md-code-number">1</span>) {
    <span class="md-code-keyword">return</span> { name: path, metadata: { type: <span class="md-code-string">&apos;node&apos;</span> } };
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-keyword">return</span> { name: path, metadata: { type: <span class="md-code-string">&apos;es6&apos;</span> } };
  }
};

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">extractNodeExports</span><span class="md-code-params">(loader, source)</span> </span>{
  <span class="md-code-keyword">var</span> loader = <span class="md-code-keyword">new</span> Loader();
  <span class="md-code-keyword">return</span> loader.eval(`
    <span class="md-code-keyword">var</span> exports = {};
    ${source};
    exports;
  `);
}

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (options.metadata.type === <span class="md-code-string">&apos;node&apos;</span>) {
    <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(extractNodeExports(<span class="md-code-keyword">this</span>, source));
  }
}
</code></pre> <h3 id="importing-from-multiple-non-es6-modules">Importing From Multiple Non-ES6 Modules</h3> <p>To import from all three of these external module systems together, you would write a resolve hook that would store off the type of module in the context, and then use that information to evaluate the source appropriately in the <code class="md-code md-code-inline">link</code> hook.</p> <p>To make this process easier, a JavaScript library like <code class="md-code md-code-inline">require.js</code>, built for the ES6 loader, could provide conveniences for registering the type of external modules and assimilation code for <code class="md-code md-code-inline">link</code>.</p> <h3 id="import-a-single-export-from-a-non-es6-module">Import a &#x201C;Single Export&#x201D; From a Non-ES6 Module</h3> <p>Some external module systems support modules that have a single export, rather than a number of named exports.</p> <p>The techniques described above could be used to register that single export under a conventionally known name.</p> <p>Consider the following &#x201C;single export&#x201D; module using node-style modules:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// string-utils/capitalize.js</span>

<span class="md-code-built_in">module</span>.exports = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(string)</span> </span>{
  <span class="md-code-keyword">return</span> string.charAt(<span class="md-code-number">0</span>).toUpperCase() + string.substr(<span class="md-code-number">1</span>);
}
</code></pre> <p>In order to support using this module in an ES6 module, a loader can create a conventional name for the export that ES6 modules can import.</p> <p>In this example, we will name the export <code class="md-code md-code-inline">exports</code> for consistency with existing node practice. Once we have done this, ES6 modules will be able to import the module:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// app.js</span>

import { exports: capitalize } from <span class="md-code-string">&quot;string-utils/capitalize&quot;</span>;

<span class="md-code-built_in">console</span>.log(capitalize(<span class="md-code-string">&quot;hello&quot;</span>)) <span class="md-code-comment">// &quot;Hello&quot;</span>
</code></pre> <p>Here, we are renaming the conventionally named <code class="md-code md-code-inline">exports</code> to <code class="md-code md-code-inline">capitalize</code>.</p> <p>In order to achieve this, we will augment the earlier node assimilation code to handle <code class="md-code md-code-inline">module.exports =</code> semantics.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">extractNodeExports</span><span class="md-code-params">(loader, source)</span> </span>{
  <span class="md-code-keyword">var</span> loader = <span class="md-code-keyword">new</span> Loader();
  <span class="md-code-keyword">var</span> exports = loader.eval(`
    <span class="md-code-keyword">var</span> <span class="md-code-built_in">module</span> = {};
    <span class="md-code-keyword">var</span> exports = {};
    ${source};
    { single: <span class="md-code-built_in">module</span>.exports, named: exports };
  `);

  <span class="md-code-keyword">if</span> (exports.single !== <span class="md-code-literal">undefined</span>) {
    <span class="md-code-keyword">return</span> { exports: exports.single }
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-keyword">return</span> exports.named;
  }
}

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (options.metadata.type === <span class="md-code-string">&apos;node&apos;</span>) {
    <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(extractNodeExports(<span class="md-code-keyword">this</span>, source));
  }
}
</code></pre> <p>A similar approach could be used to allow assimilated AMD modules to have a &#x201C;single export&#x201D;.</p> <h3 id="importing-an-es6-module-from-a-node-module">Importing an ES6 Module From a Node Module</h3> <p>When using a node module, we would want to be able to import any other module, regardless of the source.</p> <p>One major benefit of the above approaches to importing non-ES6 modules is that it means that the standard <code class="md-code md-code-inline">System.get</code> will be able to load them.</p> <p>This means that it&#x2019;s easy to support <code class="md-code md-code-inline">require</code> in a node module: just alias it to <code class="md-code md-code-inline">System.get</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">extractNodeExports</span><span class="md-code-params">(loader, source)</span> </span>{
  <span class="md-code-keyword">var</span> loader = <span class="md-code-keyword">new</span> Loader();
  <span class="md-code-keyword">var</span> exports = loader.eval(`
    <span class="md-code-keyword">var</span> <span class="md-code-built_in">module</span> = {};
    <span class="md-code-keyword">var</span> exports = {};
    <span class="md-code-keyword">var</span> <span class="md-code-built_in">require</span> = System.get;
    ${source};
    { single: <span class="md-code-built_in">module</span>.exports, named: exports };
  `);

  <span class="md-code-keyword">if</span> (exports.single !== <span class="md-code-literal">undefined</span>) {
    <span class="md-code-keyword">return</span> { exports: exports.single }
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-keyword">return</span> exports.named;
  }
}
</code></pre> <h3 id="importing-an-amd-module-with-asynchronous-dependencies">Importing an AMD Module With Asynchronous Dependencies</h3> <p>In the above examples, we assumed that all dependencies in external modules are available synchronously, so we could use System.get in the <code class="md-code md-code-inline">link</code> hook.</p> <p>AMD modules can have asynchronous dependencies that can be determined without having to execute the module.</p> <p>For this use-case, you can return (from <code class="md-code md-code-inline">link</code>) a list of dependencies and a callback to call once the Loader has loaded the dependencies. The callback will receive the list of dependencies as parameters and must return a <code class="md-code md-code-inline">Module</code> instance.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> amd = [<span class="md-code-string">&apos;string-utils&apos;</span>];

System.resolve = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(path, options)</span> </span>{
  <span class="md-code-keyword">if</span> (amd.indexOf(path) !== -<span class="md-code-number">1</span>) {
    options.metadata = { type: <span class="md-code-string">&apos;amd&apos;</span> };
  } <span class="md-code-keyword">else</span> {
    options.metadata = { type: <span class="md-code-string">&apos;es6&apos;</span> };
  }
};

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (options.metadata.type !== <span class="md-code-string">&apos;amd&apos;</span>) { <span class="md-code-keyword">return</span>; }

  <span class="md-code-keyword">var</span> loader = <span class="md-code-keyword">new</span> Loader();
  <span class="md-code-keyword">var</span> [ imports, factory ] = loader.eval(`
    <span class="md-code-keyword">var</span> dependencies, factory;
    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">define</span><span class="md-code-params">(dependencies, factory)</span> </span>{
      imports = dependencies;
      factory = factory;
    }
    ${source};
    [ imports, factory ];
  `);

  <span class="md-code-keyword">var</span> exportsPosition = imports.indexOf(<span class="md-code-string">&apos;exports&apos;</span>);
  imports.splice(exportsPosition, <span class="md-code-number">1</span>);

  <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">execute</span><span class="md-code-params">(...args)</span> </span>{
    <span class="md-code-keyword">var</span> exports = {};
    args.splice(exportsPosition, <span class="md-code-number">0</span>, [exports]);
    factory(...args);
    <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(exports);
  }

  <span class="md-code-keyword">return</span> { imports: imports, execute: execute };
};
</code></pre> <p>Returning the imports and a callback from <code class="md-code md-code-inline">link</code> allows the <code class="md-code md-code-inline">link</code> hook to participate in the same two-phase loading process of ES6 modules, but using the AMD definition to separate the phases instead of ES6 syntax.</p> <img class="" src="https://i.imgur.com/AO3mEIm.png"> <h3 id="importing-a-node-module-by-processing-require-s">Importing a Node Module By Processing <code class="md-code md-code-inline">require</code>s</h3> <p>Because node modules use a dynamic expression for imports, there is no perfectly reliable way to ensure that all dependencies are loaded before evaluating the module.</p> <p>The approach used by <em>Browserify</em> is to statically analyze the file first for <code class="md-code md-code-inline">require</code> statements and use them as the dependencies. The <em>AMD CommonJS wrapper</em> uses a similar approach.</p> <p>The <code class="md-code md-code-inline">link</code> hook could be used to analyze Node-style packages for <code class="md-code md-code-inline">require</code> lines, and return them as <code class="md-code md-code-inline">imports</code>.</p> <p>By the time the <code class="md-code md-code-inline">execute</code> callback was called, all modules would be synchronously available, and aliasing <code class="md-code md-code-inline">require</code> to <code class="md-code md-code-inline">System.get</code> would continue to work.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">import { processImports } from <span class="md-code-string">&quot;browserify&quot;</span>;

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">var</span> imports = processImports(source);

  <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">execute</span><span class="md-code-params">()</span> </span>{
    <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(extractNodeExports(source));
  }

  <span class="md-code-keyword">return</span> { imports: imports, execute: execute};
};
</code></pre> <p>Of course, this means only works as long as no <code class="md-code md-code-inline">requires</code> are used with dynamic expressions, in a conditional, or in a <code class="md-code md-code-inline">try/catch</code>, but those are already limitations of systems like <em>Browserify</em>.</p> <img class="" src="https://i.imgur.com/7EMzTG2.png"> <h3 id="interoperability-in-general">Interoperability in General</h3> <p>Let&#x2019;s review the overall strategy used for assimilating non-ES6 module definitions:</p> <ul> <li>Non-ES6 modules can be loaded through the Loader by overriding the <code class="md-code md-code-inline">resolve</code> and <code class="md-code md-code-inline">link</code> hooks.</li> <li>Non-ES6 modules can asynchronously load other modules by return imports from <code class="md-code md-code-inline">link</code> and synchronously through <code class="md-code md-code-inline">System.get</code>.</li> </ul> <p>This means that all module systems can freely interoperate, using the Loader as an intermediary.</p> <p>For example, if an AMD module (say, &#x2018;app&#x2019;), depended on a Node-style module (say, &#x2018;string-utils&#x2019;):</p> <ol> <li>When loading <code class="md-code md-code-inline">app</code>, the <code class="md-code md-code-inline">link</code> hook would return <code class="md-code md-code-inline">{ imports: [&apos;string-utils&apos;], execute: execute }</code>.</li> <li>This would cause the Loader to attempt to load <code class="md-code md-code-inline">&apos;string-utils&apos;</code>, before it would call back the provided <code class="md-code md-code-inline">execute</code> callback.</li> <li>The Loader would fetch <code class="md-code md-code-inline">string-utils</code> and evaluate it using the Node-style <code class="md-code md-code-inline">link</code> hook.</li> <li>Once this is done, the provided <code class="md-code md-code-inline">execute</code> callback would run, receiving the <code class="md-code md-code-inline">string-utils</code> Module as a parameter.</li> <li>The <code class="md-code md-code-inline">execute</code> callback would then return a Module.</li> </ol> <p>This is just an illustrative example; any combination of module systems could freely interoperate through the Loader.</p> <h3 id="a-note-on-single-export-interoperability">A Note on &#x201C;Single Export&#x201D; Interoperability</h3> <p>Many of the existing module systems support mechanisms for exporting a single value instead of a number of named values from a module.</p> <p>At the current time, ES6 modules do not provide explicit support for this feature, but it can be emulated using the Loader. One specific strategy would be to export the single value as a well-known name (for example, <code class="md-code md-code-inline">exports</code>).</p> <p>Let&#x2019;s take a look at how a Loader could support a Node-style module using <code class="md-code md-code-inline">require</code> to import the &#x201C;single export&#x201D; of another Node-style module.</p> <p>This same approach would support interoperability between module systems that support importing and exporting of single values.</p> <p>We&#x2019;ll need to enhance the previous solution we provided for this scenario:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> isSingle = <span class="md-code-keyword">new</span> Symbol();

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">extractNodeExports</span><span class="md-code-params">(loader, source)</span> </span>{
  <span class="md-code-keyword">var</span> loader = <span class="md-code-keyword">new</span> Loader();
  <span class="md-code-keyword">var</span> exports = loader.eval(`
    <span class="md-code-keyword">var</span> <span class="md-code-built_in">module</span> = {};
    <span class="md-code-keyword">var</span> exports = {};
    <span class="md-code-keyword">var</span> <span class="md-code-built_in">require</span> = System.get;
    ${source};
    { single: <span class="md-code-built_in">module</span>.exports, named: exports };
  `);

  <span class="md-code-keyword">if</span> (exports.single !== <span class="md-code-literal">undefined</span>) {
    <span class="md-code-keyword">return</span> { exports: exports.single, [isSingle]: <span class="md-code-literal">true</span> };
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-keyword">return</span> exports.named;
  }
}

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (options.metadata.type === <span class="md-code-string">&apos;node&apos;</span>) {
    <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(extractNodeExports(<span class="md-code-keyword">this</span>, source));
  }
}
</code></pre> <p>Here, we create a new unique Symbol that we will use to tag a module as containing a single export. This will avoid conflicts with Node-style modules that export the name <code class="md-code md-code-inline">exports</code> explicitly.</p> <p>Next, we will need to enhance the code that we have been using for Node-style <code class="md-code md-code-inline">require</code>. Until now, we have simply aliased it to <code class="md-code md-code-inline">System.get</code>. Now, we will check for the <code class="md-code md-code-inline">isSingle</code> symbol and give it special treatment in that case.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// this assumes that the `isSingle` Symbol is in scope</span>
<span class="md-code-keyword">var</span> <span class="md-code-built_in">require</span> = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(name)</span> </span>{
  <span class="md-code-keyword">var</span> <span class="md-code-built_in">module</span> = System.get(name);
  <span class="md-code-keyword">if</span> (<span class="md-code-built_in">module</span>[isSingle]) {
    <span class="md-code-keyword">return</span> <span class="md-code-built_in">module</span>.exports;
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-keyword">return</span> <span class="md-code-built_in">module</span>;
  }
}
</code></pre> <p>This same approach, using a shared <code class="md-code md-code-inline">isSingle</code> symbol, could be used to support interoperability between AMD and Node single exports.</p> <p>As described earlier, ES6 modules would use <code class="md-code md-code-inline">import { exports: underscore } from &apos;string-utils/underscore&apos;</code>.</p> <h2 id="configuration-of-existing-loaders">Configuration of Existing Loaders</h2> <p>The <code class="md-code md-code-inline">requirejs</code> loader has a number of useful configuration options that its users can use to control the loader.</p> <p>This section covers a sampling of those options and how they map onto the semantics of the ES6 Loader. In general, the compilation pipeline provides hooks that can be used to implement these configuration options.</p> <h3 id="base-url">Base URL</h3> <p>The <code class="md-code md-code-inline">requirejs</code> loader allows the user to configure a base URL for resolving relative paths.</p> <p>In the default browser loader, the base URL will default to the page&#x2019;s base URL. The default <code class="md-code md-code-inline">System.resolve</code> will prefix that base URL and append <code class="md-code md-code-inline">.js</code> to the end of the module name (if not already present).</p> <p>The browser&#x2019;s default Loader (<code class="md-code md-code-inline">window.System</code>) will also include a <code class="md-code md-code-inline">baseURL</code> configuration option that controls the base URL for its implementation of <code class="md-code md-code-inline">resolve</code>.</p> <p>JavaScript code could also configure the Loader&#x2019;s resolve hook to provide any policy they like:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">var resolve = System.resolve;

System.resolve = function(name, ...args) {
  if (name.match(/fun/)) {
    return `/assets/javascripts/${name}.js&quot;
  }
  return resolve(name, ...args);
};
</code></pre> <h3 id="url-arguments">URL Arguments</h3> <p>Similarly, the <code class="md-code md-code-inline">requirejs</code> loader allows the specification of additional URL arguments. This could also be handled by overriding the <code class="md-code md-code-inline">resolve</code> hook.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> resolve = System.resolve;

System.resolve = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(...args)</span> </span>{
  <span class="md-code-keyword">return</span> resolve(name, ...args) + <span class="md-code-string">&quot;?bust=&quot;</span> + (<span class="md-code-keyword">new</span> <span class="md-code-built_in">Date</span>().getTime());
};
</code></pre> <h3 id="timeouts">Timeouts</h3> <p>The <code class="md-code md-code-inline">requirejs</code> loader allows the specification of a timeout before rejecting the request.</p> <p>With the ES6 Loader, the <code class="md-code md-code-inline">fetch</code> hook can be overridden to reject the fetch after some time has passed.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> fetch = System.fetch;

System.fetch = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(url, options)</span> </span>{
  setTimeout(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span> </span>{
    options.reject(<span class="md-code-string">&quot;Timeout&quot;</span>);
  }, <span class="md-code-number">5000</span>);

  fetch(url, options);
};
</code></pre> <h3 id="support-for-legacy-modules">Support for Legacy Modules</h3> <p>The <code class="md-code md-code-inline">requirejs</code> loader provides a mechanism for declaring how a legacy module should be interpreted:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">requirejs.config({
  shim: {
    backbone: {
      deps: [<span class="md-code-string">&apos;underscore&apos;</span>, <span class="md-code-string">&apos;jquery&apos;</span>]
      exports: <span class="md-code-string">&apos;Backbone&apos;</span>
    },
  }
});
</code></pre> <p>The example above under <strong>Using Existing Libraries as Modules</strong> shows one approach to this problem. That approach should work generically, without having to list a specific export name.</p> <p>The <code class="md-code md-code-inline">link</code> hook provides a way to define dependencies for legacy modules.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> config = {
  backbone: {
    deps: [<span class="md-code-string">&apos;underscore&apos;</span>, <span class="md-code-string">&apos;jquery&apos;</span>],
    exports: [<span class="md-code-string">&apos;Backbone&apos;</span>]
  }
}

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">executeCallback</span><span class="md-code-params">(source, exportNames)</span> </span>{
  System.eval(source);
  <span class="md-code-keyword">var</span> exports = {};
  exportNames.forEach(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(name)</span> </span>{
    exports[name] = System.global[name]
  });
  <span class="md-code-keyword">return</span> <span class="md-code-keyword">new</span> Module(exports);
}

System.link = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(source, options)</span> </span>{
  <span class="md-code-keyword">if</span> (!config[options.normalized]) { <span class="md-code-keyword">return</span>; }

  <span class="md-code-keyword">var</span> { deps, exports: exportNames } = config[options.normalized];

  <span class="md-code-keyword">if</span> (moduleConfig) {
    <span class="md-code-keyword">return</span> {
      imports: moduleConfig.deps,
      execute: executeCallback(source, exportNames);
    }
  }
};
</code></pre> <h2 id="referencing-modules-in-html">Referencing Modules in HTML</h2> <p>In Ember.js, Angular.js, and other contemporary frameworks, JavaScript objects are referenced in HTML templates:</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-comment">&lt;!-- ember.js --&gt;</span>
{{#view App.FancyButton}}
<span class="md-code-tag">&lt;<span class="md-code-title">p</span>&gt;</span>Fancy Button Contents<span class="md-code-tag">&lt;/<span class="md-code-title">p</span>&gt;</span>
{{/view}}
</code></pre> <p>Here, the app is asking Ember.js to render some HTML defined in an <code class="md-code md-code-inline">App.FancyButton</code> constructor. Note that Ember encourages the use of a global namespace for coordination between JavaScript and HTML templates.</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-comment">&lt;!-- angular --&gt;</span>
<span class="md-code-tag">&lt;<span class="md-code-title">button</span> <span class="md-code-attribute">fancy-button</span>&gt;</span>
  <span class="md-code-tag">&lt;<span class="md-code-title">p</span>&gt;</span>Fancy Button Contents<span class="md-code-tag">&lt;/<span class="md-code-title">p</span>&gt;</span>
<span class="md-code-tag">&lt;/<span class="md-code-title">button</span>&gt;</span>
</code></pre> <p>Here, the app is asking Angular.js to replace the <code class="md-code md-code-inline">&lt;button&gt;</code> with some content defined in a globally registered <code class="md-code md-code-inline">fancy-button</code> directive.</p> <p>Both Angular and Ember both use globally registered names to define <em>controller</em> objects to attach to parts of the HTML controlled by the framework.</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-comment">&lt;!-- ember --&gt;</span>
{{control &quot;fancy-button&quot;}}
</code></pre> <p>Here, the app is asking Ember.js to render some HTML defined in an <code class="md-code md-code-inline">App.FancyButtonView</code> and use an instance of the <code class="md-code md-code-inline">App.FancyButtonController</code> as its controller. Again, Ember is relying on a globally rooted namespace for coordination.</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-comment">&lt;!-- angular --&gt;</span>
<span class="md-code-tag">&lt;<span class="md-code-title">div</span> <span class="md-code-attribute">ng-controller</span>=<span class="md-code-value">&quot;TodoCtrl&quot;</span>&gt;</span>
  <span class="md-code-tag">&lt;<span class="md-code-title">span</span>&gt;</span>{{remaining()}} of {{todos.length}} remaining<span class="md-code-tag">&lt;/<span class="md-code-title">span</span>&gt;</span>
<span class="md-code-tag">&lt;/<span class="md-code-title">div</span>&gt;</span>
</code></pre> <p>Here, the app is asking Angular to use a globally rooted object called <code class="md-code md-code-inline">TodoCtrl</code> as the controller for this part of the HTML. In Angular, this controller is used to control the scope for data-bound content nested inside of its element.</p> <p>To handle the kind of situation where a module is referenced by a String and needs to be looked up dynamically, ES6 modules provide an API for looking up a module at runtime.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">System.get(<span class="md-code-string">&apos;controllers/fancy-button&apos;</span>);
</code></pre> <p>Systems like Ember or Angular could use this API to allow their users to reference a module&#x2019;s exports in HTML.</p> <p>In the first Ember example, instead of referencing a globally rooted constructor, the HTML would reference a module name:</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-comment">&lt;!-- ember.js --&gt;</span>
{{#view views/fancy-button}}
<span class="md-code-tag">&lt;<span class="md-code-title">p</span>&gt;</span>Fancy Button Contents<span class="md-code-tag">&lt;/<span class="md-code-title">p</span>&gt;</span>
{{/view}}
</code></pre> <p>And the module would look like:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// views/fancy-button.js</span>
import { View } from <span class="md-code-string">&quot;ember&quot;</span>;

export <span class="md-code-keyword">let</span> view = View.extend({
  <span class="md-code-comment">// contents</span>
});
</code></pre> <p>The second Angular example could be rewritten as:</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-comment">&lt;!-- angular --&gt;</span>
<span class="md-code-tag">&lt;<span class="md-code-title">div</span> <span class="md-code-attribute">ng-controller</span>=<span class="md-code-value">&quot;controllers/todo&quot;</span>&gt;</span>
  <span class="md-code-tag">&lt;<span class="md-code-title">span</span>&gt;</span>{{remaining()}} of {{todos.length}} remaining<span class="md-code-tag">&lt;/<span class="md-code-title">span</span>&gt;</span>
<span class="md-code-tag">&lt;/<span class="md-code-title">div</span>&gt;</span>
</code></pre> <p>And the JavaScript:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// controllers/todo.js</span>

export <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">Controller</span><span class="md-code-params">($scope)</span> </span>{
  <span class="md-code-comment">// contents</span>
}
</code></pre> <p>The general pattern is to switch from globally rooted namespaces to named, registered modules. <code class="md-code md-code-inline">System.get</code> provides a way to dynamically look up already loaded modules.</p> <h2 id="creating-modules-from-html">Creating Modules from HTML</h2> <p>The new Web Components specification provides a way to create a JavaScript constructor through HTML:</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-tag">&lt;<span class="md-code-title">element</span> <span class="md-code-attribute">extends</span>=<span class="md-code-value">&quot;button&quot;</span> <span class="md-code-attribute">name</span>=<span class="md-code-value">&quot;x-fancybutton&quot;</span> <span class="md-code-attribute">constructor</span>=<span class="md-code-value">&quot;FancyButton&quot;</span>&gt;</span>
  <span class="md-code-tag">&lt;<span class="md-code-title">script</span>&gt;</span><span>
    FancyButton.prototype.razzle = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
    };
    FancyButton.prototype.dazzle = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
    };
  </span><span class="md-code-tag">&lt;/<span class="md-code-title">script</span>&gt;</span>
<span class="md-code-tag">&lt;/<span class="md-code-title">element</span>&gt;</span>
</code></pre> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// app.js</span>

<span class="md-code-keyword">var</span> b = <span class="md-code-keyword">new</span> FancyButton();
b.textContent = <span class="md-code-string">&quot;Show time&quot;</span>;
<span class="md-code-built_in">document</span>.body.appendChild(b);
b.addEventListener(<span class="md-code-string">&quot;click&quot;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(event)</span> </span>{
    event.target.dazzle();
});
b.razzle();
</code></pre> <p>Here, the <code class="md-code md-code-inline">&lt;element&gt;</code> tag is creating a globally rooted name for the constructor.</p> <p>The specifics will probably vary in practice, but something like this could work:</p> <pre class="md-code-block"><code class="md-code">
</code></pre> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// app.js</span>

import { Element: FancyButton } from <span class="md-code-string">&quot;web/x-fancybutton&quot;</span>

<span class="md-code-keyword">var</span> b = <span class="md-code-keyword">new</span> FancyButton();
b.textContent = <span class="md-code-string">&quot;Show time&quot;</span>;
<span class="md-code-built_in">document</span>.body.appendChild(b);
b.addEventListener(<span class="md-code-string">&quot;click&quot;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(event)</span> </span>{
    event.target.dazzle();
});
b.razzle();
</code></pre> <blockquote> <p>Originally <a href="http://nodejs.org/api/fs.html" target="_blank">posted by wycats as a gist on GitHub</a>, but I felt it deserved to get more public attention. Resources describing ES6 modules are scarce, and <strong>rarely this detailed</strong>. Head over to the gist for an interesting discussion on the state of ES6 modules.</p> </blockquote> <p>Related links:</p> <ul> <li><a href="http://restrictmode.org/" target="_blank">ECMAScript 6 modules: the future is now</a></li> <li><a href="http://www.2ality.com/2013/11/es6-modules-browsers.html" target="_blank" aria-label="ECMAScript 6 modules in future browsers">ECMAScript 6 modules in future browsers</a></li> <li><a href="https://github.com/google/traceur-compiler" target="_blank" aria-label="Traceur Compiler">Traceur Compiler</a></li> <li><a href="https://github.com/aaronfrost/grunt-traceur" target="_blank" aria-label="grunt-traceur on GitHub"><code class="md-code md-code-inline">grunt-traceur</code></a></li> <li><a href="http://kangax.github.io/es5-compat-table/es6/" target="_blank" aria-label="ES6 Compatibility Table">ES6 Compatibility Table</a></li> <li><a href="https://github.com/bevacqua/buildfirst/tree/latest/ch05/17_harmony-traceur" target="_blank" aria-label="Harmony Samples using Google Traceur Compiler">ES6 Code Samples in my upcoming JavaScript Application Design book</a></li> </ul> <p>If you want to toy around with the syntax, <a href="http://typescript.codeplex.com/" target="_blank">ModuleLoader/es6-module-loader</a> is the most up-to-date polyfill I could find. Believe me, <em>I looked</em>.</p></div>
