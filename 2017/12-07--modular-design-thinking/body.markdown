<hr>

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-0]
:::

::: .mde-inline.mde-66
Hey everyone!
Today I’m going to speak about Modular Design.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-1]
:::

::: .mde-inline.mde-66
But first, a little bit about me:
My name is Nicolás Bevacqua, but you can find me on Twitter as [@nzgb][nzgb].
I just tweeted a link to the slides if you want to follow along or check them out later on your own.
I’m a Senior Software Engineer at [Elastic][elastic], the company behind `elasticsearch` and Kibana.
I also run a blog and a newsletter at ponyfoo.com.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-2]
:::

::: .mde-inline.mde-66
My third book, Mastering Modular JavaScript, is available as an Early Release with O'Reilly.
It’s part of a five book series exploring JavaScript architecture.
This book explores the fundamentals of software complexity, and how those concepts can be applied in JavaScript to obtain modular applications that favor maintainability and readability.
The book also comes with lots of straightforward advice and practical examples.
It's free to read online, and you can find it at ponyfoo.com/books.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-3]
:::

::: .mde-inline.mde-66
Let's start with a brief history lesson on modularity.
When compared with any other programming environment, the web has had a very particular progression towards a module system.
The web is a unique execution environment. Scripts are loaded by way of HTML `<script>` tags, and we don't have a concept of namespaces. There wasn't a concept of modules for a long time, and today we're barely starting to scratch the surface of truly modular JavaScript.
When we look at most other environments or languages, there's a standard library that's broken into different modules you can use. There are namespaces.
In other platforms, modules existed since day one: they're just files.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-4]
:::

::: .mde-inline.mde-66
In the early days, JavaScript was embedded in HTML `<script>` tags, or inlined in attributes like `onclick` handlers.
At best, scripts were saved to one or more files, but all of those still shared a global scope or namespace. 
Any variables declared at the top of one of these files or inline scripts would be imprinted on the global `window` object, leading to situations where a variable in one script inadvertently replaced a value that another script was relying on, or creating unexpected conditions that changed the flow of a JavaScript program.
This wasn't so much of a problem at first, where JavaScript was mostly used as a toy language to add confirmation alerts before form submissions, to swap images on mouse over, and so on.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-5]
:::

::: .mde-inline.mde-66
But language adoption grew, and so did the need for a solution that would mitigate variable clashes.
We found a solution in immediately-invoking function expressions (or IIFE).
Variable bindings that are declared in a function are scoped to that function, and thanks to the self-invoking nature of these closures, we didn't have to do anything other than wrap our files with these expressions.
Given we're not dealing with implicit globals anymore, a perk of this approach was that minifiers could now minify more variables.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-6]
:::

::: .mde-inline.mde-66
Using a similar expression, we can return a value from the closure and assign that result to a variable.
This pattern allowed us to build libraries with private members, meaning the state of our library was not observable from the outside, beyond what was revealed by the API we'd offer.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-7]
:::

::: .mde-inline.mde-66
Besides global namespace pollution, another big issue with script tags was the lack of a formal dependency graph.
In other words, you had to carefully sort your script tags, making sure the most depended upon scripts, like jQuery, came before the scripts that depended on them.
This was particularly annoying, because whenever you added a new file, you had to evaluate where exactly in the list of script tags it should be placed.
Concatenation didn't resolve the problem either, because you still had to provide the concat tool with a sorted list of file names or patterns.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-8]
:::

::: .mde-inline.mde-66
The next generation of modular JavaScript also involved using functions to obtain self-contained scopes.
Instead of consuming dependencies from the global object, we received them as the parameters of module functions.
These functions were capable of exposing a public interface by returning an object or any other JavaScript value.
With both RequireJS and the Dependency Injection mechanism in Angular, we only needed to indicate the entry point, which would have a list of dependencies. Those dependencies would have their own dependencies, and so on.
Armed with this information, these module systems were able to put together a dependency graph automatically, and the hassle of having to maintain a large, carefully sorted list of every module in our application disappeared.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-9]
:::

