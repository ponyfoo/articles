# What's TC39?

TC39 means Technical Committee number 39. It's part of ECMA, the institution which standardizes the JavaScript language under the "ECMAScript" specification.

The ECMAScript specification defines how JavaScript works on a discrete step-by-step basis. Among other things, the specification explains:

- how the string `'A'` is `NaN`
- how the string `'A'` _is not equal_ to `NaN`
- how `NaN is `NaN but _is not equal_ to `NaN`
- and why introducing `Number.isNaN` was obviously a very good idea…

```js
isNaN(NaN) // true
isNaN('A') // true
'A' == NaN // false
'A' === NaN // false
NaN === NaN // false

// … solution!

Number.isNaN('A') // false
Number.isNaN(NaN) // true
```

It explains details about *when* positive zero is equal to negative zero _-- and when it isn't..._

```js
+0 == -0 // true
+0 === -0 // true
1/+0 === 1 / -0 // false
```

And they also make other jewels possible, such as encoding any valid JavaScript expression using only exclamation points, parenthesis, square brackets, and plus signs. Check out [JSFuck][jsf] to learn more about how to write any JavaScript using `+!()[]`.

*But, in all seriousness, the thankless work done by TC39 is **invaluable**.*

[jsf]: http://jsfuck.com
