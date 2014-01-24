# Continuous Integration, Automated Deployments #

In the [next post](/2013/01/18/asset-management-in-node "Asset management in Node") I'll get back to the meat of how I'm making progress with the blog application. This time however, I wanted to deviate a little, and talk about [Continuous Integration](https://travis-ci.org/ "Travis CI"), and **automated build processes**, the like of which you can achieve with **PaaS services** (also known as [Platform as a Service](http://en.wikipedia.org/wiki/Platform_as_a_service)), such as [AppHarbor](https://appharbor.com/ ".NET Cloud PaaS"), [Heroku](http://www.heroku.com/ "Cloud Application Platform"), and many others.

## Continuous Integration ##

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

## Automated Deployments ##

[Heroku](http://www.heroku.com/ "Heroku Cloud Application Platform") is an scalable web platform. You can get set up in _a few seconds_, and all you really need to do to configure your application is create what they call a [Procfile](https://devcenter.heroku.com/articles/procfile "Heroku Documentation"), mine is simply:

    web: node src/server.js

This configures my **Heroku** application to host a _web process_ and employ `src/server.js` as the **Node.JS** web server. I followed [this guide](https://devcenter.heroku.com/articles/nodejs "Getting Started with Node.JS on Heroku") to great success. And now, _whenever I want to deploy_, I can just issue the following _git command_:

```bash
$ git push heroku master
```

That's it. I might even configure a _staging environment_, which would be an exact replica of my production environment (hey, **it's _free_!**), except it would have dummy data, and it would be useful to prevent deploying faulty builds directly into production.

To manage multiple environments, [environment configuration variables](https://devcenter.heroku.com/articles/config-vars "Heroku Configuration") come in handy, to store secret values such as your session secret, your database connection string, and particularly, the environment name (such as development, staging, or production).

Having a one-step build process is crucial to minimizing room for mistakes during manual deploys, and mandatory if you are serious about protecting your production environment from development bugs. Manual deploys _can lead to mistakes_ like forgetting to copy a file, or replace a configuration value, or many other mistakes, reasons why you should be doing automated builds if you aren't already.