::: .mde-inline.mde-66
Part of the problem with RequireJS was that they offered half a dozen competing ways of declaring dependencies and exposing an API, which only created confusion.
They also encouraged asynchronous loading out the box, which was an anti-pattern in production and demanded extra configuration to optimize so that asset loading didn't take a very long time.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-10]
:::

::: .mde-inline.mde-66
Angular presented similar problems.
They encouraged relying on automagically parsing function parameter names to figure out a module's dependencies. This wasn't the safest bet, as minifiers would rename those parameter names, often breaking your production builds.
Instead of encouraging users to use a safe array format, a complicated build tool was introduced that would transform your Angular module functions into the array format, which wouldn't break when minified.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-11]
:::

::: .mde-inline.mde-66
This kind of over-engineering presented problems for teams that just wanted to get their products out, when they were transitioning to production.
At the same time, both RequireJS and Angular module declarations were quite verbose. This contrasted with IIFE modules, which were much more succinct.
Even with these issues, both systems were an improvement over using plain script tags, because of their knowledge of the dependency graph.
This enabled us to create and use new tools, that were able to make informed decisions on how to deal with the codebase without a ton of upfront configuration.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-12]
:::

::: .mde-inline.mde-66
CommonJS, the module system in Node, changed the game.
NodeJS ran on servers, without being limited by the heavy security restrictions imposed on web browsers.
For the first time we had a file-based module system.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-13]
:::

::: .mde-inline.mde-66
These modules had their own local scopes. In CommonJS, we could declare dependencies synchronously in code through `require` calls, and we also could expose an API assigning values to the local `exports` binding.
CommonJS didn't allow implicit access to the global object, and required explicit references to a `global` binding instead.
Each file was treated as a module, and since they would be internally wrapped by the system, we didn't need to wrap our code in function expressions anymore, keeping verbosity to a minimum.
CommonJS was clearly superior when compared against RequireJS or the Angular model, but it was only available on NodeJS environments.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-14]
:::

::: .mde-inline.mde-66
A tool called `browserify` brought a new wave of innovation when it launched.
For the first time, we were able to share code between the server and the client without expending a lot of effort.
This was appealing because our client-side applications would be able to leverage the thousands of small modules published to the npm registry.
Seamlessly sharing code between server and client opened up a world of possibilities, like starting to render a page on the server and then transitioning to the client-side for routing.
More importantly, the diverging NodeJS and client-side ecosystems were able to feed on each other's innovations, hoisting both ecosystems upwards through tooling and knowledge sharing.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-15]
:::

::: .mde-inline.mde-66
Tools like Babel, Webpack, and Rollup have taken us to higher highs.
The Babel transpiler means developers start relying on new language features long before they get native browser implementations. This results in shorter feedback loops between spec writers, implementors, and end users.
At the same time, it reduces the likelihood of browsers coming up with conflicting implementations, since that would result in errors when users stop transpiling that feature, once there's sufficient cross-browser support.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-16]
:::

::: .mde-inline.mde-66
ES Modules follow a model similar to CommonJS, except that exports are always static, and static imports are highly encouraged over dynamic imports.
This preference for statically defined module shapes is a great boon for tools using static analyzers to learn about a bundle's dependency graph, unused code paths, and so on.
In contrast with other module systems we've seen so far, ES Modules are native to the language.
There are two sides to the ESM equation. On the one hand, there's the module syntax.
On the other, there's the delivery mechanism: in Node we can load module dependencies synchronously by accessing the file system.
In the web, it's not that simple, but we'll come back to that topic in a minute.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-17]
:::

::: .mde-inline.mde-66
In the category where we used to have mostly just Google Closure Compiler and UglifyJS, we can now also count on tools like Babel, Rollup, and Prepack.
If we take a step back and look at the current landscape of increasingly complex build toolchains, it's only natural that new and more effective tools will be introduced in this category as well.
Tools like Rollup and Prepack are particularly interesting, because they concede we're going to write suboptimal code, and optimize our build outputs based on that principle.
Rollup analyzes ESM import and export statements, figuring out which bindings are actually being consumed somewhere else in our bundle, discarding the rest in order to reduce bundle size.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-18]
:::

