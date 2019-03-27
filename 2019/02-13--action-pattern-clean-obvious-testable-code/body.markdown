My primary question? "How do I organize complex code?" As far as questions go that was a ball of yarn, but over several videos he explained the parts I was missing:

- Using explicit names that can't be mistaken.
- Breaking your code into functions that do one thing.
- Using TDD (test-driven development) to guide yout work.

Still green, some of this made sense and some of it didn't. The other problem was that Bob's language of choice was Java, not _JavaScript_. This meant that I was able to grasp what he was saying at a high level, but practically I was still stumped.

### Several iterations later...

Eventually, what Bob taught started to sink in. As I gained experience, I slowly started to organize my code into a pattern (supported by a short list of rules):

1. Any code that involves multiple steps should be moved into its own file/module.
2. That file/module should be given a name that describes what those steps lead up to.
3. Each step in that code should be a single function with a name that describes exactly what it does (even if it's longer than we prefer).
4. If the code fails, it should be easy to see exactly _where_ it failed without a lot of backstepping.

What started out as an informal set of rules for myself eventually evolved into a concrete pattern. After years of iteration and putting it through the paces on client and personal projects, in 2017 the action pattern was christened.

### How Actions work

For the remainder of this tutorial, we're going to convert a mock API endpoint for signing up new users in a mobile app into an action. Our goals:

1. Understand the structure of an action.
2. Learn how to use JavaScript Promises with actions.
3. Find a greater "why" for using actions.
4. Understand how writing tests is simplified by using actions.

## Converting Our Existing API Endpoint

Our app, Doodler (a paid social network for artists), handles its signups via an existing Express-based API. When a new user signs up in the app, a request is made to their API at `https://doodler.fake/api/v1/users/signup`.

At that endpoint, the following steps take place:

- A new user is created in the users collection.
- A new customer is created on Stripe.
- A customer is created in the customers collection.
- A welcome email is generated.
- A "new user" message is sent to the company's Slack.

Together, these five steps represent the _action_ of signing up a new user. Because some of the steps are dependent on prior steps, we want to have some way to "stop" our code if earlier steps fail. Before we get into the weeds, let's take a look at the code we have now:

```javascript
/* eslint-disable */

import mongodb from '/path/to/mongodb';
import settings from '/path/to/settings';
import stripe from '/path/to/stripe/api';
import imaginaryEmailService from '/path/to/imaginaryEmailService';
import slackLog from '/path/to/slackLog';

export default {
  v1: {
    '/users/signup': (request, response) => {
      mongodb.connect(settings.mongodb.url, function (error, client) {
        const db = client.db('production');
        const users = db.collection('users');
        const customers = db.collection('customers');

        users.insert({ email: request.body.email, password: request.body.password, profile: request.body.profile }, async function (error, insertedUser) {
          if (error) {
            throw new Error(error);
          } else {
            const [user] = insertedUser;
            const userId = user._id;
  
            const customerOnStripe = await stripe.customers.create({
              email: request.body.email,
            });

            customers.insert({ userId, stripeCustomerId: customerOnStripe.id }, async function (error, insertedCustomer) {
              if (error) {
                throw new Error(error);
              } else {
                imaginaryEmailService.send({ to: request.body.email, template: 'welcome' });
                slackLog.success({
                  message: 'New Customer',
                  metadata: {
                    emailAddress: request.body.email,
                  },
                });

                response.end();
              }
            });
          }
        });
      });
    },  
  },
};
```

Looking at this code, assuming that all of the parts in use work on their own, it's plausible that this code will work. What's distinct about this code, however, is that it's not terribly organized. It contains a lot of nested calls and not much flow control (i.e., if something fails, the whole house of cards falls).

This is where we start to tiptoe up to the "works" vs. "works well" chasm. Unfortunately, it's code like this that leads to a lot of wasted time chasing down and fixing bugs. It's not that the code doesn't work, it's that it works _unpredictably_.

You're probably saying "well yeah, _all_ code is unpredictable." You're not wrong. But, if we're smart we can significantly reduce the amount of unpredictability, giving us more time to focus on fun stuff—not fixing mistakes of the past (either of our own making or someone on our team).

## Introducing the Action Pattern

First and foremost, it's important to understand that the action pattern is vanilla JavaScript. It's a _pattern_ to follow, not a library or framework to implement. This means that using actions requires a certain level of discipline (the majority of which can be automated via snippets in your IDE).

To get started with our conversion, let's look at a skeleton version of an action and then build it up to handle our new user signup.

```javascript
/* eslint-disable consistent-return */

const actionMethod = (someOption) => {
  try {
    console.log('Do something with someOption', someOption);
    // Perform a single step in your action here.
  } catch (exception) {
    throw new Error(`[actionName.actionMethod] ${exception.message}`);
  }
};

const validateOptions = (options) => {
  try {
    if (!options) throw new Error('options object is required.');
    if (!options.someOption) throw new Error('options.someOption is required.');
  } catch (exception) {
    throw new Error(`[actionName.validateOptions] ${exception.message}`);
  }
};

export default (options) => {
  try {
    validateOptions(options);
    actionMethod(options.someOption);
    // Call action methods in sequence here.
  } catch (exception) {
    throw new Error(`[actionName] ${exception.message}`);
  }
};
```

Actions are designed to be read from the bottom up. At the bottom of our file, we export a function known as our handler. This function is responsible for calling to all of the other steps in our action. This helps us to accomplish a few things:

1. Centralize all of our calls to other code in one place.
2. Share response values from each step with other steps.
3. Clearly delineate the order of steps in our code.
4. Make our code more maintainable and extensible by avoiding nested spaghetti code.

Inside of this function, the very first thing we do is call to `validateOptions` passing in the assumed `options` argument passed to the handler function (or, what we export from our file as our action).

With `validateOptions` we start to see a few other sub-patterns of actions appear. Specifically, the name of the `validateOptions` function is called _exactly what it does_. It's not `vldOpts` or `validateOps` or anything that leaves room for confusion. If I were to drop another developer into this code, and ask them "what does that function do?" they'll most likely respond sarcastically with "uhh, validates the options?"

The next thing you'll notice is the structure of `validateOptions`. Immediately inside of the function body, a `try/catch` statement is added, with the `catch` portion taking the `exception` and `throw`ing it using the JavaScript `Error` constructor. Notice, too, that **when this error is thrown, we tell ourselves _exactly where the error is happening_** with `[actionName.validateOptions]` followed by the specific error message.

In the `try` block, we do what our code says: validate our `options`! The logic here is kept simple on purpose. If our action requires that `options` be passed and requires specific properties to be defined in those `options`, we throw an error if they don't exist. To make sure this clear, if we were to call this action now like this:

```javascript
actionName();
```

We'd get the following error in response:

```bash
[actionName.validateOptions] options object is required.
```

This is a _serious_ boon on development. We're telling ourselves exactly what we need up front so we can skip the "what did I forget to pass now?" roulette.

If we move back down to our handler function, we'll see that after our options have been validated with `validateOptions`, our next step is to call `actionMethod`, passing `options.someOptions`.

This is where we get into the actual steps or functionality of our action. Here, `actionMethod` takes in `options.someOption`. Notice that because it's the second step called in our handler, it's defined _above_ `validateOptions` (our first step).

If we look at `actionMethod` things should—purposefully—look pretty familiar. Here, we repeat the same pattern: give a clear name for our function, run our code in a `try/catch` block, and if our code fails, `throw` an error telling ourselves that it came from `[actionName.actionMethod]`.

### Refactoring our signup

Feeling undewherlmed? Great! That's what we're after. Writing clean code shouldn't be difficult or excessively esoteric. Now, let's start to refactor our signup endpoint into an action. Let's clean up our skeleton, adding some legitimate checks to `validateOptions`:

```javascript
const actionMethod = (someOption) => {
  try {
    console.log('Do something with someOption', someOption);
    // Perform a single step in your action here.
  } catch (exception) {
    throw new Error(`[signup.actionMethod] ${exception.message}`);
  }
};

const validateOptions = (options) => {
  try {
    if (!options) throw new Error('options object is required.');
    if (!options.body) throw new Error('options.body is required.');
    if (!options.body.email) throw new Error('options.body.email is required.');
    if (!options.body.password) throw new Error('options.body.password is required.');
    if (!options.body.profile) throw new Error('options.body.profile is required.');
    if (!options.response) throw new Error('options.response is required.');
  } catch (exception) {
    throw new Error(`[signup.validateOptions] ${exception.message}`);
  }
};

export default (options) => {
  try {
    validateOptions(options);
    // Call action methods in sequence here.
    options.response.end();
  } catch (exception) {
    throw new Error(`[signup] ${exception.message}`);
  }
};
```

A few things have changed. Notice that instead of `actionName`, we've now given our action a name: `signup`.

Inside of `validateOptions`, we've set some real expectations, too. Remember that in our original code, we reuse the `request.body` object several times. Here, we think ahead and make the assumption that we'll just pass the `body` part of the request (the only part we utilize). We also make sure to validate that each of the properties _of_ the body are present.

Finally, we also want to validate that the `response` object from our endpoint is passed so we can respond to the request within our action.

The details of this are mostly arbitrary; the point here is that we're ensuring we have what we need _before we put it to use_. This helps to eliminate the inevitable "did I pass that yet?" question as well as subsequent time wasted debugging to figure it out.

### Adding additional steps as functions

Now that we have our handler function set up as well as our `validateOptions`, we can start to port over the core functionality for our action.

```javascript
/* eslint-disable consistent-return */

import mongodb from '/path/to/mongodb';
import settings from '/path/to/settings';

const connectToMongoDB = () => {
  try {
    return new Promise((resolve, reject) => {
      mongodb.connect(
        settings.mongodb.url,
        (error, client) => {
          if (error) {
            reject(error);
          } else {
            const db = client.db('production');
            resolve({
              db,
              users: db.collection('users'),
              customers: db.collection('customers'),
            });
          }
        },
      );
    });
  } catch (exception) {
    throw new Error(`[signup.connectToMongoDB] ${exception.message}`);
  }
};

const validateOptions = (options) => [...];

export default async (options) => {
  try {
    validateOptions(options);
    const db = await connectToMongoDB();
  } catch (exception) {
    throw new Error(`[signup] ${exception.message}`);
  }
};
```

First, we need to establish a connection to our database. Recall, we need access to the `users` and `customers` collection from MongoDB. Knowing this, we can streamline our code by creating an action method `connectToMongoDB` whose sole job is connecting us to MongoDB, giving us access to the databases we'll need to do our work.

To do it, we wrap our call to `mongodb.connect` using the action method pattern. By wrapping this code with a JavaScript `Promise`, we can ensure our connection is complete _before_ we try to use it. This is necessary because we're no longer running our subsequent code accessing the database inside of `mongodb.connect`'s callback. Instead, we `resolve` our `Promise` passing the `db` connection along with the two databases that we'll need: `users` and `customers`.

Why is this important? Consider this: our connection to MongoDB could fail. If it does, we not only want to know why, but we want our code to be easily debugged. With nested spaghetti code, this is possible, but adds mental weight. 

By encapsulating our call—and any failures—inside of a single function, we eliminate the need to track down errors. This is especially helpful when the errors themselves are unhelpful or ambiguous (R.I.P to souls who get an `ECONNRESET`). The difference between `ERR ECONNRESET` and `[signup.connectToMongoDB] ERR ECONNRESET` is night and day. The error may not be clear, but we've told ourselves _exactly_ who's responsible.

Back in our handler function, we utilize the `async/await` syntax to ensure that we've received a response from MongoDB _before_ we continue with the rest of our action (i.e., we've achieved what our callback gave us without opening an Italian restaurant).

