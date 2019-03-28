Here’s an even more basic example. You want to fetch some data from an API and display the first couple items:

```js
export function getRecent() {
  return fetch('/albums')
    .then(response => response.json())
    .then(albums => {
      return albums
        .reverse()
        .slice(0, 2)
        .map(({title}) => title);
    })
}
```

Simple to read. Simple to understand. But how would you test it without hitting the live API?

There’s an unfortunate belief that testing is hard. Early in my career, I heard one developer estimate that it doubled development time. And he was arguing _for_ testing, that it was worth the extra time. The problem is that most testing is easy. But testing impure functions can be very, very difficult.

Fortunately, the [Jest testing framework](https://medium.com/r/?url=https%3A%2F%2Fjestjs.io) has simplified a lot of the complexities behind testing. The secret is a tool called a [mock](https://medium.com/r/?url=https%3A%2F%2Fjestjs.io%2Fdocs%2Fen%2Fmanual-mocks). A mock is effectively a shortened version of a function. Instead of running the function you stub out the result you want. Your code will return what you want instead of running the live code.

In other words, you say what an API _should_ return. The mock will bypass the live API and return the data directly. Instead of relying on a date function that changes every 24 hours, you effectively freeze it in place by returning the same date every time. 

> ##### 🚨 Warning 🚨
>
> Mocks are a great tool, but they should be a last resort. If your tests use a significant amount of mocks (or spies or stubs or other testing trick), consider refactoring first. After you have isolated the complexities as much as possible, then it’s time to use mocks.

### Jest: A Brief Intro

Jest is a testing framework developed at facebook, it’s used most often with React, but it does work independently.

Jest offers runtime testing tools. an expect library and system for mocks and spies right out of the box.

At it’s most basic, a Jest test will look nearly identical to mocha or jasmine or any other framework. In fact, Jest started as a fork of Jasmine.

```js
import { double } from './double';

describe('Some Test', () => {
  it('should add two number', () => {
    expect(double(1)).toEqual(2);
  });
});
```

In this case, `toEqual` is the assertion.

That’s pretty much all you need for the purposes of this article. There are plenty of extra features that you can read about in the [documentation](https://medium.com/r/?url=https%3A%2F%2Fjestjs.io). You can also look there for installation and configuration. If you want to follow along, check out the [repo for this article](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2Fjsmapr1%2Fjest-mock-dive).

### Creating Testable API Functions

Return to your API code. Now that you have your code and you have your test suite all set up, it’s time to write tests. As a reminder, you have a piece of code that fetches API data and parses the results.

```js
export function getRecent() {
  return fetch('/albums')
    .then(response => response.json())
    .then(albums => {
      return albums
        .reverse()
        .slice(0, 2)
        .map(({title}) => title);
    })
}
```

The first step in creating a mock is isolating the pain point. In this case, the problem is the `fetch` block. Specifically this part:

```js
return fetch('/albums')
    .then(response => response.json())
```

The rest of the code isn’t so bad. It’s just a series of array manipulations.

The next step if figuring out what kind of data from the hard part you expect to pass down to the easy part. Again, all you need to do is look at what the live API will return by either hitting it directly or by getting the contract from some other team. In this case, you plan to get the albums of the great doom metal band Sleep. The data you expect to get will look something like this:

```json
[
  {
    "year": 1992,
    "title": "Holy Mountain"
  },
  {
    "year": 1999,
    "title": "Jerusalem"
  },
  {
    "year": 2014,
    "title": "The Clarity"
  },
  {
    "year": 2018,
    "title": "The Sciences"
  }
]
```

Now, you should split the function. Pull the hard part away from the easy part. That will make your tests cleaner and easier to mock.

As a quick side note, some consider it bad practice to break up your code to make it more testable. I would disagree. Testability tells you something about the complexity of your code. The harder it is to test, the more complex it tends to be. When you are refactoring your code to make it more testable, you just changing the complex to the simple.

Since you’ve isolate the complex part, the next the step is pulling the `fetch` call out into a separate function. Call the new function`service` and since it now has a separate responsibility go ahead and move it into a new file called `service.js`. To keep things clean, move it to a separate directory called `api` this will be important later when you set up the mock.

```js
export default function service() {
  return fetch('/albums')
    .then(response => response.json())
}
```

Now you need to import the service and replace the original `fetch` method. Your original function will be a little smaller, but you still are able to perform the exact same actions as your did before:

```js
import service from '../api/service';

export function getRecent() {
  return service()
    .then(albums => {
      return albums
        .reverse()
        .slice(0, 2)
        .map(({ title }) => title);
  })
}
```

At this point, you should start to see an opportunity for a mock. Remember, a mock is just replacing a live piece of code with a simple function that returns the data you expect. 

In this case, you have a function `service` that returns a `Promise` that returns some sort of array. You happen to know this service hits an API, but that doesn’t really matter. It can be loading data from `localstorage` as far as you’re concerned.

Time to jump into the test. Here’s what it’ll look like:

```js
import { getRecent } from './recent';

describe('getRecent', () => {
  it('should get albums', async () => {
    const albums = await getRecent();
    const expected = [
      'The Sciences',
      'The Clarity'
    ]
    expect(albums).toEqual(expected);
  })
})
```

Since `getRecent` returns a promise, you can use `async/await` syntax. This code says, run the `getRecent` function and once it resolves, make sure the albums are as expected.

Jest will wait for a promise to resolve if you return. That means you can also write the test like this:

```js
import { getRecent } from './recent';

describe('getRecent', () => {
  it('should get albums', () => {
    return getRecent()
      .then(albums => {
        const expected = [
          'The Sciences',
          'The Clarity'
        ]
        expect(albums).toEqual(expected);
      })
  })
})
```

Either way, you still need to account for the service. Currently, it will still try (and probably fail) to hit a live endpoint.

Now, it’s time to start mocking the results.

### Mocking an API Result

There are multiple ways to mock data, but the easiest is by creating a manual mock at the same level as the code you need to bypass. 

In practical terms, you are going to make a file of the same name that returns data that you want instead of executing a function to get that data.

Start by making a directory called `__mocks__` in the _same_ directory as `service.js` Suppose you had a directory called `api` that contained `services.js`, you’d add your `__mocks__` in the same `api` directory like this: 

```
| api
  | __mocks__
    | service.js
  | service.js
```

Inside `__mocks__/service.js` create a function with the same name, but that returns the data you want.

```js
const albums = [
  {
    year: 1992,
    title: 'Holy Mountain'
  },
  {
    year: 1999,
    title: 'Jerusalem'
  },
  {
    year: 2014,
    title: 'The Clarity'
  },
  {
    year: 2018,
    title: 'The Sciences'
  },
];

export default function service() {
  return Promise.resolve(albums)
}
```

Just as with the actual `service` function. This one returns a `Promise` but unlike the original function, this one has the data hard coded. It will never change.

Now that you’ve set up your mock, you need to alert your test that you want to use the manual mock instead of the real function. To do that, you need to use `jest.mock(path)` where the `path` is the location of the file you want to mock.

Suppose you had a file structure like this:

```
| albums
  | recent.js
  | recent.spec.js
| api
  | __mocks__
    | service.js
  | service.js
```

In your test, add the line `jest.mock('../api/service')`. Notice, you are not adding the path to the `__mocks__` directory, you are using the same path as if you were importing the function. Jest will find the mock for you. You are using the same path as you would in `recent.js`.

```js
import { getRecent } from './recent';

jest.mock('../api/service');

describe('recent', () => {
  it('should get albums', async () => {
    const albums = await getRecent();
    const expected = [
      'The Sciences',
      'The Clarity'
    ]
    expect(albums).toEqual(expected);
  })
})
```

Now, you are not hitting a live API. You are hitting an internal simulation. And it only takes one line of code. You don’t need network connectivity. You don’t need to worry about results changing. You don’t need to spin up a separate service. Everything is happening internally.

### What’s next?

Mocking API data is one of the best uses of mocks. Nearly all JavaScript projects need to be able to access remote data. However, mocks can do a lot more. They can also deal with complicated side effects such as date time or DOM manipulations. In my next article, I'll show you how to mock out `moment` and other libraries that return data that changes depending on the day or even the location of the test.