::: .mde-inline.mde-66
Prepack takes a different approach, where it tries to speed up the startup time of our bundles.
In order to do so, Prepack sports a full-fledged interpreter, capable of actually executing our input.
Armed with that interpreter and a few heuristic rules, Prepack is able to transform code paths that produce a constant outcome into something that's optimized for production JavaScript VMs.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-19]
:::

::: .mde-inline.mde-66
Going back to script loading, we can use `<script type='module'>` tags to define the entry point to an ESM-based application.
These scripts are deferred by default, which prevents blocking of HTML parsing while delaying script execution until the document has finished parsing.
Naturally, we can't just throw a script with hundreds of dependencies into a script tag, particularly when we keep in mind that each package dependency we introduce might involve hundreds of files and more dependencies. This would thrash the network and even the browser IPC layer. It's not a practical thing to do, and the situation is even worse if we factor in mobile devices.
Bundling is not going away for the foreseeable future. If anything, build tools will continue to grow in variety and capabilities.
The good news is that there's healthy browser support for `<script type='module'>`. The world's largest runtime platform finally has proper, native modules.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-20]
:::

::: .mde-inline.mde-66
What makes modular architecture so important that we've spent the last several minutes discussing the different approaches?
What exactly are the benefits of breaking down large applications into small components?
Let's name a few.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-21]
:::

::: .mde-inline.mde-66
No more globals. This means we don't have to be afraid of clashes, or worry about coming up with naming schemes that mitigate global namespace pollution.
This is the main reason why CSS-in-JS solutions exist: CSS is global by default, like JS in its early days.
We might eventually see CSS naturally evolve into a practical and scoped module system akin to ESM. Web Components and the shadow DOM may be part of that solution.
But let's get back to the benefits of modular JavaScript.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-22]
:::

::: .mde-inline.mde-66
Complexity. This is the silent killer. When complexity creeps unabated, our applications become increasingly harder to read and understand.
Applications that are hard to understand or reason about are also hard to change. The team suddenly approaches everyday problems with a mentality like "who knows what might break if we touch this". This grinds productivity down to a halt.
Modules can help here. By containing complexity within modules, we don't need to concern ourselves with the entirety of our codebase.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-23]
:::

::: .mde-inline.mde-66
When we think about eating a cheeseburger, we don't think about every interaction. We don't need to think about our hands, the physics that let us move them to grab a bite, synapses, chewing, or anything else. We just focus on the delicious meat, melted cheese, and soft bread buns.
We don't stop to think about what exactly makes us believe this burger is delicious either.
We're pattern recognition machines. We can recursively abstract away a great deal of complexity behind other abstractions.
Virtually all we do in life is create new abstractions to group a series of patterns we have previously internalized.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-24]
:::

::: .mde-inline.mde-66
Here's a quote from Sindre Sorhus, a prolific package author with hundreds of open-source projects on npm.
It's all about containing complexity. Think of your modules as Lego blocks.
You don't necessarily care about the details of how each block is made.
All you need to know is how to use the Lego blocks to build your Lego castle.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-25]
:::

::: .mde-inline.mde-66
In the JavaScript world, modules offer the ability of hiding as much complexity as we please behind a clearly defined interface.
We can then group together a bunch of these modules, and abstract them behind a module that's not necessarily larger, but that may have a lot of dependencies.
We then call that highly depended upon module a package, and it becomes the only interface we concern ourselves with.
We can keep doing this as much as needed, and at any point we only need to understand the top level interface.
If we detect a problem, we can dig deeper and inspect the implementation, but as long as the interface holds, we won't have to worry about that.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-26]
:::

::: .mde-inline.mde-66
To learn modularization, we need to understand how to break down complexity.
When stripped down, modules are nothing more than knowing where to rip a piece of code and put it somewhere else. Behind an interface.
An effective way of breaking down complexity is thinking in terms of the aspects of a task.
When we're looking at a large chunk of a program, is everything we're doing dealing with the same aspect of the problem? Or is part of the solution going off and pulling off an email template, preparing a model, and sending out an email?
That aspect of the solution might belong elsewhere, and we could replace all that code with a single function call that calls into the email service.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-27]
:::