```javascript
/* eslint-disable consistent-return */

import mongodb from '/path/to/mongodb';
import settings from '/path/to/settings';

const createUser = (users, userToCreate) => {
  try {
    return new Promise((resolve, reject) => {
      users.insert(userToCreate, (error, insertedUser) => {
        if (error) {
          reject(error);
        } else {
          const [user] = insertedUser;
          resolve(user._id);
        }
      });
    });
  } catch (exception) {
    throw new Error(`[signup.createUser] ${exception.message}`);
  }
};

const connectToMongoDB = () => [...];

const validateOptions = (options) => [...];

export default async (options) => {
  try {
    validateOptions(options);

    const db = await connectToMongoDB();
    const userId = await createUser(db.users, options.body);
  } catch (exception) {
    throw new Error(`[signup] ${exception.message}`);
  }
};
```

Next up is creating our user. This is where the magic of actions start to show. Down in our handler function, we add our next step `createUser` beneath our first step `connectToMongoDB`. Notice that when we need to reference the value returned by a previous step in future steps, we give it a variable name that represents exactly what's being returned.

Here, `const db` suggests we get access to our database in that variable and `const userId` suggests we expect a user's `_id` back from `createUser`. In order to get there, we know that we need to connect to the `users` collection in MongoDB and we need the user information passed in the `request.body` to create that user. To do it, we just pass those values as arguments to `createUser`. Clean and tidy.

