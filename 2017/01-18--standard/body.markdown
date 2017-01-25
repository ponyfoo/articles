This revision of JavaScript was marketed as LiveScript when it started shipping with a beta release of Netscape Navigator 2.0, in September 1995. It was rebranded as JavaScript (trademarked by Sun, now owned by Oracle) when Navigator 2.0 beta 3 was released in December 1995. Soon after this release, Netscape introduced a server-side JavaScript implementation for scripting in Netscape Enterprise Server, and named it [LiveWire][lw]. JScript, Microsoft's reverse-engineered implementation of JavaScript, was bundled with IE3 in 1996. JScript was available for Internet Information Server (IIS) in the server-side.

The language started being standardized under the ECMAScript name (ES) into the ECMA-262 specification in 1996, under a technical commitee at ECMA known as TC39. Sun wouldn't transfer ownership of the JavaScript trademark to ECMA, and while Microsoft offered JScript, other member companies didn't want to use that name, so ECMAScript stuck.

Disputes by competing implementations, JavaScript by Netscape and JScript by Microsoft, dominated most of the TC39 standards commitee meetings at the time. Even so, the committee was already bearing fruit: backward compatibility was established as a golden rule, bringing about strict equality operators (`===` and `!==`) instead of breaking existing programs that relied on the loose equality comparision algorithm.

The first edition of ECMA-262 was released June, 1997. A year later, in June 1998, the specification was refined under the ISO/IEC 16262 international standard, after much scrutiny from national ISO bodies, and formalized as the second edition.

By December 1999 the third edition was published, standardizing  regular expressions, the `switch` statement, `do`/`while`, `try`/`catch` and `Object#hasOwnProperty`, among a few other changes. Most of these features were already available in the wild through Netscape's JavaScript runtime, SpiderMonkey.

Drafts for an ES4 specification were soon afterwards published by TC39. This early work on ES4 led to [JScript​.NET in mid-2000][jscript] and, eventually, to [ActionScript 3 for Flash in 2006][flash].

Conflicting opinions on how JavaScript was to move forward brought work on the specification to a standstill. This was a delicate time for web standards: Microsoft had all but monopolized the web and they had little interest in standards development.

As [AOL laid off 50 Netscape employees in 2003][layoff], the Mozilla Foundation was formed. With over 95% of web browsing market share now in the hands of Microsoft, TC39 was disbanded.

It took two years until Brendan, now at Mozilla, had ECMA resurrect work on TC39 by using Firefox’s growing market share as leverage to get Microsoft back in the fold. By mid 2005, TC39 started meeting regularly once again. As for ES4, there were plans for introducing a module system, classes, iterators, generators, destructuring, type annotations, proper tail calls, algebraic typing, and an assortment of other features. Due to how ambitious the project was, work on ES4 was repeatedly delayed.

By 2007 the commitee was split in two: ES3.1, which hailed a more incremental approach to ES3; and ES4, which was overdesigned and underspecified. It wouldn't be until August 2008 when [ES3.1 was agreed upon][brendanmail] as the way forward, but later rebranded as ES5. Although ES4 would be abandoned, many of its features eventually made its way into ES6 (which was dubbed Harmony at the time of this resolution), while some of them still remain under consideration. The ES3.1 update served as the foundation on top of which the ES4 specification could be laid upon in bits and pieces.

In December 2009, on the ten-year anniversary since the publication of ES3, the fifth edition of ECMAScript was published. This edition codified de facto extensions to the language specification that had become common among browser implementations, adding get and set accessors, reflection and introspection, functional improvements to the `Array` prototype, native support for JSON parsing, and strict mode.

A couple of years later, in June 2011, the specification was once again reviewed and edited to become the third edition of the international standard ISO/IEC 16262:2011, and formalized under ECMAScript 5.1.

It took TC39 another four years to formalize ECMAScript 6, in June 2015. Starting with ES6, revisions are also known by their release year: ES6 is also known as ES2015, ES7 is also referred to as ES2016, and so on. The sixth edition is the largest update to the language that made its way into publication, implementing many of the ES4 proposals that were deferred as part of the Harmony resolution. I'll be exploring ES6 in depth [throughout Practical ES6][ch1].