::: .mde-inline.mde-66
When we start tackling problems this way, we'll start to notice there's two fundamental purposes a piece of code could be serving.
It could be solving a specific aspect of a broad variety of use cases, like sending emails or generating HTML out of Markdown.
Or it could be dealing with a specific flow control, such as user registration, where we ask a service to validate the request body, ask another service to insert the User in our database, ask another service to shoot off a verification email, and finally respond to the request.
Surely if the flow is simple enough, we might not need to worry about moving each aspect out of the way and into its own reusable functions, but even moving these aspects into named functions that identify them as such can help make our code so much more readable.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-28]
:::

::: .mde-inline.mde-66
Now, code flows are usually where we pack the meat of our business logic. This code is unlikely to have value when it comes to reusability.
Aspects on the other hand are much more reusable.
For instance, Converting Markdown into HTML is almost ubiquitous in modern JavaScript applications.
By creating a clear separation between aspects and flows, we're able to maximize reusability for these portions of code that, coincidentally, deal with a particular aspect of our system.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-29]
:::

::: .mde-inline.mde-66
At the same time, having clearly delimited aspects helps us compose small concerns into larger solutions that help us attack bigger problems.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-30]
:::

::: .mde-inline.mde-66
This way we can focus on the deliciousness of our burger, without worrying about what exactly we're doing with our hands or how our synapses are performing.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-31]
:::

::: .mde-inline.mde-66
Let's get more specific.
How do we write great modules?
What should we focus our time and attention on?
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-32]
:::

::: .mde-inline.mde-66
This one can't be stressed enough. Our components should focus on a single thing they should do well.
We should strive to keep functions as focused as possible, dealing with just one aspect of the responsibility our component aims to take ownership of.
When a function is primarily in charge of the flow of our application, it shouldn't concern itself with how each branch of said flow is executed.
Instead, that responsibility should be offloaded to other functions or components.
Having such a clear separation of concerns greatly improves readability and our ability to effect change to portions of the system even if we don't have the faintest idea of how the rest of the application works.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-33]
:::

::: .mde-inline.mde-66
When designing modular components, it's best to start with the API.
Our process usually starts with a module we want because we need to use it.
If we could build out anything, how would the ideal API we'd like to consume look like?
Then, we need to take that further. How would that API look for the most frequent use case? What about for the most advanced use cases?
Focusing on making common use cases accessible leads to cleaner API design that optimizes readability on the consumer's side while not necessarily compromising on our API's own usability.
Once we've settled for an interface we like, _then_ we can worry about the implementation.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-34]
:::

::: .mde-inline.mde-66
Our API should only expose things consumers absolutely need.
Anything we add beyond the absolutely necessary is just unnecessary complexity.
Making the API surface larger is something we can always do with relative ease, but removing bits from the API is not that simple.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-35]
:::

::: .mde-inline.mde-66
In a similar way to how we expect patterns to be used consistently throughout a codebase, consumers expect consistency in the different APIs they consume.
As we add more modules to an application layer, a package, or to an entire project, we need to take into account other modules in that same system.
For example, if three interfaces expect currency values in cents, we'd need a _very_ good reason to create a new method that expects currency values in units instead of cents.
Consistency helps us avoid surprises.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-36]
:::

::: .mde-inline.mde-66
What we're trying to do is to avoid ambiguity.
An effective way to do that is to rely on the same interface shapes every time.
Take jQuery for example.
You can pass in a selector, a DOM element, an array of DOM elements, a NodeList, a jQuery object, or a plain JavaScript object, and optionally a context element.
Regardless of your input, the output always is a jQuery object that wraps whatever input you provided, and that jQuery object shares a common set of methods you can call on it.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-37]
:::

::: .mde-inline.mde-66
This boils down to being flexible in inputs as long as we can map them to outputs with a consistent shape.
Regardless of what an interface receives, it must always respond in a predictable way.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-38]
:::

::: .mde-inline.mde-66
Whatever you put into the burger, you expect to be able to eat it all the same.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-39]
:::

