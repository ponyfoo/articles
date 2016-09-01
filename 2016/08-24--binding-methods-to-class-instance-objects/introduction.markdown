The problem is when we have a class method such as the following.

```js
class Logger {
  printName (name = 'there') {
    this.print(`Hello ${name}`);
  }

  print (text) {
    console.log(text); 
  }
}
```

Then -- for whatever reason -- the `printName` method's context changes, expectations aren't met, exceptions are thrown, and kittens die.

```js
const logger = new Logger();
const { printName } = logger;
printName();
// <- <mark>Uncaught TypeError: Cannot read property 'print' of undefined</mark>
```

Twitter drama began with André's salty remarks about a competing front-end framework.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">constructor() {<br>  this.onClick = this.onClick.bind(this);<br>}<br><br>Congrats, 3 this in 1 LOC, and it&#39;s not even app logic. Oh, it&#39;s official docs.</p>&mdash; André Staltz (@andrestaltz) <a href="https://twitter.com/andrestaltz/status/768087662557274112">August 23, 2016</a></blockquote>

The problem lies, of course, with the fact that JavaScript doesn't yet have semantics where we can easily mark every class method as bound to that  class, which is how they probably should work.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">🏁 The reality is classes are in dire need for improvement<br>2️⃣ Usability via sensible defaults ⤵️<br>☑️ Own scope<br>☑️ Methods bound to instance</p>&mdash; Nicolás Bevacqua (@nzgb) <a href="https://twitter.com/nzgb/status/768152975676174336">August 23, 2016</a></blockquote>

Let's look at our options.
