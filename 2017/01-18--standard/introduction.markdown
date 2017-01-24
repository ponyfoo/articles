*This is an excerpt extracted from the [first chapter in Practical ES6][ch1]. Special thanks to [Brendan Eich][be], for reviewing countless drafts of this piece --- as well as [Mathias Bynens][mb], [Jordan Harband][jh], [Alex Russell][aru] and [Brian Terlson][bt]. Thanks peeps! 🎉*

_<sub>Disclaimer: this article is not about the ["actual" "JavaScript" standard][jss]!</sub>_

# A Brief History of JavaScript Standards

Back in 1995, Netscape envisioned a dynamic web beyond what HTML could offer. Brendan Eich was initially brought into Netscape to develop a language that was functionally akin to Scheme, but for the browser. Once he joined, he learned that upper management wanted it to look like Java and a deal to that effect was already underway.

Brendan created the first JavaScript prototype in ten days, taking Scheme's first-class functions and Self's prototypes as its main ingredients. The initial version of JavaScript was code-named Mocha. It didn't have array or object literals, and every error resulted in an alert. The lack of exception handling is why, to this day, many operations result in `NaN` or `undefined`. Brendan's work on DOM level 0 and the first edition of JavaScript set the stage for standards work.

[ch1]: /books/practical-es6/chapters/1 "Read the first chapter of Practical ES6 online"
[be]: https://twitter.com/BrendanEich "@BrendanEich on Twitter"
[bt]: https://twitter.com/bterlson "@bterlson on Twitter"
[aru]: https://twitter.com/slightlylate "@slightlylate on Twitter"
[jh]: https://twitter.com/ljharb "@ljharb on Twitter"
[jss]: https://blog.whatwg.org/javascript "Read about the JavaScript standard here."
[mb]: https://twitter.com/mathias "@mathias on Twitter"
