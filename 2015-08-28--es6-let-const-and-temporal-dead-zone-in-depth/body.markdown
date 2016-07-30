## Let Statement

The `let` statement is one of the most well-known features in ES6, which is partly why I grouped it together with a few other new features. It works like a `var` statement, but it has different scoping rules. JavaScript has always had a complicated ruleset when it came to scoping, driving many programmers crazy when they were first trying to figure out how variables work in JavaScript.

Eventually, you discover this thing called [hoisting][1], and things start making a bit more sense to you. Hoisting means that variables get pulled from anywhere they were declared in user code to the top of their scope. For example, see the code below.

```js
function areTheyAwesome (name) {
  if (name === 'nico') {
    var awesome = true
  }
  return awesome
}
areTheyAwesome('nico')
// <- true
areTheyAwesome('christian heilmann')
// <- undefined
```

The reason why this doesn't implode into oblivion is, as we know, that `var` is function-scoped. That coupled with hoisting means that what we're really expressing is something like the piece of code below.

```js
function areTheyAwesome (name) {
  var awesome
  if (name === 'nico') {
    awesome = true
  }
  return awesome
}
```

Whether we like it or not (or we're just used to it -- I know I am), this is plainly more confusing than having block-scoped variables would be. Block scoping works on the bracket level, rather than the function level.

Instead of having to declare a new `function` if we want a deeper scoping level, block scoping allows you to just leverage existing code branches like those in `if`, `for`, or `while` statements; you could also create new `{}` blocks arbitrarily. As you may or may not know, the JavaScript language allows us to create an indiscriminate number of blocks, just because we want to.

```js
{{{{{var insane = 'yes, you are'}}}}}
console.log(insane)
// <- 'yes, you are'
```

With `var`, though, one could still access the variable from outside those many, many, many blocks, and not get an error. Sometimes it can be very useful to get errors in these situations. Particularly if one or more of these is true.

- Accessing the inner variable breaks some sort of encapsulation principle in our code
- The inner variable doesn't belong in the outer scope at all
- The block in question has many siblings that would also want to use the same variable name
- One of the parent blocks already has a variable with the name we need, but it's still appropriate to use in the inner block

## So how does this `let` thing work?

> The `let` statement is an alternative to `var`. It follows block scoping rules instead of the default function scoping rules. This means you **don't need entire functions** to get a new scope -- _a simple `{}` block will do!_

```js
let outer = 'I am so eccentric!'
{
  let inner = 'I play with neighbors in my block and the sewers'
  {
    let innermost = 'I only play with neighbors in my block'
  }
  // accessing innermost here would throw
}
// accessing inner here would throw
// accessing innermost here would throw
```

Here is where things got interesting. As I wrote this example I thought _"well, but if we now declare a function inside a block and access it from outside that block, things will **surely go awry**"_. Based on my existing knowledge of ES5 I fully expected the following snippet of code to work, and it does in fact _work in ES5_ but it's [broken in ES6][2]. That would've been a problem because it'd make super easy to expose block-scoped properties through functions that become hoisted outside of the block. I didn't expect this to `throw`.

```js
{
  let _nested = 'secret'
  function nested () {
    return _nested
  }
}
console.log(nested())
// nested is not defined
```

As it turns out, this wasn't a bug in Babel, but in fact a _(much welcome)_ change in ES6 language semantics.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/nzgb">@nzgb</a> <a href="https://twitter.com/rauschma">@rauschma</a> <a href="https://twitter.com/sebmck">@sebmck</a> AFAIR, this is correct - ES6 finally specified functions in blocks to behave as block-scoped.</p>&mdash; Ingvar Stepanyan (@RReverser) <a href="https://twitter.com/RReverser/status/637349812485099520">August 28, 2015</a></blockquote>

Note that you can still expose nested `let` things to outer scopes simply by assigning them to a variable that has more access. I wouldn't recommend you do this however, as there probably are cleaner ways to write code in these situations -- such as **not using `let` when you don't want block scoping**.

```js
var nested
{
  let _nested = 'secret'
  nested = function () {
    return _nested
  }
}
console.log(nested())
// <- 'secret'
```

In conclusion, block scoping can be quite useful in new codebases. Some people will tell you to drop `var` forever and just use `let` everywhere. Some will tell you to never use `let` because that's not the _One True Way of JavaScript_. My position might change over time, but this is it -- for the time being:

> I plan on using `var` most of the time, and `let` in those situations where I would've otherwise hoisted a variable to the top of the scope for no reason, when they actually belonged inside a conditional or iterator code branch.

## The _Temporal Dead Zone_ and the Deathly Hallows

One last thing of note about `let` is a mystical concept called the ["Temporal Dead Zone" _(TDZ)_][3] _-- ooh... so scary, I know._

![enter image description here][4]

In so many words: if you have code such as the following, it'll throw. 

```js
there = 'far away'
// <- ReferenceError: there is not defined
let there = 'dragons'
```

If your code tries to access `there` in any way before the `let there` statement is reached, the program will throw. Declaring a method that references `there` before it's defined is okay, as long as the method doesn't get executed while `there` is in the TDZ, and `there` will be in the TDZ for as long as the `let there` statement isn't reached _(while the scope has been entered)_. This snippet won't throw because `return there` isn't executed until after `there` leaves the TDZ.

```js
function readThere () {
  return there
}
let there = 'dragons'
console.log(readThere())
// <- 'dragons'
```

But this snippet will, because access to `there` occurs _before leaving the TDZ for `there`_.

```js
function readThere () {
  return there
}
console.log(readThere())
// ReferenceError: there is not defined
let there = 'dragons'
```

Note that the semantics for these examples doesn't change when `there` isn't actually assigned a value when initially declared. The snippet below still throws, as it still tries to access `there` before leaving the TDZ.

```js
function readThere () {
  return there
}
console.log(readThere())
// ReferenceError: there is not defined
let there
```

This snippet still works because it still leaves the TDZ before accessing `there` in any way.

```js
function readThere () {
  return there
}
let there
console.log(readThere())
// <- undefined
```

The only tricky part is to remember that _(when it comes to the TDZ)_ functions work sort of like blackboxes until they're actually executed for the first time, so it's okay to place `there` inside functions that don't get executed until we leave the TDZ.

> The whole point of the TDZ is to make it easier to catch errors where accessing a variable before it's declared in user code leads to unexpected behavior. This happened a lot with ES5 due both to hoisting and poor coding conventions. In ES6 it's easier to avoid. Keep in mind that hoisting still applies for `let` as well -- this just means that the variables will be created when we enter the scope, and the TDZ will be born, but they will be inaccessible until code execution hits the place where the variable was actually declared, at which point we leave the TDZ and are cleared to use the variable.

## Const Statement

Phew. I wrote more than I ever wanted to write about `let`. Fortunately for both of us, `const` is quite similar to `let`.

- `const` is also _block-scoped_
- `const` also enjoys the marvels of _TDZ semantics_

There's also a couple of major differences.

- `const` variables must be declared using an initializer
- `const` variables can only be assigned to once, in said initializer
- `const` variables **don't** make the assigned value immutable
- Assigning to `const` will fail silently
- Redeclaration of a variable by the same name _will_ throw

Let's go to some examples. First, this snippet shows how it follows block-scoping rules just like `let`.

```js
const cool = 'ponyfoo'
{
  const cool = 'dragons'  
  console.log(cool)
  // <- 'dragons'
}
console.log(cool)
// <- 'ponyfoo'
```

Once a `const` is declared, you can't change the reference or literal that's assigned to it.

```js
const cool = { people: ['you', 'me', 'tesla', 'musk'] }
cool = {}
// <- "cool" is read-only
```

You can however, change the reference itself. It does not become immutable. You'd have to use [`Object.freeze`][4] to make the value itself immutable.

```js
const cool = { people: ['you', 'me', 'tesla', 'musk'] }
cool.people.push('berners-lee')
console.log(cool)
// <- { people: ['you', 'me', 'tesla', 'musk', 'berners-lee'] }
```

You can also make other references to the `const` that *can*, in fact, change.

```js
const cool = { people: ['you', 'me', 'tesla', 'musk'] }
var uncool = cool
uncool = { people: ['edison'] } // so uncool he's all alone
console.log(uncool)
// <- { people: ['edison'] }
```

I think `const` is great because it allows us to mark things that we really need to preserve as such. Imagine the following piece of code, which does come up in some situations _-- sorry about the extremely contrived example._

```js
function code (groceries) {
  return {eat}
  function eat () {
    if (groceries.length === 0) {
      throw new Error('All out. Please buy more groceries to feed the code.')
    }
    return groceries.shift()
  }
}
var groceries = ['carrot', 'lemon', 'potato', 'turducken']
var eater = code(groceries)
console.log(eater.eat())
// <- 'carrot'
```

I sometimes come across code where someone is trying to add more `groceries` to the list, and they figure that doing the following would _just work_. In many cases this does work. However, if we're passing a reference to groceries to something else, the re-assignment wouldn't be carried away to that other place, and hard to debug issues would ensue.

```js
// a few hundred lines of code later...
groceries = ['heart of palm', 'tomato', 'corned beef']
```

If `groceries` were a constant in the piece of code above, this re-assignment would've been far easier to detect. Yay, ES6! I can definitely see myself using `const` a lot in the future, but I haven't quite internalized it yet.

> I guess more coding is in order!

  [1]: /articles/javascript-variable-hoisting "JavaScript Variable Hoisting on Pony Foo"
  [2]: http://www.2ality.com/2015/02/es6-scoping.html "Variables and scoping in ECMAScript 6, see section 7"
  [3]: http://jsrocks.org/2015/01/temporal-dead-zone-tdz-demystified/ "Temporal Dead Zone (TDZ) Demystified"
  [4]: https://i.imgur.com/79mp6As.jpg
  [5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze "Object.freeze() – MDN"