```javascript
const createUser = (users, userToCreate) => {
  try {
    return new Promise((resolve, reject) => {
      users.insert(userToCreate, (error, insertedUser) => {
        if (error) {
          reject(error);
        } else {
          const [user] = insertedUser;
          resolve(user._id);
        }
      });
    });
  } catch (exception) {
    throw new Error(`[signup.createUser] ${exception.message}`);
  }
};
```

Focusing just on our `createUser` definition, we can see that we take in that `db.users` argument as `users` and `options.body` as `userToCreate` (remember, this should be an `Object` with `email`, `password,` and `profile` as properties).

Using the same `Promise` approach, we call to `users.insert` and rely on our `resolve` and `reject` to handle the respective error and success states of our call to `users.insert`. If our insert is successful, we get the `_id` of the `insertedUser` and `resolve()` our `Promise` with it.

Pay close attention. Because we're calling `resolve(user._id)`, this means that back in our `handler` function, our `const userId = createUser()` is now "truthful" because once that `Promise` resolves, we'll get the `userId` in return, assigned to that variable. Sweet!

### Completing our action

At this point, we're familiar with the core concepts of an action. Once the full conversion is complete, here's what we get:

```javascript
import mongodb from '/path/to/mongodb';
import settings from '/path/to/settings';
import stripe from '/path/to/stripe/api';
import imaginaryEmailService from '/path/to/imaginaryEmailService';
import slackLog from '/path/to/slackLog';

const logCustomerOnSlack = (emailAddress) => {
  try {
    slackLog.success({
      message: 'New Customer',
      metadata: {
        emailAddress,
      },
    });
  } catch (exception) {
    throw new Error(`[signup.logCustomerOnSlack] ${exception.message}`);
  }
};

const sendWelcomeEmail = (to) => {
  try {
    return imaginaryEmailService.send({ to, template: 'welcome' });
  } catch (exception) {
    throw new Error(`[signup.sendWelcomeEmail] ${exception.message}`);
  }
};

const createCustomer = (customers, userId, stripeCustomerId) => {
  try {
    return new Promise((resolve, reject) => {
      customers.insert({ userId, stripeCustomerId }, (error, insertedCustomer) => {
        if (error) {
          reject(error);
        } else {
          const [customer] = insertedCustomer;
          resolve(customer._id);
        }
      });
    });
  } catch (exception) {
    throw new Error(`[signup.createCustomer] ${exception.message}`);
  }
};

const createCustomerOnStripe = (email) => {
  try {
    return stripe.customer.create({ email });
  } catch (exception) {
    throw new Error(`[signup.createCustomerOnStripe] ${exception.message}`);
  }
};

const createUser = (users, userToCreate) => {
  try {
    return new Promise((resolve, reject) => {
      users.insert(userToCreate, (error, insertedUser) => {
        if (error) {
          reject(error);
        } else {
          const [user] = insertedUser;
          resolve(user._id);
        }
      });
    });
  } catch (exception) {
    throw new Error(`[signup.createUser] ${exception.message}`);
  }
};

const connectToMongoDB = () => {
  try {
    return new Promise((resolve, reject) => {
      mongodb.connect(
        settings.mongodb.url,
        (error, client) => {
          if (error) {
            reject(error);
          } else {
            const db = client.db('production');
            resolve({
              db,
              users: db.collection('users'),
              customers: db.collection('customers'),
            });
          }
        },
      );
    });
  } catch (exception) {
    throw new Error(`[signup.connectToMongoDB] ${exception.message}`);
  }
};

const validateOptions = (options) => {
  try {
    if (!options) throw new Error('options object is required.');
    if (!options.body) throw new Error('options.body is required.');
    if (!options.body.email) throw new Error('options.body.email is required.');
    if (!options.body.password) throw new Error('options.body.password is required.');
    if (!options.body.profile) throw new Error('options.body.profile is required.');
    if (!options.response) throw new Error('options.response is required.');
  } catch (exception) {
    throw new Error(`[signup.validateOptions] ${exception.message}`);
  }
};

export default async (options) => {
  try {
    validateOptions(options);

    const db = await connectToMongoDB();
    const userId = await createUser(db.users, options.body);
    const customerOnStripe = await createCustomerOnStripe(options.body.email);

    await createCustomer(db.customers, userId, customerOnStripe.id);
    sendWelcomeEmail(options.body.email);
    logCustomerOnSlack(options.body.email);
  } catch (exception) {
    throw new Error(`[signup] ${exception.message}`);
  }
};
```

