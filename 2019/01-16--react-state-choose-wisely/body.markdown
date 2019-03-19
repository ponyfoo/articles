Here’s a non-exhaustive list of things you should consider:

 1. How quickly can you add new state code?
 2. How well encapsulated is your stateful data?
 3. How easy is it to pull the data into a new component?

Let’s use this list as a basis for comparing different ways of handling state. To do this, we’ll make the same simple component using three different forms of state handling. By making the same thing three ways, you’ll get a few nice perspectives.

First, you’ll see that any approach can solve any problem. This is important because if you know that you can solve nearly any problem with any form of state management, you can turn your attention from the technical problem of getting it working, to the more abstract problem of making it work well in your over all application.

Second, by making the same thing three ways, you’ll start to see the subtle differences which can become big headaches as an applications grows.

What are the three ways to manage state?

1.  **Local State**: State stored directly on the component.
2.  **Global State**: A global store. In this example, you’ll use redux.
3.  **Context**: A newer API that lets you access state higher up the tree.

Using the three points of comparison and the three ways of managing state, you’ll be able to build a nice table that will let you compare the different options.

|             | Local | Global | Context |
|-------------|-------|--------|---------|
| Easy         | `?`   | `?`    | `?` |
| Encapsulated | `?`   | `?`    | `?` |
| Available    | `?`   | `?`    | `?` |

Alright. Let’s get started.

You’ll be making a very simple counter. It’s just two things: the current count and a button to add to the current count. As mentioned you’ll have one for each type of state, but they all act exactly the same way.

![][initial-state]

So far, nothing too exciting. The first thing you are going to do is make the presentation layer. This will be a simple reusable component that displays the count and the button. Making this independent is not only a good practice, it will also a show you how easy it is to swap different forms of state handling.

```javascript
import React from 'react';

export default function Display({ addOne, count, title}) {
  return(
    <div className="counter">
      <h3>{title}</h3>
      Current Count: {count}
      <button onClick={addOne}> Add One! </button>
    </div>
  )
}
```

Now that you have your basic display component, you can start to build the stateful components.

### Local State

The first component you’ll build will store state locally, directly on the component.

All you need is a basic class with `state` as a property and a method to add to state. Pass both the `count` from the state object and the `addOne()` method into your presentation component and you’re all done.

```javascript
import React, { Component } from 'react';
import Display from '../Display/Display';

export default class StateCounter extends Component {
  state = {
    count: 0
  }

  addOne = () => {
    this.setState(state => ({count: state.count + 1}))
  }

  render() {
    return (
      <Display
        addOne={this.addOne}
        count={this.state.count}
        title="Local Counter"
      />
    )
  }
}
```

At this point, one thing should jump out: it’s easy. Even without knowing much about other forms of state handling, you can see this one is very simple to set up. But it gets even better.

