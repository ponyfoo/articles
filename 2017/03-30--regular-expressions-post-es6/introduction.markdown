# Sticky Matching Flag `/y`

The sticky matching `y` flag introduced in ES6 is similar to the global `g` flag. Like global regular expressions, sticky ones are typically used to match several times until the input string is exhausted. Sticky regular expressions move `lastIndex` to the position after the last match, just like global regular expressions. The only difference is that a sticky regular expression must start matching where the previous match left off, unlike global regular expressions that move onto the rest of the input string when the regular expression goes unmatched at any given position.

The following example illustrates the difference between the two. Given an input string like `'haha haha haha'` and the `/ha/` regular expression, the global flag will match every occurrence of `'ha'`, while the sticky flag will only match the first two, since the third occurrence doesn't match starting at index `4`, but rather at index `5`.

```js
function matcher(regex, input) {
  return () => {
    const match = regex.exec(input)
    const lastIndex = regex.lastIndex
    return { lastIndex, match }
  }
}
const input = 'haha haha haha'
const nextGlobal = matcher(/ha/g, input)
console.log(nextGlobal()) // <- { lastIndex: 2, match: ['ha'] }
console.log(nextGlobal()) // <- { lastIndex: 4, match: ['ha'] }
console.log(nextGlobal()) // <- { lastIndex: 7, match: ['ha'] }
const nextSticky = matcher(/ha/y, input)
console.log(nextSticky()) // <- { lastIndex: 2, match: ['ha'] }
console.log(nextSticky()) // <- { lastIndex: 4, match: ['ha'] }
console.log(nextSticky()) // <- { lastIndex: 0, match: null }
```

We can verify that the sticky matcher would work if we forcefully moved `lastIndex` with the next piece of code.

```js
const rsticky = /ha/y
const nextSticky = matcher(rsticky, input)
console.log(nextSticky()) // <- { lastIndex: 2, match: ['ha'] }
console.log(nextSticky()) // <- { lastIndex: 4, match: ['ha'] }
rsticky.lastIndex = 5
console.log(nextSticky()) // <- { lastIndex: 7, match: ['ha'] }
```

Sticky matching was added to JavaScript as a way of improving the performance of lexical analyzers in compilers, which heavily rely on regular expressions.
