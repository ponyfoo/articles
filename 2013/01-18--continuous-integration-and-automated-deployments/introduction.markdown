# Continuous Integration

Configuring [Travis CI](https://travis-ci.org/bevacqua/ponyfoo "ponyfoo build status on Travis CI") was tricky to get right. I configured my [.travis.yml](http://about.travis-ci.org/docs/user/build-configuration/ "Configuring your Travis CI build") file like this:

```
language: node_js

node_js:
  - 0.8

script:
  - "npm install"
  - "cd src"
  - "make test"
```

This simple configuration just makes sure the server runs on **Node.JS 0.8.x**, and then it runs a little script, where all [npm](https://npmjs.org/ "Node Packaged Modules") packages are installed, and then it runs any unit tests I might have written, making use of a _Makefile_.

_I didn't actually write any tests just yet_, but I will once I get more comfortable with **Node.JS**, and until then, this will come in handy anyways when I forget to include a package in my JSON config.
