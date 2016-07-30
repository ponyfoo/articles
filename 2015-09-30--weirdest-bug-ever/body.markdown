## Using an `<img>` tag

Specifically, in the header of the site. I saved the file as `ponyfoo.svg` and placed it in my images directory. Then I updated my HTML markup with something like the code below. The fallback `onerror` was a nice touch, I guess.

```html
<a href='/'>
  <img src='/img/ponyfoo.svg' <mark>onerror='this.src="/img/ponyfoo.png"'</mark> />
</a>
```

The problem with using an `<img>` tag to embed SVG graphics in a site is that it completely ignores CSS. That approach was immediately out then, as I couldn't animate the logo on hover -- or at all -- with an `<img>` tag. After that approach failed me, I tried an `<object>` next.

## Using an `<object>` tag

I know, what year is this? Anyways. Here's the HTML code I used.

```html
<a href='/'>
  <object data='/img/ponyfoo.svg' type='image/svg+xml'>
    <img src='/img/ponyfoo.png' />
  </object>
</a>
```

This one was even weirder. The link **stopped working altogether**, because the `<object>` goes into obscure sandboxing mode. The CSS animation wouldn't work either, but I could [place the CSS styles _inside_][2] the SVG graphic, and then they worked on hover. Still, the link wasn't working either. I guess I could [add the link inside the SVG][1] as well! At this point the SVG was an utter mess.

```html
<svg>
  <style><!-- lots of compiled css --></style>
  <!-- all the rects -->
  <a xmlns='http://www.w3.org/2000/svg' xlink:href='/' xmlns:xlink='http://www.w3.org/1999/xlink' target='_top'>
    <rect x='0' y='0' width='100%' height='100%' fill-opacity='0' />
  </a>
</svg>
```

That worked -- kind of. It wasn't the prettiest of files, though. I hated the fact that I had to inline the entire CSS inside the SVG, and even the link. So un-DRY of you, SVG.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Ugh! Could SVG have any more interop issues w/ html?&#10;- &lt;img&gt; ignores css/animations&#10;- &lt;object&gt; ignores links, needs embedded css</p>&mdash; Nicolas Bevacqua (@nzgb) <a href="https://twitter.com/nzgb/status/647325093891321856">September 25, 2015</a></blockquote>

Okay then, I decided to try one more and that's when it all went to hell.

## Using an `<svg>` tag

I decided that maybe using an inline `<svg>` tag wouldn't be that bad. After all, this was the logo. It's a relatively small graphic, and then I could have the CSS lying around with all its other CSS rule pals. This time, the markup went like below. I included an `<image>` tag as a [clever hack][3] that provided a fallback for older browsers.

```html
<a href='/'>
  <svg>
    <!-- all the rects -->
    <mark><image src='/img/ponyfoo.png'/></mark>
  </svg>
</a>
```

This one was supposed to work great. You can find the full snippet of code [here][5]. If you paste that code into an HTML file and open it, it'll work in most browsers. If you host that same file in a Node.js server, and navigate to that page under _Google Chrome 45.0.2454_, the animation won't work.

Let's recap to highlight how fringe all this is.

- It doesn't matter whether you set `<!doctype>` or not, that's inferred
- The [snippet][5] works on all browsers when opened as a file
- The animation doesn't work on _Google Chrome 45.0.2454_ when hosted on a server [(example in CodePen)][6]
- The animation works if you remove the `<a>` around the SVG
- The animation works if the snippet is opened as a file
- The animation works if **you aren't logged into your Google Chrome profile**
- The animation works if you use Chrome Canary _(we didn't try older versions)_
- It gets Google engineers to go _"holy shit!"_

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/nzgb">@nzgb</a> holy shit, it&#39;s working on one profile, but failing on another - same version of Chrome</p>&mdash; Jake Archibald (@jaffathecake) <a href="https://twitter.com/jaffathecake/status/647458048991227904">September 25, 2015</a></blockquote>

For the record, I'm _not bashing Google Chrome_ here. I'm just pointing at a very weird bug I came across and got pretty frustrated about. I guess this is one of those things Chris Heilmann talks about when he speaks of how we're failing the web.

Granted, we [may not][7] need a [moratorium][8] on web features -- but we do need this kind of primitives to be fixed. Basic stuff like styling a `<select>` or an `<input type='check'>` **shouldn't** be an art form (as Chris points out). It **should** be really simple to embed SVG graphics in our websites. It should be easier to set up SSL. 

We can do better.

[1]: http://stackoverflow.com/a/19553517/389745 "Make an html svg object also a clickable link question on StackOverflow"
[2]: https://css-tricks.com/using-svg/ "Using SVG as an <object> section, 'Using SVG on CSS-Tricks'"
[3]: https://twitter.com/jaffathecake/status/647328352664190976
[4]: http://codepen.io/bevacqua/pen/avpKBG?editors=110
[5]: https://gist.githubusercontent.com/bevacqua/07bf49036dcc534130b0/raw/e99b95fbf99bf5d70c6719090f8d2f2e5058c961/a.html
[6]: http://codepen.io/bevacqua/pen/QjdxgG?editors=100
[7]: /articles/fast-forwarding-the-web-platform "Fast-forwarding the Web Platform on Pony Foo"
[8]: http://www.quirksmode.org/blog/archives/2015/07/stop_pushing_th.html "Stop Pushing the Web Forward"
[9]: https://www.youtube.com/watch?v=RVEntHvof2w "Advancing JavaScript without breaking the web"
