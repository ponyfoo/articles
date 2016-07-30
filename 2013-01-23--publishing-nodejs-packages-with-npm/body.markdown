The first thing you have to do is **identify yourself**, create an account on **npm**:

```bash
$ npm adduser
```

After that all you have to do is code up a [package.json](https://npmjs.org/doc/developers.html "package.json specs"), which essentially requires a package _name_ and a _version_ string.

Here's an example **package.json**, extracted from [jsn](https://github.com/bevacqua/jsn "JSN on GitHub"):

```json
{
  "name": "jsn",
  "description": "JavaScript Node server-side variable parser",
  "author": {
    "name": "Nicolas Bevacqua",
    "email": "nicolasbevacqua@gmail.com",
    "url": "http://www.ponyfoo.com"
  },
  "version": "0.0.3",
  "dependencies": {
  },
  "devDependencies": {
    "vows": "0.5.x",
    "mocha": "*",
    "should": "*"
  },
  "repository": {
    "type": "git",
    "url": "git://github.com/bevacqua/jsn.git"
  },
  "bugs": {
    "url": "https://github.com/bevacqua/jsn/issues"
  },
  "licenses": [{
    "type": "MIT"
  }],
  "main": "./src/compiler.js",
  "engines": {
    "node": ">= 0.8.x",
    "npm": ">= 1.1.x"
  }
}
```

The only thing that might be worth mentioning is that you should make sure to specify a `"main"` property, so that the module gets exported appropriately through _the entry point you intended_.

Always make sure to thoroughly test the logic in your packages before unleashing them into the wild. If you happen to publish a _faulty image_ of your package, you **can overwrite** it using the `--force` flag:

```bash
$ npm publish --force
```
    
In addition to **.gitignore**, **.npmignore** files can come in handy, here is an example, again from the **jsn** repository on _GitHub_.

```
*.md
.DS_Store
.git*
.idea
Makefile
docs/
examples/
support/
test/
```

Spread the word!
