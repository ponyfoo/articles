# The API

The API in question can be found below. The `saveProducts` method would `GET` the JSON description for both products in series, and then `POST` data about the products to a user's shopping cart.

```js
saveProducts(function* () {
  yield '/products/javascript-application-design';
  yield '/products/barbie-doll';
  return '/cart';
});
```

In addition, I thought it'd be nice if `saveProducts` also returned a `Promise`, meaning you could [chain some other operations][2] to be executed after the products had been saved to the cart.

```js
saveProducts(productList)
  .then(data => console.log('Saved', data));
```

Naturally, some conditional logic would allow this hypothetical method to save the products to a wish list instead of onto the shopping cart.

```js
saveProducts(function* () {
  yield '/products/javascript-application-design';
  yield '/products/barbie-doll';
  if (addToCart) {
    return '/cart';
  }
  return '/wishlists/nerd-items';
});
```

This example could also apply to the server side, where each yielded value could result in a database query and the returned value could also indicate what kind of object we want to save back to the database. Similarly, the iterator can decide the pace at which yielded inputs are processed: it could be as simple as a synchronous queue, process all queries in parallel, or maybe use a concurrent queue with limited concurrency. Regardless, the API can stay more or less the same _(depending on whether consumers expect to be able to use the product data in the generator itself or not)_.

# Implementing `saveProducts`

First off, the method in question takes in a generator and initializes a generator object to iterate over the values produced by the generator function.

```js
function saveProducts (productList) {
  var g = productList();
}
```

In a naïve implementation, we could pull each product one by one in an asynchronous series pattern. In the piece of code below, I'm using `fetch` to pull the resources yielded by the user-provided generator _-- as JSON_.

```js
function saveProducts (productList) {
  var g = productList();
  var <mark>item = g.next()</mark>;
  <mark>more()</mark>;
  function more () {
    if (item.done) {
      return;
    }
    fetch(item.value)
      .then(res => res.json())
      .then(product => {
        <mark>item = g.next(product)</mark>;
        <mark>more()</mark>;
      });
  }
}
```

> By calling `g.next(<mark>product</mark>)` we're allowing the consumer to read product data by doing `data = yield '/resource'`.

So far we're pulling all data and passing it back, an item at a time to the generator, which has a synchronous feel to it. In order to leverage the `return` statement, we'll save the products in a temporary array and then `POST` them back when we're done iterating.

```js
function saveProducts (productList) {
  var <mark>products = []</mark>;
  var g = productList();
  var item = g.next();
  more();
  function more () {
    if (item.done) {
      <mark>save(item.value)</mark>;
    } else {
      details(item.value);
    }
  }
  function details (endpoint) {
    fetch(endpoint)
      .then(res => res.json())
      .then(product => {
        <mark>products.push(product)</mark>;
        item = g.next(product);
        more();
      });
  }
  function save (endpoint) {
    fetch(endpoint, {
      method: '<mark>POST</mark>',
      body: JSON.stringify({ products })
    });
  }
}
```

At this point product descriptions are being pulled down, cached in the `products` array, forwarded to the generator body, and eventually saved in one fell swoop using the endpoint provided by the `return` statement. Where are the promises? Those are very simple to add: `fetch` returns a `Promise`, and it's `return` all the way down.

```js
function saveProducts (productList) {
  var products = [];
  var g = productList();
  var item = g.next();
  <mark>return</mark> more();
  function more () {
    if (item.done) {
      <mark>return</mark> save(item.value);
    }
    <mark>return</mark> details(item.value);
  }
  function details (endpoint) {
    <mark>return</mark> fetch(endpoint)
      .then(res => res.json())
      .then(product => {
        products.push(product);
        item = g.next(product);
        return more();
      });
  }
  function save (endpoint) {
    <mark>return</mark> fetch(endpoint, {
        method: 'POST',
        body: JSON.stringify({ products })
      })
      <mark>.then(res => res.json())</mark>;
  }
}
```

> We're also casting the `save` operation's response as JSON, so that promises chained onto `saveProducts` can leverage response `data`.

As you may notice the implementation doesn't hardcode any important aspects of the operation, which means you could use something like this pretty generically, as long as you have zero or more inputs you want to pipe into one output. The consumer ends up with an elegant-looking method that's easy to understand -- they `yield` input stores and `return` an output store. Furthermore, our use of promises makes it easy to concatenate this operation with others. This way, we're keeping a potential tangle of conditional statements and flow control mechanisms in check, by abstracting away flow control into the iteration mechanism under the `saveProducts` method.

[1]: /articles/es6-generators-in-depth "ES6 Generators in Depth on Pony Foo"
[2]: /articles/es6-promises-in-depth "ES6 Promises in Depth on Pony Foo"
