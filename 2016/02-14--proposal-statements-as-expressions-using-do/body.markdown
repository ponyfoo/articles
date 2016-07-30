# Using `do`

The `do` expression proposal lets you write a block of statements to be evaluated. The "completion value" is returned as the result of a `do` expression. In the following example, there's just a `3 * 10` expression in our block, so that's our completion value, and `30` is returned.

```js
<mark>do {</mark> 3 * 10 <mark>}</mark> // just an expression
// <- 30
```

The following bit of code is equivalent to the two `.map` examples we saw earlier. In this case, we use a `do` expression, allowing us to use `if` and `else` instead of ternary or logical operators.
```js
[1, -1, -0.5].map(x => <mark>do { if (x > 0) {</mark> x * 10 <mark>} else {</mark> -x * 10 <mark>} }</mark>)
// <- [10, 10, 5]
```

Side effects in `do` expressions become easier to read, and we are able to declare variables. In the following example we're purposely missing a `return` statement, as `do` expressions already implicitly return, as we saw in the case of the `if` / `else` example. Naturally, `const` and `let` variables declared inside a `do` block are scoped to that block, while `var` variables are scoped to the containing function.

```js
var data = do {
  const data = pullSomeData()
  doSomethingElse() // sideEffect
  data
}
```

# Using `do` Today

It's easy, there's [a Babel plugin][2] we can use.

```shell
npm install --save-dev babel-plugin-syntax-do-expressions
```

Then add the following to your `.babelrc` file or the `babel` property in `package.json`.

```json
{
  "plugins": ["syntax-do-expressions"]
}
```

_That's it._

Whenever you run the Babel CLI, it'll understand `do` expressions.

# Conditionals in JSX

A while back I wrote about [_"the weird parts"_ of using JSX][1] -- the JavaScript syntax extension Facebook built to help you write templates for React apps. Back then, I mentioned how sometimes you have to write code like the following when you want to conditionally render a piece of markup.

```js
return (
  <nav>
    <Home />
    { loggedIn && <LogoutButton /> || <LoginButton /> }
  </nav>
);
```

With `do` expressions, you could get rid of the weird-looking and _oft-confusing_ logical operators. This makes the code easier to read and saves you from having to deal with falsy expressions like `0` or `''`.

```js
return (
  <nav>
    <Home />
    {
      <mark>do {</mark>
        <mark>if (loggedIn) {</mark>
          <LogoutButton />
        <mark>} else {</mark>
          <LoginButton />
        <mark>}</mark>
      <mark>}</mark>
    }
  </nav>
)
```

When there's larger pieces of markup it becomes more elegant to be able to use statements instead of expressions -- and `do` let's you `do` that. The `do` syntax makes it easier to read conditionals, allows you to use variables, and makes it clear when part of an expression is a side effect.

[1]: /articles/react-jsx-and-es6-the-weird-parts#using-conditionals-in-your-view-components "Using conditionals in your JSX view components"
[2]: https://github.com/babel/babel/tree/master/packages/babel-plugin-syntax-do-expressions "babel/packages/babel-plugin-syntax-do-expressions on GitHub"
