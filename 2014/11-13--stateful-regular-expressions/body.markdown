```html
<button id='bad'>Wait, WHAT?</button>
```

```js
stateful = /button/ig;
bad.onclick = function (e) {
  alert(stateful.test(e.target.tagName));
};
```

The button alternates between alerting `true` and `false` in Google Chrome `38.0.2125.122`, and also on Firefox `31`.

As it turns out, this is the _"expected"_ behavior. I did not expect that. Regular expressions with the `g` modifier on them will keep track of the `lastIndex` where a match occurred, even across calls to `.test`.

<iframe src='https://codepen.io/bevacqua/fullembedgrid/xbxqQQ/?type=embed&safe=true' width='100%' height='300'></iframe>

You can check out the [live example on CodePen][1], too.

[1]: https://codepen.io/bevacqua/full/xbxqQQ/ "What is going on?"
