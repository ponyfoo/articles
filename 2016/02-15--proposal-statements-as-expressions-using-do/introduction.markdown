When JavaScript expressions are evaluated, they produce a single value.

```js
3 * 10
// <- 30
```

If we want to add a condition in an expression, we need to use ternary expressions or logical operators. The following example displays both alternatives, although the former is usually preferred.

```js
[1, -1, -0.5].map(x => <mark>x > 0 ? x * 10 : -x * 10</mark>)
// <- [10, 10, 5]
[1, -1, -0.5].map(x => <mark>x > 0 && x * 10 || -x * 10</mark>)
// <- [10, 10, 5]
```

It can be confusing to create side effects, but you can achieve that through use of commas.

```js
sideEffect(), 3 * 10
// <- 30
```

It's not possible to declare variables you might need for temporary storage _within an expression_. As such, you typically extract these variables into the enclosing scope. More often than not, that's better for readability anyways, so it doesn't have a hugely negative impact.
