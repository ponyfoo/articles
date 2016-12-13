# Design Iteration

As usual, the design process started with a simple PR [Kim][kim] made that displayed bars which depicted how full an allocator was relative to its total capacity.

![Initial implementation.][first]

Then [Panos][pmoust] proposed using a color palette to illustrate breaks between different nodes.

> Nice! Is it possible to get like a `1px` divider per occupied capacity segment? Or an alternating color palette indicating different nodes and their size? i.e:
>
> ```
> 10.10.10.10   -  (y)  -  6  - 0  -   4 GB   -  [###|###|######|#########|####|####|    ]
> 10.10.10.11   -  (y)  -  4  - 0  -  10 GB   -  [###|###|######|#####|                  ]
> ```

That ended up in a redesign [Kim][kim] implemented, where he also made the full visualization width relative to the largest allocator's total capacity. This redesign also [helped][second] get a notion of how many nodes an allocator has, and roughly how big they are. [Kim][kim]'s PR got merged shortly after that, but [I had][nico] my own ideas as well.

I removed the border used to display how much total capacity an allocator had, instead choosing to rely on a striped repeating linear gradient to represent available capacity. I also added tooltips that mentioned how large the nodes were, and made each slice in the visualization a link to that node's view. The links turned out to be pretty useful, as they avoid going through an allocator's page when we want to visit a particular node -- especially when they're large and thus easy to identify.

![First draft of my changes.][third]

Then [Alex][alex] came in with one of my favorite suggestions: using color shades per node size variation.

> Could it make sense to have different shades of the same color per size? Instead of explaning, threw together this ugly example:
>
> <img src='https://i.imgur.com/DnMhWAR.png' alt='' />

This change made colors more meaningful communicators of how the data is shaped. Before, each node took a color from a repeating list with a few colors. After, as seen below, each node with same capacity is colored in different shades of one color, making colors more meaningful.

[![Color coding helps distinguish different capacity tiers by looking at the color of each slice.][shade-1]][shade-1]

We experimented with color shading. **A lot.** At first we used the original colors Kim had used. Then I came up with a brighter color scheme, but that was too bright. Then we used some of the brand's colors, and lastly I improved the shade [generator][gen] algorithm quite a bit.

::: .mde-inline.mde-33
[<img src='https://i.imgur.com/4HVYWRr.png' alt='' />][shade-2]
:::

::: .mde-inline.mde-33
[<img src='https://i.imgur.com/35EjlGL.png' alt='' />][shade-3]
:::

::: .mde-inline.mde-33
[<img src='https://i.imgur.com/gi50jBw.png' alt='' />][shade-4]
:::

The design process is still ongoing -- as you know, web applications are never really "done"!

We're experimenting with a small line at the bottom that describes whether the node we're illustrating is a Kibana instance or an Elasticsearch instance. This piece of work also normalizes colors so that all nodes of the same capacity use the same color -- across all visualizations. Colors changed a bit, *again*, because bright contiguous shades of red, yellow, and green could be mistaken for cluster state. We've also added a legend so that humans know at a glance what each color means.