A few things to point out. First, all of our additional action methods have been added to our handler, called in sequence.

Notice that after we've created a customer on Stripe (and returned that customer as `const customerOnStripe`), none of the steps after this need a value from the previous steps. In turn, we just call these steps independently without storing their `return` value in a variable.

Notice, too, that our `sendWelcomeEmail` and `logCustomerOnSlack` steps remove the usage of an `await` because there's nothing for us to wait on.

That's it! At this point, we have a complete action.

### Wait, but why?

You're probably wondering "didn't we just add a ton of extra code to do the same thing?" We did. But something important to consider is how much context and clarity adding that (negligible amount of) extra code gave us.

This is the point of actions: giving us a consistent, predictable pattern for organizing complex processes. That's a mouthful, so another way to think about this is reducing maintenance cost. _Nobody_ likes to maintain code. Often, too, when we're tasked with maintaining a "legacy" codebase, it tends to look more  like the code we started with.

What this translates to is cost. Cost in time, money, and for the people doing the work: peace of mind. When code is a tangle of pasta, there's a cost to _understanding that code_. The less structure and consistency, the higher that cost.

With actions, we can significantly reduce the amount of thinking that goes into maintaining our code. Not only that, but we also make it incredibly easy to extend our code. For example, if we're asked to add the ability to log the new user in our analytics system, there's little to no thought involved.