In parallel with the ES6 effort, in 2012 the WHATWG (a standards body interested in pushing the web forward) set out to document the differences between ES5.1 and browser implementations, in terms of compatibility and interoperability requirements. The taskforce standardized `String#substr`, which was previously unspecified; unified several methods for wrapping strings in HTML tags, which were inconsistent across browsers; and documented `Object.prototype` properties like `__proto__` and `__defineGetter__`, among [other improvements][jssnote]. This effort was condensed into a Web ECMAScript specification, which eventually made its way into Annex B in 2015. Annex B was also updated to be normative and required for web browsers, whereas it used to be merely informative before.

The sixth edition is a significant milestone in the history of JavaScript. Besides the dozens of new features, ES6 marks a key inflection point where ECMAScript would become a rolling standard.

# ECMAScript as a Rolling Standard

Having spent ten years without observing significant change to the language specification after ES3, and four years for ES6 to materialize, it was clear the TC39 process needed to improve. The revision process used to be deadline-driven. Any delay in arriving at consensus would cause long wait periods between revisions, which lead to feature creep, causing more delays. Minor revisions were delayed by large additions to the specification, and large additions faced pressure to finalize so that the revision would be pushed through avoiding further delays.

Since ES6 came out, TC39 has [streamlined its proposal revisioning process][streamlined-slides] and adjusted it to meet modern expectations: the need to iterate more often and consistently, and to democratize specification development. At this point, TC39 moved from an ancient Word-based flow to using Ecmarkup and GitHub Pull Requests, greatly increasing the number of [proposals being created][proposals] as well as external participation by non-members.

Firefox, Chrome, Edge, Safari and Node.js all offer [over 95% compliancy][compat] of the ES6 specification but we’ve been able to use the features as they came out in each of these browsers rather than having to wait until the flip of a switch when their implementation of ES6 was 100% finalized.

The new process involves [four different maturity stages][stages]. The more mature a proposal is, the more likely it is to eventually make it into the specification.

Any discussion, idea or proposal for a change or addition which has not been submitted as a formal proposal is considered to be an aspirational "strawman" proposal (stage 0) and has no acceptance requirements. At the time of this writing, there's over a dozen active [stage 0 proposals][stage0].

At stage 1 a proposal is formalized and expected to address cross-cutting concerns, interactions with other proposals, and implementation concerns. Proposals at this stage should identify a discrete problem and offer a concrete solution to the problem. A stage 1 proposal often includes a high level API description, illustrative usage examples and a discussion of internal semantics and algorithms. Stage 1 proposals are likely to change significantly as they make their way through the process.

Proposals in stage 2 offer an initial draft of the specification. At this point, it's reasonable to begin experimenting with actual implementations in runtimes. The implementation could come in the form of a polyfill, user code that mangles the runtime into adhering to the proposal; an engine implementation, natively providing support for the proposal; or compiled into something existing engines can execute, using build-time tools to transform source code.

Proposals in stage 3 are candidate recommendations. At this point, implementors have expressed interest in the proposal. In practice, proposals move to this level with at least one browser implementation, a high-fidelity polyfill or when supported by a build-time compiler like Babel. At this stage, a proposal is unlikely to change beyond fixes to issues identified in the wild.

In order for a proposal to attain stage 4 status, two independent implementations need to pass conformance tests. Proposals that make their way through to stage four will be included in the next revision of ECMAScript.

Starting with ES6, new releases of the specification are expected to be published every year from now on. To accomodate the yearly release schedule, versions will now be referred to by their publication year. Thus ES6 becomes ES2015, ES7 is ES2016, and so on. Colloquially, ES2015 hasn't taken and is still largely regarded as ES6. ES7 had been announced before the naming convention changed, thus it is only sometimes referred to as ES2016.

It may be expected that, going forward, ES2017 and beyond won't be referenced by the old naming convention anymore. The streamlined proposal process combined with the yearly cut into standardization translates into a more consistent publication process, and it also means specification revision numbers are becoming less important. The focus is now on [proposal stages][proposals], and we can expect references to specific revisions of the ECMAScript standard to become more uncommon.

# Browser Support and Complementary Tooling
d
A stage 3 candidate recommendation proposal is most likely to make it into the specification in the next cut, provided two independent implementations land in JavaScript engines. Effectively, stage 3 proposals are considered safe to use in real-world applications, be it through an experimental engine implementation, a polyfill, or using a compiler. Stage 2 and earlier proposals are also used in the wild by JavaScript developers, tightening the feedback loop between implementors and consumers.

