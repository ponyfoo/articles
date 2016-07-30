I've stopped using jQuery [_years ago_][1] now. I learned a lot from jQuery, and from [not using it][1] anymore as well. Besides the things you can do with or without jQuery in terms of DOM selection, manipulation, and traversal _(or AJAX, the sole reason people keep adding it to their projects nowadays -- even though modules like [`xhr`][2] exist)_ -- there's unsurprisingly a ton of stuff you can easily do in plain native DOM API, without involving any libraries.

Some of them even enjoy wide browser support! This article explores `getBoundingClientRect`, `elementFromPoint`, and text selection with `selectionStart` and `selectionEnd`.

[1]: /articles/getting-over-jquery
[2]: https://github.com/Raynos/xhr