```javascript
[...]
import analytics from '/path/to/analytics';

const trackEventInAnalytics = (userId) => {
  try {
    return analytics.send(userId);
  } catch (exception) {
    throw new Error(`[signup.trackEventInAnalytics] ${exception.message}`);
  }
};

const logCustomerOnSlack = (emailAddress) => [...];

const sendWelcomeEmail = (to) => [...];

const createCustomer = (customers, userId, stripeCustomerId) => [...];

const createCustomerOnStripe = (email) => [...];

const createUser = (users, userToCreate) => [...];

const connectToMongoDB = () => [...];

const validateOptions = (options) => [...];

export default async (options) => {
  try {
    validateOptions(options);

    const db = await connectToMongoDB();
    const userId = await createUser(db.users, options.body);
    const customerOnStripe = await createCustomerOnStripe(options.body.email);

    await createCustomer(db.customers, userId, customerOnStripe.id);
    sendWelcomeEmail(options.body.email);
    logCustomerOnSlack(options.body.email);
    trackEventInAnalytics(userId);
  } catch (exception) {
    throw new Error(`[signup] ${exception.message}`);
  }
};
```

This means that instead of wasting your own time and energy, you can implement features and fix bugs with very little stress. The end result is a happier you and happier stakeholders. Good deal, right?