::: .mde-inline.mde-66
Always strive for simplicity.
The simpler our interface is, the easier it is to consume, the more feedback we get, the better we can make our interface.
In order to optimize for simplicity, cater to the most frequent use cases, while hiding infrequently used features behind an options object.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-40]
:::

::: .mde-inline.mde-66
Naturally, we need to write tests.
The interface is all our consumers care about.
Write tests against the interface, too. If we get the appropriate output for the inputs provided in each use case, that's good enough.
This kind of thinking frees us to write the underlying implementation of these interfaces as we see fit, provided that tests pass.
At the same time, tests become less brittle.
We stop worrying about whether an underlying routine was "called once", meaning we need to change tests only when we make changes to the interface or uncover actual bugs.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-41]
:::

::: .mde-inline.mde-66
Sane documentation is a key aspect of proper modular design.
Documentation helps consumers, but it can also help us identify usability issues.
If a consumer has to peek into the implementation to understand what an interface does or how it works, then that means we need better documentation or a simpler interface.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-42]
:::

::: .mde-inline.mde-66
Abstractions are a useful complement to modular architectures.
They allow us to establish reusable patterns that lead to consistent and simpler interfaces.
For example, event emitters are a great solution to event handling that works well across modules.

:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-43]
:::

::: .mde-inline.mde-66
We already discussed how -- in a sense -- _everything_ is an abstraction. That includes modules.
In modules we have a simple interface hiding an implementation that's more complex. 
Abstractions are a powerful tool. But abstractions also present downsides. 
It's easy to fall into the trap where we think they will solve all of our problems.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-44]
:::

::: .mde-inline.mde-66
If we add too many abstractions, it can quickly get to a point where nobody really understands what they're doing, and that translates into a burden when we need to onboard new developers to the team but can't really justify or even explain the abstractions we bought into.
If we add the wrong abstractions, we might have to shape code so that it conforms to an abstraction that doesn't really apply to the problems we're trying to solve.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-45]
:::

::: .mde-inline.mde-66
This can happen when we create abstractions too prematurely, before we get a chance to properly weigh the advantages and disadvantages of hiding code behind that abstraction.
To remediate this issue, we should give abstractions some considered thought before buying into them.
Allow patterns to emerge naturally before categorically deciding that they deserve to be abstracted away so that we can avoid repeated logic.
While some light code repetition in our codebase might be bad, it would be far more expensive to settle for an abstraction that doesn't take into account the ample majority of potential use cases.
A bad abstraction could end up warping what would otherwise be simple code, adding unnecessary complexity to our application.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-46]
:::

::: .mde-inline.mde-66
Finding the right abstractions can be tricky, but it's important to give ourselves time to do so.
Save yourself and your team mates from a world of pain, while keeping code readable in the process.
In short, abstractions are an excellent tool when leveraged carefully, but mistakes can be costly and hard to roll back, so try not to over-commit early on.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-47]
:::

::: .mde-inline.mde-66
For the last part of this presentation, I wanted to share some tips on how we can write modules that are easier to read.
When writing code, always try to keep in mind the human beings that will need to read and make sense of what you wrote.
Things like accessibility, performance, and even security are second to collaboration and your ability to iterate.
If your team can't collaborate, you'll have failed your users, regardless of how great you think your accessibility, performance, or security measures are.
If you can't collaborate, that means you can't iterate on making your products better.
And that will almost assuredly lead to accessibility, performance, and security bugs.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-48]
:::

::: .mde-inline.mde-66
In that light, my first tip is pretty obvious. Stop writing clever code.
Clever code is a misnomer for code that only a handful of the engineers in a team understand.
While clever code might resolve an urgent issue or fix a performance problem, it can just as easily lead to unforeseen bugs.
Instead of writing clever code, unpack it into something that's easier for everyone to understand. You will thank yourself in the future.
If all else fails, leave thorough comments so that developers in the future can understand why the code was written the way it was.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-49]
:::

::: .mde-inline.mde-66
We often use a lot less variables than we should.
Using more variables makes it easier to step through code, not just with a debugger but also with our minds, as we're reading the code and executing it in our heads.
Instead of going for clever code and trying to cram all our logic into a single line of code, let's instead use more variables to store intermediate results.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-50]
:::

