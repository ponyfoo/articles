### Creating Predictable Outcomes with Mocks

Suppose you have an application that manages employees or users. You need to make a simple function that will create a new employee, but you want to make sure that the employee always contains certain fields and will have a unique id.

Your code will look something like this:

```js
import uniqueId from 'lodash/uniqueId'

export function createEmployee(details) {
  return {
    name: '',
    position: '',
    ...details,
    key: uniqueId()
  }
}
```

In this case, you are using the Lodash method `uniqueId` to, *well*, create a unique ID. Specifically you are creating a key that you can use if you do any manipulations on an array of employees. The key, though, is a minor thing.

For the most part, your tests should focus on other aspects of your code such as ensuring default fields while allowing those fields to override. Here are two simple tests.

```js
import { createEmployee, } from './utils'

describe('createEmployee', () => {
  it('should create a blank employee', () => {
    const blank = createEmployee()
    const expected = {
      name: '',
      position: '',
      key: '1'
    }

    expect(blank).toEqual(expected)
  })

  it('should create use existing details when creating employee', () => {
    const blank = createEmployee({ name: 'Bill' })
    const expected = {
      name: 'Bill',
      position: '',
      key: '2'
    }

    expect(blank).toEqual(expected)
  })
})
```

Notice anything strange in the expectations? In the first case, the key is `'1'` in the next test it’s `'2'`. Here’s where the problems start to sneak in. Every time you add a test, you need to iterate the `key`. The `uniqueId` function is doing exactly what you would want in production, but in the testing case, it’s creating a situation where your tests _always_ have to run in the same order.

Suppose you needed to change the function slightly to handle cases where you get an object with a field of `firstName` and you want to use as the `name` field:

```js
import uniqueId from 'lodash/uniqueId'

export function createEmployee(details = {}) {
  const { firstName: name, ...rest } = details

  return {
    name: name || '',
    position: '',
    ...rest,
    key: uniqueId()
  }
}
```

Your tests now look like this:

```js
import { createEmployee, } from './utils.next'

describe('createEmployee', () => {
  it('should create a blank employee', () => {
    const blank = createEmployee()
    const expected = {
      name: '',
      position: '',
      key: '1'
    }

    expect(blank).toEqual(expected)
  })

  it('should create use existing details when creating employee', () => {
    const blank = createEmployee({ name: 'Bill' })
    const expected = {
      name: 'Bill',
      position: '',
      key: '2'
    }

    expect(blank).toEqual(expected)
  })

  it('should convert firstName to name', () => {
    const blank = createEmployee({ firstName: 'Bill' })
    const expected = {
      name: 'Bill',
      position: '',
      key: '3'
    }

    expect(blank).toEqual(expected)
  })
})
```

What happens if you change the order? Suppose you wanted to only run the third test:

```js
import { createEmployee, } from './utils.next'

describe('createEmployee', () => {
  // … other tests …

  it.only('should convert firstName to name', () => {
    const blank = createEmployee({ firstName: 'Bill' })
    const expected = {
      name: 'Bill',
      position: '',
      key: '3'
    }

    expect(blank).toEqual(expected)
  })
})
```

You’d get an error. But it’s not because your code is wrong. The order has just changed.

```diff
  expect(<mark>received</mark>).toEqual(<mark>expected</mark>)

  Expected value to equal:
-   {"key": "3", "name": "Bill", "position": ""}
  Received:
+   {"key": "1", "name": "Bill", "position": ""}

  Difference:

-   Expected:
+   Received:

    Object {
-     "key": "3",
+     "key": "1",
      "name": "Bill",
      "position": "",
    }

  …
```

So how can you fix it? In this case, you have a couple of options.

The easiest option is to override it inside the test itself. By using `mockImplementation`. 

```js
import { createEmployee, } from './utils.next'
import uniqueId from 'lodash/uniqueId'

uniqueId.mockImplementation(() => '1')

describe('createEmployee', () => {
  it('should create a blank employee', () => {
    const blank = createEmployee()
    const expected = {
      name: '',
      position: '',
      key: '1'
    }

    expect(blank).toEqual(expected)
  })

  it('should create use existing details when creating employee', () => {
    const blank = createEmployee({ name: 'Bill' })
    const expected = {
      name: 'Bill',
      position: '',
      key: '1'
    }

    expect(blank).toEqual(expected)
  })

  it('should convert firstName to name', () => {
    const blank = createEmployee({ firstName: 'Bill' })
    const expected = {
      name: 'Bill',
      position: '',
      key: '1'
    }

    expect(blank).toEqual(expected)
  })
})
```