While it's a minor detail, just so it's clear, let's look at how we actually _use_ our action back in our API:

```javascript
import signup from '/path/to/signup/action';

export default {
  v1: {
    '/users/signup': (request, response) => {
      return signup({ body: request.body, response });
    },  
  },
};
```

This would be an appropriate time for a Bill Cosby "puddin' face" GIF, but, well...you know.

### Testing our action

The final "wow" of actions is how easy they are to test. Because the code is already in steps, an action tells us what we need to test. Assuming we've mocked the functions in use inside of our action (e.g., `stripe.customers.create`) an integration test for our action might look like this:

```javascript
import signup from '/path/to/signup/action';
import stripe from '/path/to/stripe';
import slackLog from '/path/to/slackLog';

const testUser = {
  email: 'test@test.com',
  password: 'password',
  profile: { name: 'Test User' },
};

describe('signup.js', () => {
  beforeEach(() => {
    stripe.customers.create.mockReset();
    stripe.customers.create.mockImplementation(() => 'user123');

    slackLog.success.mockReset();
    slackLog.success.mockImplementation();
  });

  test('creates a customer on stripe', () => {
    signup({ body: testUser });
    expect(stripe.customers.create).toHaveBeenCalledTimes(1);
    expect(stripe.customers.create).toHaveBeenCalledWith({ email: testUser.email });
  });

  test('logs the new customer on slack', () => {
    signup({ body: testUser });
    expect(slackLog.success).toHaveBeenCalledTimes(1);
    expect(slackLog.success).toHaveBeenCalledWith({
      message: 'New Customer',
      metadata: {
        emailAddress: testUser.email,
      },
    });
  });
});
```

Here, each test represents a verification that the step in our action completed as expected. Because we only care that our action performed the steps, our test suite is dirt simple. All we need to do is make a call to our action with some input (in this case, we pass a `testUser` object as the `options.body` value in our action).

Next, we verify that our steps complete. Here, we verify that given a user with an email `test@test.com`, our action calls to `stripe.customers.create` passing that same email. Similarly, we test to see of our `slackLog.success` method was called, passing the message we'd like to see in our logs.

There's ample nuance with testing, of course, but hopefully the point here is clear: we have a very tidy chunk of code that's remarkably easy to test. No confusion. No time wasted "figuring it out." The only true cost would be the time mocking out the code called by our action if we hadn't done that already.

### Wrapping Up

So there you have it! Actions are a wonderful way to clean up your codebase, make things more predictable, and save yourself a ton of time in the process. 

Because actions are just a JavaScript pattern, the cost to test them out in your own app is zero. Try it, see if you like it. Most importantly: see if they improve the quality of your code. If you're struggling to write code that performs predictably, give this pattern a try. You won't regret it.