::: .mde-inline.mde-66
Arrow functions with an implicit return look great when they're simple and concise.
Avoid falling into the trap of justifying their use by complicating what would've been a simple regular old function.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-51]
:::

::: .mde-inline.mde-66
Instead of nesting conditionals, optimize for bailing early. Let's look at some code.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-52]
:::

::: .mde-inline.mde-66
I'm sure you've seen code like this.
The happy path is near the top of the function, while the error cases are near the bottom.
The problem with this approach is that, as we read the code, we need to keep more and more state in our heads.
Then when branches are finally finished, we need to remind ourselves what piece of state we can let go of. We're not built like this.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-53]
:::

::: .mde-inline.mde-66
It's far easier if you flip the conditionals and instead "guard" against your expectations not being met. In this piece of code, we negated the conditionals, eliminated `else` branches, and hoisted the exceptions to the top of the function.
All failure cases are now front-and-center, which means you're less likely to ignore the case that matters the most: what happens when things go wrong?
Guard clauses also help understand the contract of a function at a glance, by reading the first few lines.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-54]
:::

::: .mde-inline.mde-66
Whenever you notice branches of code that could be isolated into a function call, extract them into a function.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-55]
:::

::: .mde-inline.mde-66
Here's a pattern I see all the time.
Suppose we find this piece of code, where we're creating a JSX node if we received a `label` prop, inside a larger function that's used to render a form row.
As somebody reads the function, they begin to wonder about whether this branch will result in a fork of the logic, depending on the input, or if the logic will stay the same regardless of the label contents.
At the same time, since this `optionalLabel` is a `let` binding, we're now also wondering if its value might be changing somewhere else.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-56]
:::

::: .mde-inline.mde-66
Instead of leaving readers wondering, we can extract the `optionalLabel`'s value into a function.
Now our binding can be `const`.
More importantly, we've shoved the label creation out of the flow of operations, leaving behind a function call that can be read as "oh, and here we render some label".
When we're reading the form row's render method, we wanted to understand how rows are rendered, but not necessarily how the labels it contains are rendered.
By extracting the logic into its own function, we are being explicit. It's clear that it doesn't depend on anything other than the `labelText` variable, and we might even find expanding the functionality without disrupting what used to be the parent function, or reusing the function elsewhere.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-57]
:::

::: .mde-inline.mde-66
Sometimes we're not worried about readability because of unrelated logic getting in the way, but because of the sheer length of a function.
When everything else fails, you can just chop your large function into smaller pieces.
Try to focus on the aspects of the larger functionality that are being attacked, and then create smaller functions that deal only with each of those aspects.
The parent function might end up as just a list of 5 or 6 function calls.
This is okay: we're outlining, at a very high level, what the function does, and developers can learn more by reading how those sub-routines are implemented.
:::

::: .mde-pad-15
:::

::: .mde-inline.mde-33
![][slide-58]
:::

::: .mde-inline.mde-66
Thanks!
:::