Babel and similar compilers that take code as input and produce output native to the web platform (HTML, CSS or JavaScript) are often referred to as transpilers, which are considered to be a subset of compilers. When we want to leverage a proposal that's not widely implemented in JavaScript engines in our code, compilers like Babel can transform the portions of code using that new proposal into something that's more widely supported by existing JavaScript implementations.

This transformation can be done at build-time, so that consumers receive code that's well supported by their JavaScript runtime of choice. This mechanism improves the runtime support baseline, giving JavaScript developers the ability to take advantage of new language features and syntax sooner. It is also significantly benefitial to specification writers and implementors, as it allows them to collect feedback regarding viability, desirability, and possible bugs or corner cases.

A transpiler can take the ES6 source code we write and produce ES5 code that browsers can interpret more consistently. This is the most reliable way of running ES6 code in production today: using a build step to produce ES5 code that any modern browser can execute.

The same applies to ES7 and beyond. As new versions of the language specification are released every year, we can expect compilers to support ES2017 input, ES2018 input and beyond. Similarly, as browser support becomes better, we can also expect compilers to reduce complexity in favor of ES6 output, then ES7 output, and so on. In this sense, we can think of JavaScript-to-JavaScript transpilers as a moving window that takes code written using the latest available language semantics and produces the most modern code they can output without compromising browser support.

# The Future of JavaScript

The JavaScript language has evolved from its humble beginnings in 1995, to the formidable language it is today. While ES6 is a great step forward, it's not the finish line. Given we can expect new specification updates every year, it's important to learn how to stay up to date with the specification.

Having gone over the rolling standard specification development process, one of the best ways to keep up with the standard is by periodically visiting the [TC39 proposals repository][proposals]. Keep an eye on candidate recommendations *(stage 3)*, which are likely to make their way into the specification. Subscribe to weekly email newsletters such as [Pony Foo Weekly][pfw] or [JavaScript Weekly][jsw]. Read JavaScript blogs with a focus on ECMAScript development, like [Pony Foo][pfes] or *Axel Rauschmayer's* [2ality.com][ar].

*This is an excerpt extracted from the [first chapter in Practical ES6][ch1]. Special thanks to [Brendan Eich][be], for reviewing countless drafts of this piece --- as well as [Mathias Bynens][mb], [Jordan Harband][jh], [Alex Russell][aru] and [Brian Terlson][bt]. Thanks peeps! 🎉*

[ch1]: /books/practical-es6/chapters/1 "Read the first  chapter of Practical ES6 online"
[lw]: https://mjavascript.com/out/serverside "A booklet from 1998 explains the intricacies of Server-Side JavaScript with LiveWire"
[flash]: https://mjavascript.com/out/brendan-devchat "Listen to Brendan Eich in the JavaScript Jabber podcast, talking about the origin of JavaScript"
[layoff]: https://mjavascript.com/out/aol-netscape "Read a news report from July 2003"
[brendanmail]: https://mjavascript.com/out/harmony "Brendan Eich sent an email to the es-discuss mailing list in 2008 where he summarized the situation, almost ten years after ES3 had been released."
[streamlined-slides]: https://mjavascript.com/out/tc39-improvement "The September 2013 presentation which lead to the streamlined proposal revisioning process."
[proposals]: https://mjavascript.com/out/tc39-proposals "Check out all proposals being considered by TC39."
[compat]: https://mjavascript.com/out/es6-compat "A detailed ES6 compatibility report across browsers"
[stages]: https://mjavascript.com/out/tc39-process "TC39 proposal process documentation"
[stage0]: https://mjavascript.com/out/tc39-stage0 "Check out the proposals currently at stage 0"
[pfw]: /weekly "Subscribe to Pony Foo Weekly!"
[jsw]: https://mjavascript.com/out/jsw "Check out JavaScript Weekly"
[ar]: https://mjavascript.com/out/ar "2ality.com by Axel Rauschmayer"
[pfes]: https://mjavascript.com/out/pf "Articles tagged ECMAScript on Pony Foo"
[jscript]: https://mjavascript.com/out/jscript-net ""
[be]: https://twitter.com/BrendanEich "@BrendanEich on Twitter"
[bt]: https://twitter.com/bterlson "@bterlson on Twitter"
[aru]: https://twitter.com/slightlylate "@slightlylate on Twitter"
[jh]: https://twitter.com/ljharb "@ljharb on Twitter"
[jssnote]: http://mjavascript.com/out/javascript "Check out this article for the full set of changes made when merging the Web ECMAScript specification upstream."
[mb]: https://twitter.com/mathias "@mathias on Twitter"