Notice what’s happening. First, you import the code with `import uniqueId from 'lodash/uniqueId'`, then you add a `mockImplementation` to the function you want to control. Jest will now use the `mockImplementation` rather than the actual code anytime you call the function.

In other words, you now have a function that returns the result you want everytime. After that, you can write each test expect the exact same key. After all, you don’t really care what the key is, you just want it to be there.

Another option is to just define the return value using `mockReturnValue`. The steps are the same, but instead of returning a function, you just declare the desired return value.

```js
import { createEmployee, } from './utils.next'
import uniqueId from 'lodash/uniqueId'

uniqueId.mockReturnValue('2')

// Tests are the same
```

Those are both perfectly good solutions. You may reach for `mockImplementation` in situations where you need to check parameters before you decide what to return.

The problem with both solutions is that you have a little extra boilerplate for every single test suite that uses the `uniqueId`. To solve that, you can use a similar technique to mocking out the API.

Create a new directory called `__mocks__` at the same level as your `node_modules` directory. This will almost certainly be the root of your project. Inside `__mocks__` create a file structure that emulates your import. In this case, you need a Lodash directory that contains a `uniqueId.js` file.

```
| __mocks__
| + lodash
|   + uniqueId.js
| node_modules
```

Inside, `uniqueId.js` export either your `mockImplementation` or your `mockReturnValue`. The only difference is that you have to create a `jest.fn()` first. It’s not strictly required that you return `jest.fn()` you can just return any function. However, by returning a mocked `jest.fn()` you have more options to change the return value (more on that in a bit).

```js
export default jest.fn().mockImplementation(() => '1')
```

 Now that you have mocked the function at the `import` level, you don’t need to do anything in your actual test. The mock is detected for you.

```js
import { createEmployee, } from './utils.next'

describe('createEmployee', () => {
  it('should create a blank employee', () => {
    const blank = createEmployee()
    const expected = {
      name: '',
      position: '',
      key: '1'
    }

    expect(blank).toEqual(expected)
  })

  // Other tests

})
```

### Changing on the fly

Of course, if you want to change if for an individual test you can still do so. Suppose you had a helper function that sorts the employees by key. This would, essentially, reset the order.

The function is pretty straightforward:

```js
export function sortEmployees(employees) {
  return [...employees].sort((a,b) => Number(a.key) - Number(b.key))
}
```

This makes testing a little more tricky. Remember, you just mocked the function so it always returns the same value. And that is good for all the previous tests, but now you need the few _different_ unique ids.

Fortunately, once you have it mocked, it’s easy to make changes at runtime.. As a reminder here’s the mock in `__mocks__/lodash/uniqueId.js`:

```js
export default jest.fn().mockImplementation(() => '1')
```

The important thing is that you are exporting a jest function. That means you can import that mocked function and change it on the fly.

First, import `uniqueId` into your test:

```js
import uniqueId from 'lodash/uniqueId'
```

Now, in your test anytime you want to change the value, you declare what you want it to return. As a safety, use the method `mockImplementationOnce` this will ensure that the method will only return the alternate value, well, once. Then it will return to the original mock implementation. This will prevent some confusing bugs from sneaking into your code.

```js
import uniqueId from 'lodash/uniqueId'

uniqueId.mockImplementationOnce(() => '2')

uniqueId()
// 2

uniqueId()
// 1
```

With this power you can create a few employees with different unique ids. After that the test is simple:

```js
import { createEmployee, sortEmployees } from './utils.next'
import uniqueId from 'lodash/uniqueId'

describe('createEmployee', () => {
  // … tests

  it('should sort employees by id', () => {
    const blank1 = createEmployee({ name: 'Olivia'})

    uniqueId.mockImplementationOnce(() => '2')
    const blank2 = createEmployee({ firstName: 'Xander'})

    uniqueId.mockImplementationOnce(() => '3')
    const blank3 = createEmployee({ name: 'Bill' })

    const expected = ['Olivia', 'Xander', 'Bill']
    const employees = [blank2, blank3, blank1]

    const mapped = sortEmployees(employees).map(employee => employee.name)

   expect(mapped).toEqual(expected)
  })

  // … more tests
})
```

In this situation, you are creating different unique ids for only this one specific set of assertions. Everything else falls back to the predictable unique id.

### Ignoring Black Boxes

There’s more ways you can use `mockImplementation` to change or modify individual library methods. But occasionally, you want to completely remove a piece of external code. You may have a heavy piece of code that itself has a large number of external dependencies. Rather than import the large code, it may be easier to bypass it altogether. This is a good technique when you have a piece of code that’s indepedent from the rest of the code, but still is included.

