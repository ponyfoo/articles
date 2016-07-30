By now you must've read [how I use `tape` for all the things][1], or how others [do as well][2]. Am I the only one here amused that we wrote _almost identical articles_ about Tape, and published them **on the same day**? We've already covered why Tape is better than Mocha et al. It's modular, it's sensical, and it [produces TAP output][3].

When it comes to server-side code, consider a module like this:
    
    function greet (name) {
      return 'hi ' + name;
    }
    module.exports = greet;
    

A test written in `tape` might look like:
    
    var test = require('tape');
    var greet = require('../greet');
    
    test('greeting produces a salutation', function (t) {
      t.equal(greet('stranger'), 'hi stranger', 'greeting is proper and amicable');
      t.end();
    });
    

Running the above test might involve adding a `test` script to your `scripts` in the `package.json` manifest, like below.
    
    {
      "scripts": {
        "test": "node test/*.js"
      }
    }
    

I cheated, and that piece of code will run any test files inside the `test` directory. To run that, simply use the `npm run` alias for `test`.
    
    npm test
    

How do we deal with the DOM, continuous testing during development hours, and continuous integration on remote server? For the answers, we might want to go back to the seemingly innocuous [TAP output][3] from `tape`.

[1]: /articles/testing-javascript-modules-with-tape
[2]: https://medium.com/javascript-scene/why-i-use-tape-instead-of-mocha-so-should-you-6aa105d8eaf4
[3]: https://testanything.org/