[slide-0]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.0-45fd43eced3f48d0946e660ddb130ccf.png
[slide-1]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.1-798760c3018b4e028949319628502f2e.png
[slide-2]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.2-fd49f69530184e3abefcd22193e0f527.png
[slide-3]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.3-eb0f45b363704832854006e749294645.png
[slide-4]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.4-a5f02846fa6d4d9fa6f36e7f0314cbeb.png
[slide-5]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.5-2894db81615142088a83101770a0098f.png
[slide-6]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.6-78d4192f7a23437e957f131db5cfe12b.png
[slide-7]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.7-4ff8d7cf453c46c5b6d45c2e3208ef40.png
[slide-8]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.8-1c81560bbb7d4ebd8bf35d4f21f04c1c.png
[slide-9]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.9-126f943e1b414fcab79ab579ecc1a006.png
[slide-10]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.10-b0addf081f7e468b96637ec0134f4eaa.png
[slide-11]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.11-9a443a67dbfc41839ed3b7e0b02eca12.png
[slide-12]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.12-add06f85174647c393327fd2a6e23d69.png
[slide-13]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.13-00056e80ec8548efbcfe24b9ec635792.png
[slide-14]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.14-4a18d972a1494681922ab0f74e7fddc5.png
[slide-15]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.15-b43608b359d34acd8c3e1f70031ab858.png
[slide-16]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.16-95289d0cebef4759822a464668ea6696.png
[slide-17]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.17-33cbb61a236c4ca689d4db062513f774.png
[slide-18]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.18-83c006876f8049ceaadb23768bba23cb.png
[slide-19]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.19-6ec2800544fc4fea996c2cb7e6592fe9.png
[slide-20]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.20-c08d063117a34b68ba02fbef8751ae71.png
[slide-21]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.21-9b0a04ad31c3473683af0c80571194e1.png
[slide-22]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.22-33bf0dd323164b3087b2ea31ac1c5ac1.png
[slide-23]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.23-ec1c1b7068f949818dd8cae72624c7b8.png
[slide-24]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.24-a4c548fcf7484d838ea4ddee1c4b9c56.png
[slide-25]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.25-4f365eb5fe99497fabdd441a9e11df8f.png
[slide-26]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.26-eca027f5f67947af9aab5f0f09a3fa87.png
[slide-27]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.27-14b3abf29ffe432f8f8c2469dd3cbef9.png
[slide-28]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.28-8c2c313387f84ab2a2063527ff851659.png
[slide-29]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.29-8531c09ebf8740dca757e2defd1a993a.png
[slide-30]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.30-6751b1cde4c14411b860c64e4b259f3a.png
[slide-31]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.31-79f576f3461d4e688d25987a2698f212.png
[slide-32]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.32-c8ac4b3388884174914d420a9b167be2.png
[slide-33]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.33-0440892d48fc4330a9158a00cd23a161.png
[slide-34]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.34-b248c21833af49049235037752e01512.png
[slide-35]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.35-ff657bfcf1504408aa7c9e3656b3cf23.png
[slide-36]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.36-76a0a8effdf049f1b3724150136a3139.png
[slide-37]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.37-0a6877ab33614319b9bd608f47678072.png
[slide-38]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.38-5d52959cf9bd41009cd29098817dbbc3.png
[slide-39]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.39-133ca760955144b1b74120be58f699d2.png
[slide-40]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.40-0a5f3e6c3cd447d8a8c2f643503fa88a.png
[slide-41]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.41-3f6fdc1c735043b2a331fa1eaad3ceea.png
[slide-42]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.42-97f0a85bb7df4528ae9b9a571a24a098.png
[slide-43]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.43-ae3003dc71794a72bbae7f784078daae.png
[slide-44]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.44-677c125041f546ed9486186802b98ca5.png
[slide-45]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.45-8d78cef333494bbf9ff39053a82434ff.png
[slide-46]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.46-1585edf218bc4c7aa03716192bfdbc6f.png
[slide-47]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.47-a401b40b4c18497e8db82c50a6aff8cd.png
[slide-48]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.48-ca6d85e124d341c7b4e6a581e2866a69.png
[slide-49]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.49-d434da64173c454cbdfe2aba362d2092.png
[slide-50]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.50-1ef438db942d493faa57dbb2ea3cc5e8.png
[slide-51]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.51-cbe03604fbc243a3bee21f1c3b8bab0a.png
[slide-52]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.52-d73dc347c9ad45ee89397f2a8c8e880c.png
[slide-53]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.53-545e7722c07c4a8e91407a51404bf062.png
[slide-54]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.54-a8950002b8fc4ffda4289e6acd28a4c1.png
[slide-55]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.55-f63c5d0533e94b37b55c86e6d863c177.png
[slide-56]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.56-fcda95e4ea5c41d59cb7ed12149810c9.png
[slide-57]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.57-f706b4c19b0f46ac97ba3dd06aa3925a.png
[slide-58]: https://s3.amazonaws.com/images.ponyfoo.com/uploads/keynote-modular-design.58-cb86e1ebede042a19d2fd40131fb7a56.jpg
[nzgb]: https://twitter.com/nzgb
[elastic]: https://elastic.co
