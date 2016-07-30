## [What do you mean][1]; classes in JavaScript?

JavaScript is a prototype-based language, so what are ES6 classes really? They're syntactic sugar on top of prototypical inheritance -- a device to make the language more inviting to programmers coming from other paradigms who might not be all that familiar with prototype chains. Many features in ES6 _(such as [destructuring][2])_ are, in fact, syntactic sugar -- and classes are no exception. I like to clarify this because it makes it much easier to understand the underlying technology behind ES6 classes. There is no huge restructuring of the language, they just made it easier for people used to classes to leverage prototypal inheritance.

> While I may dislike the term _"classes"_ for this particular feature, I have to say that the syntax is in fact much easier to work with than regular prototypal inheritance syntax in ES5, and that's a win for everyone -- regardless of them being called classes or not.

Now that that's out of the way, I'll assume you understand prototypal inheritance -- just because you're reading a blog about JavaScript. Here's how you would describe a `Car` that can be instantiated, fueled up, and move.

```js
function Car () {
  this.fuel = 0;
  this.distance = 0;
}

Car.prototype.move = function () {
  if (this.fuel < 1) {
    throw new RangeError('Fuel tank is depleted')
  }
  this.fuel--
  this.distance += 2
}

Car.prototype.addFuel = function () {
  if (this.fuel >= 60) {
    throw new RangeError('Fuel tank is full')
  }
  this.fuel++
}
```

To move the car, you could use the following piece of code.

```js
var car = new Car()
car.addFuel()
car.move()
car.move()
// <- RangeError: 'Fuel tank is depleted'
```

Neat. What about with ES6 classes? The syntax is very similar to declaring an object, except we precede it with `class Name`, where `Name` is the name for our class. Here we are leveraging the [method signature notation][3] we covered yesterday to declare the methods using a shorter syntax. The `constructor` is just like the constructor method in ES5, so you can use that to initialize any variables your instances may have.

```js
class Car {
  constructor () {
    this.fuel = 0
    this.distance = 0
  }
  move () {
    if (this.fuel < 1) {
      throw new RangeError('Fuel tank is depleted')
    }
    this.fuel--
    this.distance += 2
  }
  addFuel () {
    if (this.fuel >= 60) {
      throw new RangeError('Fuel tank is full')
    }
    this.fuel++
  }
}
```

In case you haven't noticed, and for some obscure reason that escapes me, **commas are invalid** in-between properties or methods in a class, as opposed to object literals where commas are _(still)_ mandatory. That discrepancy is bound to cause headaches to people trying to decide whether they want a plain object literal or a class instead, but the code *does* look sort of cleaner without the commas here.

Many times _"classes"_ have static methods. Think of your friend the `Array` for example. Arrays have instance methods like `.filter`, `.reduce`, and `.map`. The `Array` _"class"_ itself has static methods as well, like [`Array.isArray`][4]. In ES5 code, it's pretty easy to add these kind of methods to our `Car` _"class"_.

```js
function Car () {
  this.topSpeed = Math.random()
}
Car.isFaster = function (left, right) {
  return left.topSpeed > right.topSpeed
}
```

In ES6 `class` notation, we can use precede our method with `static`, following a similar syntax as that of `get` and `set`. Again, just sugar on top of ES5, as it's quite trivial to transpile this down into ES5 notation.

```js
class Car {
  constructor () {
    this.topSpeed = Math.random()
  }
  static isFaster (left, right) {
    return left.topSpeed > right.topSpeed
  }
}
```

One sweet aspect of ES6 `class` sugar is that you also get an `extends` keyword that enables you to easily _"inherit"_ from other _"classes"_. We all know Tesla cars move further while using the same amount of fuel, thus the code below shows how `Tesla extends Car` and "overrides" _(a concept you might be familiar with if you've ever [played around with C#][5])_ the `move` method to cover a larger distance.

```js
class Tesla extends Car {
  move () {
    super.move()
    this.distance += 4
  }
}
```

The special `super` keyword identifies the `Car` class we've inherited from -- and since we're speaking about C#, it's akin to [`base`][6]. It's _raison d'être_ is that most of the time we _override_ a method by re-implementing it in the inheriting class, -- `Tesla` in our example -- we're supposed to call the method on the base class as well. This way we don't have to copy logic over to the inheriting class whenever we re-implement a method. That'd be particularly lousy since whenever a base class changes we'd have to paste their logic into every inheriting class, turning our codebase into a maintainability nightmare.

If you now did the following, you'll notice the Tesla car moves two places because of `base.move()`, which is what every regular car does as well, and it moves an additional four places because `Tesla` is just that good.

```js
var car = new Tesla()
car.addFuel()
car.move()
console.log(car.distance)
// <- 6
```

The most common thing you'll have to override is the `constructor` method. Here you can just call `super()`, passing any arguments that the base class needs. Tesla cars are twice as fast, so we just call the base `Car` constructor with twice the advertised `speed`.

```js
class Car {
  constructor (speed) {
    this.speed = speed
  }
}
class Tesla extends Car {
  constructor (speed) {
    super(speed * 2)
  }
}
```

Tomorrow, we'll go over the syntax for `let`, `const`, and `for ... of ...`. Until then!

[1]: https://www.youtube.com/watch?v=fCEo2wfudqk "Peace Sells - Megadeth"
[2]: /articles/es6-destructuring-in-depth "ES6 JavaScript Destructuring in Depth on Pony Foo"
[3]: /articles/es6-object-literal-features-in-depth#method-signatures "ES6 Object Literal Features in Depth on Pony Foo"
[4]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray "Array.isArray() - MDN"
[5]: https://msdn.microsoft.com/en-us/library/aa645768(v=vs.71).aspx "Overriding methods in C# - MSDN"
[6]: https://msdn.microsoft.com/en-us/library/hfw7t1ce.aspx "The base keyword ok MSDN"