In other words, whenever a sales or marketing person forwards you an email with code you “just have to drop in” — a tracking pixel, a buy now widget or so on — you may have something you need to ignore.

Suppose you have a piece of code that adds a map of some sort. Assume the stripped down version of the code you are importing looks something like this:

```html
const tiles = [...Array(1000)].map(() => '<div>🎄</div>')
const container = `
  <div class="container">${tiles.join('')}</div>
`

export default function render() {
  const mapContainer = document.querySelector('#map')
  mapContainer.innerHTML = container
}
```

Nothing too difficult here, but it does make a few assumptions about the DOM. It runs a `querySelector` and expect to see the `innerHTML`.

Suppose that you have a small React component that list some ride options and also renders the map. The map is effectively an image as far as you are concerned. You just need to make sure that a `div` exists so you can add the map to it at runtime.

```html
import React, { useEffect } from 'react'
import renderMap from 'streetMap'

export default function RideFinder({ rides }) {
  useEffect(() => {
    renderMap()
  }, [])
  return (
    <div>
      <h3>Find a Ride</h3>
      <ul>
        {rides.map(ride => <li key={ride.id}>{ride.name}</li>)}
      </ul>
      <div id="map"></div>
    </div>
  )
}
```

In your test, you don’t care about the map. It’s a blackbox, but you do want to make sure you are creating the correct number of list items.

```html
import React from 'react'
import { mount } from 'enzyme'
import RideFinder from './RideFinder'

function flushPromises() {
  return new Promise(resolve => setTimeout(resolve, 10))
}

describe('RideFinder', () => {
  const rides = [
    {
      name: 'River Trail',
      id: 1,
    },
    {
      name: 'Downtown Bikepath',
      id: 2,
    }
  ]

  it('should render ride finder', async () => {
    const wrapper = mount(<RideFinder rides={rides}/>)
    const renderedRides = wrapper.find('li')
    await flushPromises()
    expect(renderedRides.length).toBe(2)
  })
})
```

Note there are a few complications. To make sure `useEffect` runs in enzyme, you need a small timeout to ensure everything finishes rendering. There are [other testing options than `enzyme`][react-testing-library], but `enzyme` remains popular.

After running this code, you get an error:

`Uncaught TypeError: Cannot set property 'innerHTML' of null`

The DOM manipulations are causing problems. In this case, you may want to mock out the map code.

As with `lodash` above, make a file of the same name as the module in the `__mocks__` directory. In this case, you are using the main default, to you don’t need an additional directory. Together, it would look like this:

```
| __mocks__
| + lodash
|   + uniqueId.js
| + streetMap.js
| node_modules
```

Inside of `streetMap.js` simply return a mock function. Remember, in this case, you don’t really care what it does. You just want to make sure it runs.

```js
export default jest.fn()
```

At this point, your test will run with no problems. However, if you want to ensure that the your code will run the blackbox, even though you don’t care about the output, you can still assert that it works. As with Lodash, import the code you want to check, then add an expect function to see if it is called:

```html
import React from 'react'
import { mount } from 'enzyme'
import RideFinder from './RideFinder'
import renderMap from 'streetMap'

function flushPromises() {
  return new Promise(resolve => setTimeout(resolve, 10))
}

describe('RideFinder', () => {
  const rides = [
    {
      name: 'River Trail',
      id: 1,
    },
    {
      name: 'Downtown Bikepath',
      id: 2,
    }
  ]

  it('should render ride finder', async () => {
    const wrapper = mount(<RideFinder rides={rides}/>)
    await flushPromises()
    expect(renderMap).toBeCalled()
  })
})
```

You now have a test that ensures that blackbox code is called while preventing the code from running itself. Since you have no control over the code, and it is fairly independent, this is a good strategy to ensure coverage without overcomplicating your test suite.

### Using mocks responsibly

Mocks give you a lot of power to bypass code. However, that power can also be dangerous. As soon as you bypass code, you are creating tests that do not fully execute your code. If you make false assumptions in your mocks, errors will begin to sneak into your code. And I can tell you from experience, this can happen quickly and tracking it down can be frustrating.

As a rule, only use mocks in situations where the side effect is _very_ isolated. As you can see with the Lodash functions, you don’t want to mock everything, just the piece that causes problems. If you use other Lodash methods, you want to ensure they run properly.

Same with the blackbox code. Nothing in the rest of your code is dependent on the results. So you can safely mock it out without creating too many additional assumptions in your tests.

Mocks can be a powerful tool that can get you more code coverage with a simple interface. Use them with caution, but use them confidently when you do.

[react-testing-library]: https://www.github.com/kentcdodds/react-testing-library "react-testing-library on GitHub"
