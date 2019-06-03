As part of [my mentorship business](https://cleverbeagle.com), I have the unique opportunity to work with React daily across a wide range of projects. Teaching others how to build their own software products, I've come to find that most problems in React can be solved with some very simple techniques. 

While some cases may call for a full-blown architecture using something like Redux (or other fanciness), a lot of the time React's built-in lifecycle methods and local state do the trick. In my opinion, the mark of a really great developer is one who can solve a problem with as few moving parts as possible. While it can feel great to build a monument to engineering in the form of a complex system, often it's just overkill.

In this tutorial, I'm going to share what I call my "data survival kit:" the most common patterns and hacks I use in my day-to-day work for managing data. By the time you finish, you'll have everything you need to build data-driven UIs that offer a polished user experience.

# Loading Data via Hooks

As most folks are building "database apps," loading data is arguably one of the most common tasks to perform. Ultimately, where your data is coming from dictates your need for this. This pattern is best utilized in apps using a REST API or an RPC (remote procedure call) to fetch data.

For example, if you're loading data via GraphQL, it's likely that you'll use this technique sparingly (e.g., when you need to fetch data programmatically as opposed to on page load), relying on tools like Apollo to load your data for you.

```javascript
import React, { useEffect, useState } from 'react';

function Posts() {
  const [posts, setPosts] = useState([]);

  function getData() {
    fetch('https://jsonplaceholder.typicode.com/posts').then(async (fetchedPosts) => {
      const postsAsJson = await fetchedPosts.json();
      setPosts(postsAsJson);
    });
  }

  useEffect(() => {
    getData();
    const pollForData = setInterval(() => getData(), 5000);
    return () => {
      clearTimeout(pollForData);
    };
  }, []);

  return (
    <div>
      <h4>Posts</h4>
      {posts.map(({ id, title, body }) => (
        <div key={id}>
          <h3>{title}</h3>
          <p>{body}</p>
        </div>
      ))}
    </div>
  );
}

export default Posts;
```

This example is relatively new but is ultimately just a refactor of a previous approach where you'd use `componentWillMount()` or `componentDidMount()` to use React's new hooks feature. The idea is simple here: when our component loads into memory, go and fetch the data it will need and put it onto state.

To make this work, we leverage the `useEffect()` hook to say "when our component loads, get our data and then set up a poll interval to refetch every five seconds." `useEffect()` is essential, here, because side effects (e.g., fetching data) are not allowed in the body of a functional component due to their ability to produce "confusing bugs and inconsistencies in the UI."

The idea at play here is that `useEffect()` will allow us to update the state of our functional component via the `updatePosts()` method returned from our call to the `useState()` hook. To prevent `useEffect()` from running on every render of our component, we pass an empty array `[]` as its second argument (this can contain values to conditionally fire `useEffect()` when they change—[learn more here](https://reactjs.org/docs/hooks-reference.html#conditionally-firing-an-effect)).

In order to clear out the interval when our component unmounts, `useEffect()` accepts a return value of [a "clean up" function](https://reactjs.org/docs/hooks-reference.html#cleaning-up-an-effect) (this function behaves similar to `componentWillUnmount()` in a class). The end result is that our data will be fetched and put onto state when our component initially loads and again every five seconds until our component unmounts.

The reason I like this pattern is that it makes data fetching easy to understand for beginners and veterans alike. It also makes tasks like polling/refetching to keep data up to date easy—no need to call to an out-of-scope refetch function or Redux action. Technically speaking, too, this is handy because it helps us to avoid getting tangled up in global state—something that should only be used if it's absolutely necessary.

# Autosaving Using State

As applications have evolved, features like "autosave" have become common enough for users to expect them. Fortunately, React—and by proxy, it's state—makes this sort of feature painless for us to implement.

The way I like to accomplish this is by write input changes to state directly (a controlled component) and then after a delay, make a call to write to the database. Depending on the UI and the amount of data involved, I'll either send up a `PATCH` for an individual field, or, a `PUT` for the entire object it's part of.

For the delay, I like to use this `delay` function that I picked up a few years ago—it's simple and has served its purpose well:

```javascript
const delay = (() => {
  let timer = 0;
  return (callback, ms) => {
    clearTimeout(timer);
    timer = setTimeout(callback, ms);
  };
})();
```

The basic premise of this function is that it's a self-clearing `setTimeout()`. So, when it's called, if it's called again before it's timeout `ms`, it clears itself to avoid overflowing JavaScript's call stack. After the specified delay in ms has passed, the function performs like a regular `setTimeout`, executing the code it contains (i.e., once a user stops typing for 3000ms, _then_ make the call).

```javascript
class UserProfile extends React.Component {
  state = {};

  handleLiveUpdate = (event) => {
    const { name, value } = event;
    const { updateProfile } = this.props;

    this.setState({ [name]: value }, () => {
      // After handleLiveUpdate has stopped being called for 3 seconds, call to update database.
      delay(() => {
        // Example #1: Updating the database via GraphQL mutation.
        updateProfile({
          variables: {
            [name]: value,
          },
        });

        // Example #2: Updating the database via Meteor Method.
        Meteor.call('users.updateProfile', { [name]: value }, (error) => {
          if (error) {
            alert(error.reason);
          }
        });

        // Example #3: Updating the database via HTTP PATCH.
        fetch('https://app.com/api/users', {
          method: 'PATCH',
          body: JSON.stringify({ [name]: value }),
        });
      }, 3000);
    });
  };

  render() {
    return (
      <form>
        <h4>User Profile</h4>
        <div>
          <label htmlFor="firstName">First Name</label>
          <input type="text" name="firstName" value={this.state.firstName} onChange={this.handleLiveUpdate} />
        </div>
        <div>
          <label htmlFor="lastName">Last Name</label>
          <input type="text" name="lastName" value={this.state.lastName} onChange={this.handleLiveUpdate} />
        </div>
        <div>
          <label htmlFor="emailAddress">Email Address</label>
          <input type="email" name="emailAddress" value={this.state.emailAddress} onChange={this.handleLiveUpdate} />
        </div>
      </form>
    );
  }
}
```

The idea here is that as the user makes changes to the form we want to autosave, we immediately set state. Then, in the "background," we utilize the `delay()` function to say "after they've stopped typing, persist the current state in the database." The database part depends on how you handle data in your application.

Because I spend a lot of my time [working in Meteor and GraphQL with the boilerplate I maintain, Pup](https://cleverbeagle.com/pup), I'll either use Meteor's `Meteor.call()` convention to send my data to the server for storage, or, [call to a mutation](https://ponyfoo.com/articles/graphql-in-depth-what-why-and-how#understanding-mutations) if I'm relying on GraphQL. If I'm building a mobile app, I'll rely on `fetch()` or a library like `axios()` to talk to my REST API.

# Using Local Storage to Persist Unsaved Data

One of the worst bits of UX is having a large form without a backup. As a user, there's nothing quite as disheartening as filling out a large form and accidentally hitting refresh to find all of your work is gone.

Fortunately, the trick to getting around this is pretty simple (and utilizes a similar approach to the autosave example above). This pattern necessitates that our data live on our component's state, allowing us to maintain the illusion of a user's data being persisted without hitting a database.

```javascript
import React from 'react';
import store from 'store';

class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    this.state = store.get('myApp.userProfile') || {};
  }

  handleUpdate = (event) => {
    const { name, value } = event;

    this.setState({ [name]: value }, () => {
      delay(() => {
        store.set('myApp.userProfile', this.state);
      }, 500);
    });
  };

  render() {
    return (
      <form>
        <h4>User Profile</h4>
        <div>
          <label htmlFor="firstName">First Name</label>
          <input type="text" name="firstName" value={this.state.firstName} onChange={this.handleUpdate} />
        </div>
        <div>
          <label htmlFor="lastName">Last Name</label>
          <input type="text" name="lastName" value={this.state.lastName} onChange={this.handleUpdate} />
        </div>
        <div>
          <label htmlFor="emailAddress">Email Address</label>
          <input type="email" name="emailAddress" value={this.state.emailAddress} onChange={this.handleUpdate} />
        </div>
      </form>
    );
  }
}
```

This should look pretty familiar. Everything here is identical to the autosave approach save for a few small details.

First, we've introduced a library `store` which will help us to get cross-browser access to local storage (or a comparable browser cache that's available to our user). Our usage of the library is limited to two calls: one when we load our component/page up for the first time and whenever a user changes their data.

The first takes place in our component's `constructor()` function. Here, we're setting our default state value relative to the current value of our local storage key (here, `myApp.userProfile` is the name of the key we've chosen to store our data in local storage). What we expect is that `store.get()` will return an object containing properties that reflect the state values our UI requires.

```javascript
  handleUpdate = (event) => {
    const { name, value } = event;

    this.setState({ [name]: value }, () => {
      delay(() => {
        store.set('myApp.userProfile', this.state);
      }, 500);
    });
  };
```

In order to get those values into local storage, we use a `handleUpdate` method on our component that uses the same approach to our autosave pattern. Here, we immediately set the user's input on state and then after a short delay, we update the local storage value via `store.set('myApp.userProfile', this.state)`.

Two details here: notice that we've reduced our delay significantly to `500ms`. This is because it's far less expensive to hit our local storage than it is to make a trip to the server. Also, notice that we're setting the entirety of the current `this.state` value onto local storage when we change any input. This, too, is inexpensive for performance and also saves us a messy `constructor()` loaded with calls to `store.get()` for each individual field.

# Reaching Into a Child Component via Refs

One of the bittersweet features of React is the ability to nest child components. It's bittersweet because while it makes composition easy—bringing together multiple components in a cohesive UI—it can make tasks like getting data out of child components a chore.

A simple hack that I like to leverage for this is accessing child components via refs. While it's more common to pass a function via props to a child component that handles tracking the child's state on the parent (or at least, notifying the parent when its internal state changes), this can get messy and cumbersome.

Instead, if all we care to know is the current internal state of a child component, refs make our lives a hell of a lot easier. Let's consider the example of a job application. We have some basic form fields we'd like to grab along with two lists: the candidates strengths and weaknesses.

```javascript
import React from 'react';

class List extends React.Component {
  state = {
    items: [],
  };

  handleRemoveItem = (id) => {
    this.setState(({ items }) => ({
      items: items.filter((item) => item.id !== id),
    }));
  };

  handleAddItem = (event) => {
    event.persist(); // Use event.persist() so we don't lose React synthetic event in nested function below.
    this.setState(({ items }) => ({
      // randomIdGenerator() is used for example here and doesn't exist.
      items: [...items, { id: randomIdGenerator(), item: event.item.value }],
    }));
  };

  render() {
    return (
      <div>
        {this.state.items.map(({ id, item }) => (
          <li key={id}>
            {item}
            <button onClick={() => this.handleRemoveItem(id)}>
              <i className="fas fa-remove" />
            </button>
          </li>
        ))}
        <form onSubmit={this.handleAddItem}>
          <input type="text" name="item" />
          <button type="submit">Add Item</button>
        </form>
      </div>
    );
  }
}
```

To manage those two lists, we use a nested component with its own state called `<List />`. Internally, the component gives the applicant an input to add as many list items as they choose. As a standalone component, this doesn't present us with any issues.

```javascript
import React from 'react';
import List from './path/to/List';

class JobApplication extends React.Component {
  state = {
    firstName: '',
    lastName: '',
    emailAddress: '',
  };

  handleSubmitApplication = (event) => {
    const { submitApplication } = this.props;

    // Example call to a GraphQL mutation here to perform the submission.
    submitApplication({
      variables: {
        ...this.state,
        strengths: this.strengths.state.items,
        weaknesses: this.weaknesses.state.items,
      },
    });
  };

  render() {
    return (
      <React.Fragment>
        <h1>Job Application</h1>
        <form onSubmit={this.handleSubmitApplication}>
          <div>
            <label htmlFor="firstName">First Name</label>
            <input type="text" name="firstName" value={this.state.firstName} onChange={this.handleUpdate} />
          </div>
          <div>
            <label htmlFor="lastName">Last Name</label>
            <input type="text" name="lastName" value={this.state.lastName} onChange={this.handleUpdate} />
          </div>
          <div>
            <label htmlFor="emailAddress">Email Address</label>
            <input type="email" name="emailAddress" value={this.state.emailAddress} onChange={this.handleUpdate} />
          </div>
          <label>What are some of your strengths?</label>
          <List ref={strengths => (this.strengths = strengths)} />
          <label>What are some of your weaknesses?</label>
          <List ref={weaknesses => (this.weaknesses = weaknesses)} />
          <button type="submit">Submit Job Application</button>
        </form>
      </React.Fragment>
    );
  }
}
```

Where things can potentially get messy is when we render the `<List />` component inside of a parent. Here, `<JobApplication />` represents that parent. In the `render()` we can see two instances of `<List />` being generated.

Traditionally, we could add a prop to `<List />` called `onUpdate()` that was called internally by `<List />` passing up the current items. Where this becomes problematic is in having to track the list's state both internally as well as on the parent (or having the parent feed the child its data—no bueno unless we need to load data from the database).

To simplify this, we add a `ref` to each instance of the `<List />`, assigning it back to our `<JobApplication />` component as either `this.strengths` or `this.weaknesses`. What's great about this is that now, we have direct access to these components from within `<JobApplication />`.

When our applicant submits the form, all we need to do get its items is call to either `this.strengths.state.items` or `this.weaknesses.state.items`.

## Bonus: Controlling a Child via the Parent

You may be wondering, does this mean I can control the child component via refs as well? Yes. You want to be careful with this, however, as it can produce unexpected side effects. For example, when it comes to updating a component's internal state relative to a parent's data, it's best to pass changes via props.

Sometimes, though, this isn't always feasible or wanted. For example, consider our example above. After the application is submitted, let's assume we want to "reset" the form. Because our `<List />` components maintain their state internally, this means we need to manipulate their state from `<JobApplication />`.

```javascript
import React from 'react';
import List from './path/to/List';

class JobApplication extends React.Component {
  state = {
    firstName: '',
    lastName: '',
    emailAddress: '',
  };

  handleSubmitApplication = (event) => {
    const { submitApplication } = this.props;

    // Example call to a GraphQL mutation here to perform the submission.
    submitApplication({
      variables: {
        ...this.state,
        strengths: this.strengths.state.items,
        weaknesses: this.weaknesses.state.items,
      },
    }).then(() => {
      this.strengths.setState({ items: [] });
      this.weaknesses.setState({ items: [] });
    });
  };

  render() {
    return (
      <React.Fragment>
        <h1>Job Application</h1>
        <form onSubmit={this.handleSubmitApplication}>
          <div>
            <label htmlFor="firstName">First Name</label>
            <input type="text" name="firstName" value={this.state.firstName} onChange={this.handleUpdate} />
          </div>
          <div>
            <label htmlFor="lastName">Last Name</label>
            <input type="text" name="lastName" value={this.state.lastName} onChange={this.handleUpdate} />
          </div>
          <div>
            <label htmlFor="emailAddress">Email Address</label>
            <input type="email" name="emailAddress" value={this.state.emailAddress} onChange={this.handleUpdate} />
          </div>
          <label>What are some of your strengths?</label>
          <List ref={strengths => (this.strengths = strengths)} />
          <label>What are some of your weaknesses?</label>
          <List ref={weaknesses => (this.weaknesses = weaknesses)} />
          <button type="submit">Submit Job Application</button>
        </form>
      </React.Fragment>
    );
  }
}
```

Here, when our applicant submits the form, as soon as we get an "all good" back from the database, we call to `this.strengths.setState()` and `this.weaknesess.setState()` to reset their internal state.

Again, use this wisely—I consider this a hack which means it should be applied with caution and ample experimentation/testing.

## Conclusion

Using data in React doesn't need to be complicated. In fact, one of the joys of using React is that it can help you produce interactive UIs with very little effort. If you're already committed to React, it's worth considering how you might simplify your own use of it. More often than not, the problems I see in React applications involve unnecessarily complex data patterns.