With the introduction of [React Hooks](https://medium.com/r/?url=https%3A%2F%2Freactjs.org%2Fdocs%2Fhooks-overview.html), you’ll remove nearly all the boilerplate code.

Here’s an updated version using React hooks that’s nearly identical to the official docs:

```javascript
import React, { useState } from 'react';
import Display from '../Display/Display';

export default function StateCounterHooks() {
  const [count, add] = useState(0);
  const addOne = () => add(count + 1);
  return (
    <Display
      addOne={addOne}
      count={count}
      title="Local Counter"
    />
  )
}
```

As above, `count` is the stateful data, and `add()` is a method for updating state. It would really be difficult to make this easier.

### Global State

The next component you’ll build will use a global store. In this example, you’ll use redux since it’s by far the most popular, but many of the issues would be the same if you were using other options like MobX or even Apollo and GraphQL.

We’ll move quickly past some of the complexities of implementing redux. If you need a refresher, check out the [official docs](https://medium.com/r/?url=https%3A%2F%2Fredux.js.org%2F).

To start make an action to add an item to the global store.

```javascript
export function add() {
  return ({
    type: 'ADD_ONE',
  })
}
```

Next, make a reducer to actually store the count.

```javascript
function count(state = 0, action) {
  switch (action.type) {
    case 'ADD_ONE':
      return state + 1;
    default:
      return state;
  }
}

export default combineReducers({
  count,
})
```

If you’ve worked with redux before you may have ignored this part, but it’s important to take a moment to think about what you just did here.

You just created a global state object with the a property of `count` . That property name is now unique for this particular action. If you wanted to create another property with a similar name you’d need to either give it a unique name such as `clickCount` or encapsulate one, the other, or both inside a nested object.

You’ll return to this idea in a bit. For now, use `connect` to pull the state into your display component.

```javascript
import React from 'react';
import { connect } from 'react-redux'

import { add } from '../../store/actions';
import Display from '../Display/Display';

export function ReduxCounter({ addOne, count }) {
  return(
    <Display
      addOne={addOne}
      count={count}
      title="Redux Counter"
    />
  )
}

const mapStateToProps = ({ count }) => ({ count });

const mapDispatchToProps = { addOne: add }

export default connect(mapStateToProps, mapDispatchToProps)(ReduxCounter);
```

Again, we won’t explore the details, but as a summary, the component is tapping into the global store so that you can both read from it, using `mapStateToProps`, and update it using `mapDispatchToProps`. Other than that, your display component doesn’t care where the data comes from.

So far, the thing that should jump out immediately is how much work is involved. There are more files, more imports, more complexity. That’s not a problem if the advantages outweigh the extra work — and they often will — but it is still something to consider.

### Context

The final component will use context. This is a fairly new form of state handling that was introduced in React 16.3. Context is similar to local state in many ways, but it has a slightly different implementation.

To start off, you need to create a context. This will set up a couple defaults that you will use when you create an implementation.

```javascript
import React from 'react';

const CounterContext = React.createContext({
  addOne(){},
  counter: 0,
});

export default CounterContext;
```
This sets the initial state of `0` and adds a noop function that you will override later.

To actually use context, you will first need to make a **Provider**. A provider is the base component that will hold the state object and the methods for updating the state.

```javascript
import React, { Component } from 'react';
import CounterContext from './CounterContext';

export default class ContextProvider extends Component {
  state = {
    count: 0,
  }

  addOne = () => {
    this.setState(state => ({count: state.count + 1}))
  }
  
  render() {
    const { count } = this.state;

    const value = {
      addOne: this.addOne,
      count,
    };

    return (
      <CounterContext.Provider value={value}>
        {this.props.children}
      </CounterContext.Provider>
    )
  }
}
```

If you scroll up, you’ll notice this looks very similar to the local state component. The big difference is that you are combing the state and the update function in single object which you then pass down into a **Provider** component that wraps everything else. In this case, you are wrapping `this.props.children` but you can also wrap a component directly.

To use this component, you’d wrap some other components. In this case, you’ll wrap a single component.

```javascript

import React from 'react';
import CounterProvider from '../ContextCounter/CounterProvider';
import OtherComponent from '../ContextCounter/OtherComponent';

export default function Counter() {
  return(
    <div>
      <CounterProvider>
          <OtherComponent />
      </CounterProvider>
    </div>
  )
}
```

Notice, you are not passing any props to `OtherComponent` . More clearly, you are not _explicitly_ passing anything to `OtherComponent`. The provider is holding the state information so that you can tap into it later.

Even if you go several components deep, you will still be able to access the provider’s state. So if `OtherComponent` returns another component:

```javascript
import React from 'react';
import AnotherComponent from './AnotherComponent';

export default function OtherComponent() {
  return(
    <AnotherComponent />
  )
}
```

And `AnotherComponent` returns yet _another_ component. That information is still hanging in the background.

```javascript
import React from 'react';
import ContextDisplay from './ContextDisplay';

export default function AnotherComponent() {
  return(
    <ContextDisplay />
  )
}
```

Ok, that’s far enough. It’s time to pull out the state and do something with it.

To get the information you need to create a **Consumer**. The consumer uses render props to pull out that single value object. Remember, `value` contains `count` and `addOne` . You can use destructuring as a short hand.

Now that you have the state and the function to update state, you can pass it into your display component:

```javascript
import React from 'react';
import CounterContext from './CounterContext';
import Display from '../Display/Display'

export default function ContextDisplay() {
  return(
    <CounterContext.Consumer>
      {({ addOne, count}) => {
        return (
          <Display
            addOne={addOne}
            count={count}
            title="Context Counter"
          />
        )
      }}
    </CounterContext.Consumer>
  )
}
```

### Comparisons 

As you saw, any form of state handling can do the same thing. In reality, if you put effort into it, you can make any state handling system work for any piece of data in your application.

Of course, who wants to put effort into it? The goal is always less effort, better results. With that in mind, it’s time to compare the different approaches.

### Simplicity

This is the easiest comparison. Nothing beats local state particularly when you use React hooks. It’s quick, it does that job and it’s easy to refactor into a more complicated system if necessary. As a rule, you should always start with local state. If you need something more complicated, you can refactor as you go.

For this reason, Local State wins for simplicity.

|             | Local | Global | Context |
|-------------|-------|--------|---------|
| Easy         | ✅    | ❎     | ❎ |
| Encapsulated | `?`   | `?`    | `?` |
| Available    | `?`   | `?`    | `?` |

### Encapsulation

Next, let’s explore how well the data is encapsulated. If data is well encapsulated, you can reuse a component multiple times without any concern that another component may accidentally change your state. Encapsulation isn’t necessarily a goal in itself. There are times where you absolutely want different components to access and modify shared data (more on that in the next section).

As with most things, it’s about predictability. If you expect a component to be independent, the data shouldn’t be open to modifications by other components.

To explore encapsulation, you’ll need to add multiple versions of a component to see how they interact.

To start off return to the original page that has all three components.

![][multi-count]

The code looks something like this:

```javascript
import React from 'react';
import StateCounter from '../StateCounter/StateCounterHooks';
import ReduxCounter from '../ReduxCounter/ReduxCounter';
import CounterProvider from '../ContextCounter/CounterProvider';
import OtherComponent from '../ContextCounter/OtherComponent';

export default function Counter() {
  return(
    <div>
      <StateCounter />
      <ReduxCounter />
      <CounterProvider>
          <OtherComponent />
      </CounterProvider>
    </div>
  )
}
```

You have a simple parent component that wraps one instance of each component. To test encapsulation, all you need to do is add a second instance. 

Start off with the `StateCounter` component. The updated code looks like this:

```javascript
import React from 'react';
import StateCounter from '../StateCounter/StateCounterHooks';
import ReduxCounter from '../ReduxCounter/ReduxCounter';
import CounterProvider from '../ContextCounter/CounterProvider';
import OtherComponent from '../ContextCounter/OtherComponent';

export default function Counter() {
  return(
    <div>
      <StateCounter />
      <StateCounter />
      <ReduxCounter />
      <CounterProvider>
          <OtherComponent />
      </CounterProvider>
    </div>
  )
}
```

When you open that in the browser, you can easily see that the data is well encapsulated. Every time you click the add button, it only updates that count on that particular component.

![][double-local]

Now try adding a second redux component.

```javascript
import React from 'react';
import StateCounter from '../StateCounter/StateCounterHooks';
import ReduxCounter from '../ReduxCounter/ReduxCounter';
import CounterProvider from '../ContextCounter/CounterProvider';
import OtherComponent from '../ContextCounter/OtherComponent';

export default function Counter() {
  return(
    <div>
      <StateCounter />
      <ReduxCounter />
      <ReduxCounter />
      <CounterProvider>
          <OtherComponent />
      </CounterProvider>
    </div>
  )
}
```

Open this in the browser and notice how different it is. Every time you click on **Add **, you update the data on both components.

![][double-global]

In this case, these components are not independent. You can alter the data in any component that use the same action.

Now, there are certainly ways around this. You can have an `id` on the global store for each component. You can use an array. You can use a different namespace. But the fact is that global stores are best for global data. Encapsulation is not a primary goal.

How about context? Well, this is were things get a little more tricky. What does it mean to add a second component? Does that mean you add a second consumer? In this case, that would mean adding a second `OtherComponent` under the same provider. Or do you add a second `CounterProvider`?

Why not both? First, add another provider. Then in the existing provider, add a second instance of `OtherComponent`.

```javascript
import React from 'react';
import StateCounter from '../StateCounter/StateCounter';
import ReduxCounter from '../ReduxCounter/ReduxCounter';
import CounterProvider from '../ContextCounter/CounterProvider';
import OtherComponent from '../ContextCounter/OtherComponent';

export default function Counter() {
  return(
    <div>
      <StateCounter />
      <ReduxCounter />
      <CounterProvider>
        <OtherComponent />
      </CounterProvider>
      <CounterProvider>
        <OtherComponent />
        <OtherComponent />
      </CounterProvider>
    </div>
  )
}
```

Let’s see what happens.

![][multi-context]

When you click on the first component, the counter iterates, but the other counters do not. But when you click on the second context counter, it _will_ change the third counter. Similarily, if you click on the third counter, it will change the second.

This means that context is somewhere in the middle. It can be encapsulated when you want it to be, and it can be global when you want it to be.

Now, this claim comes with a big caveat. The ability to share date or keep it private all depends on how you order the components. Anything in the same hierarchy as the provider will share data, anything outside will not. This means that you may have to move providers up or down depending on how you want to use the consumers.

Still, at this point, you can finish filling out the chart. Local state is well encapsulated and context can be. They each get a check. Global state is not well encapsulated to it does not get credit.

|             | Local | Global | Context |
|-------------|-------|--------|---------|
| Easy         | ✅    | ❎     | ❎ |
| Encapsulated | ✅    | ❎     | ✅ |
| Available    | `?`   | `?`    | `?` |

### Availability

That just leaves one more area of concern: availability. The beauty part is that you don’t need to write any more code. Sometimes you want your data to be easy to access.

Consider a shopping cart. Pushing users to a sale is the most important thing your code can do. Consequently, nearly every part of your app will potentially need to know about the cart (how much is in it, the total, etc). This is data you want to make easily accessible.

In this case, you can use what you learned above. Local state, for example, could work, but you would have to store it so far up the component hierarchy that you’d be passing it down a lot of props.

Global state, on the other hand, was designed specifically for this. It’s an easy thumbs up 👍.

Context as you saw above can either be global (at least depending on where you put it in the hierarchy) or encapsulated. Go ahead and give it credit.

The final comparison looks like this:

|             | Local | Global | Context |
|-------------|-------|--------|---------|
| Easy         | ✅    | ❎     | ❎ |
| Encapsulated | ✅    | ❎     | ✅ |
| Available    | ❎    | ✅     | ✅ |


One thing that’s not reflected in this chart is the large collection of third-party code that ties in with a global store. If you want to use observables with your store, then you’re better off using [redux-observables](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2Fredux-observable%2Fredux-observable) than trying to roll your own solution.

### Building and Refactoring

You may look at this table and think that local state or context are somehow better. That’s not true. The best solution is the one that works best for your data and your app.

Still, there are a few common patterns that may help the decision making process.

First, always start with local state. As you saw above, local state is the easiest to build and prototype. When you start with local state you don't have to deal with all the complexities of setting up a `Provider` or writing actions and reducers. Even if you plan on moving to a more widely available state management, local state is a great way prove your concept.

Next, use context if you want to maximize reusability and local state is not sufficient. After you go through the hard work of building a component, you'll want to reuse it as much as possible. That may even mean reusing it across different project and teams.

By choosing context, you reduce the number of additional dependencies your independent component will need. In other words, it will be much easier to drop the new component into an existing project. The component will be internally independent. You gain all the advantages of widely available components while avoiding extra code.

Finally, don't worry about the rules. If your team uses global state and has a well defined structure in place, you'll do more harm by using a different solution. Internal consistency and clear communication should always be the first priority. If your new component breaks the pattern, you may save yourself a few minutes of coding, but your teammates will lose their ability to quickly skim and understand a component. New patterns aren't bad, but they should add value.

Each of these solutions exist for a reason. Your job is to find the fit that makes your code easiest to build and maintain.

  [initial-state]: https://images.ponyfoo.com/uploads/initial-e9c8dcc3726a4acba76a8f445c1fac9c.gif
  [multi-count]: https://images.ponyfoo.com/uploads/multicount-26b623cdb35448a4a979a51799f41c2b.gif
  [double-local]: https://images.ponyfoo.com/uploads/doublelocal-2621a9c3317a4193b1a3bc77a5f2e907.gif
  [double-global]: https://images.ponyfoo.com/uploads/doubleglobal-d6e57f1ba52d49e4b2507e2af0c71a93.gif
  [multi-context]: https://images.ponyfoo.com/uploads/multicontext-7ab077100e344cdcb678ad43de92ee08.gif
