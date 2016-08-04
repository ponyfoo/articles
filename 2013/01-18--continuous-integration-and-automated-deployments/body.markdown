# Automated Deployments

[Heroku](http://www.heroku.com/ "Heroku Cloud Application Platform") is an scalable web platform. You can get set up in _a few seconds_, and all you really need to do to configure your application is create what they call a [Procfile](https://devcenter.heroku.com/articles/procfile "Heroku Documentation"), mine is simply:

    web: node src/server.js
    
This configures my **Heroku** application to host a _web process_ and employ `src/server.js` as the **Node.JS** web server. I followed [this guide](https://devcenter.heroku.com/articles/nodejs "Getting Started with Node.JS on Heroku") to great success. And now, _whenever I want to deploy_, I can just issue the following _git command_:

```bash
$ git push heroku master
```

That's it. I might even configure a _staging environment_, which would be an exact replica of my production environment (hey, **it's _free_!**), except it would have dummy data, and it would be useful to prevent deploying faulty builds directly into production.

To manage multiple environments, [environment configuration variables](https://devcenter.heroku.com/articles/config-vars "Heroku Configuration") come in handy, to store secret values such as your session secret, your database connection string, and particularly, the environment name (such as development, staging, or production).

Having a one-step build process is crucial to minimizing room for mistakes during manual deploys, and mandatory if you are serious about protecting your production environment from development bugs. Manual deploys _can lead to mistakes_ like forgetting to copy a file, or replace a configuration value, or many other mistakes, reasons why you should be doing automated builds if you aren't already.