![The next version, that's being currently worked on!][next]

Let's look at some code.

# Color Shade Generation in JavaScript

All of the colors and shades are produced by an [infinite generator sequence][gen]. You can see [a demo in action on CodePen][pen]. I'll explain the code below.

First off, we import the [`color-ops`][cops] package. This nice little utility let's us perform basic color operations like spin, darken, or desaturate.

```js
import ops from 'color-ops'
```

Next up we have a tiny utility that can be paired with `Array#reduce` to apply several functions in an array to transform a value. An usage example for `waterfall` could look like `[v => v * 2, v => v + 3].reduce(waterfall, 5)`, which would result in `5 * 2`, and then `10 + 3`, returning `13`.

```js
const waterfall = (currentColor, nextCommand) => nextCommand(currentColor)
```

Next up we have a couple of utility functions to transform `rgb` components into an `[r, g, b, a]` array, which is the data structure `color-ops` relies on. The `rgbString` function takes the output from `color-ops` and turns it into a solid CSS color.

```js
const rgb = (r, g, b, a = 1) => [r, g, b, a]
const rgbString = ([r, g, b]) => `rgb(${r}, ${g}, ${b})`

const unknownColor = rgb(123, 123, 123)

const baseColors = new Map([
  [1024 * 64, rgb(40, 114, 150)], // turquoise
  [1024 * 32, rgb(213, 189, 62)], // yellow
  [1024 * 16, rgb(50, 167, 194)], // cyan
  [1024 * 8, rgb(37, 146, 98)], // green
  [1024 * 4, rgb(61, 64, 120)], // blue
  [1024 * 2, rgb(121, 88, 67)], // brown
  [1024, rgb(138, 66, 138)], // purple
])
```

We're finally getting to the generator code! Our generator is called `capacityShader` and it produces an infinite sequence, so we need to be careful about how we read values off of it, so that we're not stuck in an infinite loop. The `shadeIndex` tracks what shade of the base color we should `yield`.

```js
function* capacityShader(baseColor) {
  let shadeIndex = 0

  while (true) {
    // the infinite sequence
  }
}
```

As you probably know, the `%` remainder operator returns the remainder of the division. We use this to cycle in the `0..baseColors.length` range regardless of how large the `currentColorIndex` becomes, without having to worry about doing the cycling math by hand.

```js
const baseColorIndex = currentColorIndex % baseColors.length
const baseColor = baseColors[baseColorIndex]
```

In the loop, we calculate a `light` index and an `offset`, which are used to generate slightly different variations of each color. The `light` variable sits in the `0..16` range, _in `4` unit steps:_ `0`, `4`, .., `16`. The `offset` value sits in the `0..2` range. Note how for indices of `0`, our modifiers are also `0`. This means any sequence begins using the base color itself. The combination of both offsets means there's 15 possible shades of each color: `light` as `0..4 * 4` and `offset` as `0..2`, so `5 * 3`.

```js
const light = currentCapacityIndex * 4 % 20
const offset = currentCapacityIndex % 3
```

The `light` value will be used to determine what shade of the color we want, while the `offset` is also used to toggle saturation so that shades of the same color are less uniformly distributed, and thus easier to tell apart from each other. We use the `Array#reduce` waterfall to apply the color operations on the base color, and then round each component of the `[r, g, b, a]` result.

```js
const tasks = [
  currentColor => ops.lighten(currentColor, light + offset * 4),
  currentColor => ops.saturate(currentColor, offset * 8),
  currentColor => currentColor.map(Math.round)
]

const shade = tasks.reduce(waterfall, baseColor)
```

At long last, we `yield` the CSS RGB string version of the resulting color shade, suspending execution in the generator.

```js
yield rgbString(shade)
```

Then we just add one to the counter, and return to the top of the `while (true)` loop.

```js
shadeIndex++
```

In order to consume the generator, we instantiate it and read the sequence by hand.

```js
const shader = createCapacityShader(baseColor)
const color = shader.next().value
```

While we want a new shade of the same color, we keep calling `g.next()`. When we want to switch to a different color, we'll need to create a new shader object. The screenshot below shows the generator in action, rendering each shade 15 times before moving on to a new generator. As you can see, the generators are deterministic in that the same shades are generated every 15 steps. Saturation offsets make it easy to notice the difference between consecutive shades of the same color. Compare the screenshot below to [the previous screenshot][shade-3], where saturation offsets hadn't been introduced yet.

[![A CodePen demo with all the colors produced by the generator code.][colors]][pen]

The other cool piece of code was something I dubbed the `<NonBlockingRenderLoop>` component.

# Rendering Non-Critical UI Components Last

There's thousands of nodes in this page, which translates into thousands of DOM elements for the visualizations. As we were iterating on this page, we noticed a *significantly noticeable* performance drop that resulted in the entire page being frozen for several seconds at a time, as shown in the following performance profile taken during page load, as DOM nodes were being mounted.

![All work happens upfront, and every DOM node is mounted at the same time.][profile-before]

My solution was to defer all visualizations by wrapping them in a magical `NonBlockingRenderLoop` component:

```html
<NonBlockingRenderLoop>
  <InstanceCapacitiesViz { ...props } />
</NonBlockingRenderLoop>
```

Afterwards, the UI stopped freezing for seconds at a time. Instead, non-critical work was deferred until after the rest of the UI loaded, and then it was only done progressively, not constantly. At the same time, the visualizations load fast enough that waiting for them to load doesn't become an issue.

![After deferring non-critical UI components to an asynchronous yet sequential rendering loop.][profile-after]

So how does the `<NonBlockingRenderLoop>` component work? The idea is fairly simple: any component controls when and how its children are rendered. If our `render` method returns `null`, then nothing would be rendered, but if we returned `this.props.children` then we'd render our child components. We could _-- for example --_ implement an element like the following, which would block rendering of any children for one second.

```js
class NonBlockingRenderLoop extends Component {
  state = {
    wait: true
  }
  componentDidMount() {
    setTimeout(() => this.setState({ wait: false }), 1000)
  }
  render() {
    return this.state.wait ? null : this.props.children
  }
}
```

While this would be good enough to load non-critical components after critical ones, they'd still load all at the same time, which wouldn't be very much of an improvement. Instead, I came to a solution that's a bit similar, except that it relies on `requestAnimationFrame` and a lock. The way it works is:

1. All components start off blocking their children
1. All components loop through `requestAnimationFrame`, checking on a lock
1. One component obtains the lock
1. Inside the `requestAnimationFrame` callback, the component updates its `state` turning off the `wait` flag
1. That component releases the lock
1. Back to step 3
1. Repeat until all components were rendered

In order for the lock to work across different component instances, it needs to be shared somehow. We could just add the locking functionality to our module's top level, and then export the `NonBlockingRenderLoop` class, but that would mean there could only ever be a single category of non-critical components. Instead, what we could do is export a factory function that returns different component classes. This way, when we need different groups of components to load in separate asynchronous sequences rather than being blocked on each other, we could just wrap each component in a `NonBlockingRenderLoop`.

Consider the following pure component, where we render an item list wrapped in a `<NonBlockingRenderLoop>` component. We're creating a `NonBlockingRenderLoop` class just for the list being rendered at `renderItemList`. If the list is re-rendered, `createNonBlockingRenderLoop` will hit the cache and avoid creating a different component class, which would've resulted in remounting the entire tree, something we definitely want to avoid. A different portion of the application could just create a different NBRL, making any elements under that other component also load in an asynchronous sequence, but it'd be a different asynchronous sequence.

```js
const renderItemList = ({ id, items }) => {
  const NonBlockingRenderLoop = createNonBlockingRenderLoop({
    key: `list-items-for-${ id }`
  })
  return items.map(item =>
    <NonBlockingRenderLoop key={ item.id }>
      <ListItem item={ item } />
    </NonBlockingRenderLoop>
  )
}
```

Going then to the implementation, let's start with the skeleton. Here we're just exporting the factory function, and returning a `NonBlockingRenderLoop` that may or may not exist in the cache. We'll use a `concurrencyLevel` instead of a boolean lock, so that the component could be used to create asynchronous render loops that are able to render more than a single elements at a time.

```js
import Component from 'react'

const cache = new Map()

export function createNonBlockingRenderLoop({ key, concurrencyLevel = 1 }) {
  if (cache.has(key)) {
    return cache.get(key)
  }
  // ...
  cache.set(key, NonBlockingRenderLoop)
  return NonBlockingRenderLoop
}
```

Next up, we'll define a few functions that manage the lock slots. We start at `concurrencyLevel` slots, which are taken by each component being rendered, and then released. We will consider the lock to be busy if there are no slots left.

```js
let slots = concurrencyLevel
const isBusy = () => slots < 1
const takeSlot = () => slots--
const releaseSlot = () => slots++
```

Next up is the `NonBlockingRenderLoop` component. As we've done earlier, it starts on a `wait` state of `true` and only renders its children when `wait` is `false`. Instead of using a timeout, though, it calls a method named `enqueue` when it's mounted. The `enqueue` method checks whether the lock is busy, in which case it uses `requestAnimationFrame` to trigger another call to `enqueue`. This loop goes on until a slot is released. When the lock is not busy, which is the starting state, we'll take a slot from the lock and defer with `requestAnimationFrame`. Once the callback is executed we call `setState` to turn `wait` off, after which we release our lock on one of the slots. This process repeats itself until there's no more work left.

```js
class NonBlockingRenderLoop extends Component {
  state = {
    wait: true
  }

  componentDidMount() {
    this.enqueue()
  }

  componentWillUnmount() {
    this.stop()
  }

  render() {
    return this.state.wait ? null : this.props.children
  }

  enqueue() {
    if (isBusy()) {
      this.defer(() => this.enqueue())
    } else {
      takeSlot()
      this.defer(() => this.dequeue())
    }
  }

  dequeue() {
    this.setState({ wait: false })
    releaseSlot()
  }

  defer(fn) {
    requestAnimationFrame(() => {
      if (this.defer !== noop) {
        fn()
      }
    })
  }

  stop() {
    this.defer = noop
    if (this.state.wait) {
      releaseSlot()
    }
  }
}

function noop () {}
```

Lastly, we have a sanity check at `componentWillUnmount`, where we call the `stop` method. This sets `defer` to `noop` and releases our lock on a slot if the component is still waiting to render. As an additional sanity check, we avoid executing the callback in the `defer` method if the component has been unmounted since `requestAnimationFrame` was called.

_You can find the [full source code on gist.github.com][code-nonblock]._

In the not-so-distant-future React's next-generation renderer -- Fiber -- will help with these kinds of scenarios by allowing us to determine the rendering priority of different portions of our applications. Improving performance in these kinds of React development scenarios is about to become way easier!

Did React Fiber pique your interest? Here you go. Some of these are long reads _--- or presentations ---_, but they're well worth it. Take your time to dig through these links.

- [React Components, Elements, and Instances](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html)
- [Reconciliation in React](https://facebook.github.io/react/docs/reconciliation.html)
- [Basic React Theoretical Concepts](https://github.com/reactjs/react-basic)
- [React Design Principles](https://facebook.github.io/react/contributing/design-principles.html)
- [Fiber Principles](https://github.com/facebook/react/issues/7942)
- [Future and experimental React APIs](https://github.com/reactjs/react-future)
- [Evolving Complex Systems Incrementally](https://www.youtube.com/watch?v=d0pOgY8__JM) **<mark>Talk</mark>**
- [Don't Rewrite, React!](https://www.youtube.com/watch?v=BF58ZJ1ZQxY) **<mark>Talk</mark>**
- ["Is Fiber Ready Yet?"](http://isfiberreadyyet.com/)
- [React Fiber Renderer Code](https://github.com/facebook/react/tree/master/src/renderers/shared/fiber)

By the way, if you got this far, you may be interested in the [remote senior JavaScript engineer position][job] at Elastic! We're always looking for smart and talented individuals like yourself.

Ta! 🎉

[kim]: https://github.com/kimjoar "@kimjoar on Twitter"
[pmoust]: https://github.com/pmoust "@pmoust on GitHub"
[first]: https://i.imgur.com/ADGi8Jf.png
[second]: https://i.imgur.com/LLG5gaC.png "After implementing Panos' idea of color-coding each different node"
[nico]: https://twitter.com/nzgb "@nzgb on Twitter"
[third]: https://i.imgur.com/2tya0uL.png
[alex]: https://github.com/alexbrasetvik "@alexbrasetvik on GitHub"
[shade-1]: https://i.imgur.com/uKX3bug.png "Expand image to full size"
[shade-2]: https://i.imgur.com/4HVYWRr.png "Expand image to full size"
[shade-3]: https://i.imgur.com/35EjlGL.png "Expand image to full size"
[shade-4]: https://i.imgur.com/gi50jBw.png "Expand image to full size"
[colors]: https://i.imgur.com/Bryi7ZG.png
[pen]: https://codepen.io/bevacqua/pen/YpabJB
[cops]: https://github.com/tmcw/color-ops "tmcw/color-ops on GitHub"
[gen]: /articles/es6-generators-in-depth "ES6 Generators in Depth"
[next]: https://i.imgur.com/ZpyYMV1.png
[profile-before]: https://i.imgur.com/NvFEsAm.png
[profile-after]: https://i.imgur.com/NqJeetF.png
[code-nonblock]: https://gist.github.com/bevacqua/b7c481cc0bfb56320ac96c7f8c7cb0b6 "Full code for NonBlockingRenderLoop.js on gist.github.com"
[job]: https://www.jsco.re/kcih "Senior JavaScript Engineer, Cloud"
