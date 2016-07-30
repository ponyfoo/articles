### Log **exceptions** with their _full stack traces_ ###

This one is _obvious enough_. You have to watch out for exceptions getting lost in the sea of asynchronous code. This basically means figuring out whether a piece of code _`throw`s exceptions or uses the callback convention_.

```js
// the usual, sync way
try{
   syncOperation(); 
}catch(e){
    // rethrow, or handle it
}

// the async way
asyncOperation(function(err, data){
    if(err){
        // bubble the exception up, or handle it
        return;
    }
});
```

Keep in mind another important aspect of exception logging, is doing so in a persistant way. That is, _use a database storage_ for your logging purposes. The `console` is _just fine_ for development environments, but you probably want something _more robust_ in production.

![logging.jpg][1]

If you are, however, hosting on a platform such as [heroku](https://www.heroku.com/ "Heroku Cloud Hosting"), where `console` output is persisted, then you can opt not to log to a database yourself.

Popular logging options for Node include [winston](https://github.com/flatiron/winston "flatiron/winston on GitHub") and [bunyan](https://github.com/trentm/node-bunyan "trentm/node-bunyan on GitHub"). Both support various logging adapters.

### Log `uncaughtException`, but _then exit_ ###

When an exception is not handled anywhere else, it will be emitted on the `process`'s `'uncaughtException'` event. We can listen for this event to do some logging, but we should allow the process to _shut down gracefully_.

```js
process.on('uncaughtException', function(err){
    // log the error
    process.exit(1);    
});
```

## Using Domains ##

Node has recently released the [Domain API](http://nodejs.org/api/domain.html "Domain - Node Docs"), which  provides _a context_ in which we can deal with uncaught exceptions.

It's hard to put it any better than what the _Node Docs_ have to offer:

> Domains provide a way to handle multiple different IO operations as a single group. If any of the event emitters or callbacks registered to a domain emit an `error` event, or throw an error, then the domain object will be notified, rather than losing the context of the error in the `process.on('uncaughtException')` handler, or causing the program to exit immediately with an error code.

## Failover Clusters ##

![cluster.jpg][2]

The [Cluster API](http://nodejs.org/api/cluster.html "Cluster - Node Docs") was published alongside `domain`. Clusters allow us to use several processes, taking advantage of _multi-core systems_. The usefulness of clusters lies in the _ability to listen to the same port using several processes_. This is provided by the API itself.

Here is a very unpolished example HTTP server, using clusters.

```js
var cluster = require('cluster');
var http = require('http');
var numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers.
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', function(worker, code, signal) {
    console.log('worker ' + worker.process.pid + ' died');
  });
} else {
  // Workers can share any TCP connection
  // In this case its a HTTP server
  http.createServer(function(req, res) {
    res.writeHead(200);
    res.end("hello world\n");
  }).listen(8000);
}
```

Obviously we'd like to _separate this_ into two files, the master cluster and the forks. A nice improvement over this design might be _forking the cluster whenever a worker crashes_. We would simply need to add `cluster.fork()` to the `exit` listener.

Node Docs explain:

> Because workers are all separate processes, they can be killed or re-spawned depending on your program's needs, without affecting other workers. As long as there are some workers still alive, the server will continue to accept connections. Node does not automatically manage the number of workers for you, however. It is your responsibility to manage the worker pool for your application's needs.

I **highly recommend** skimming _(at the very least)_ through both [domain](http://nodejs.org/api/domain.html "Domain - Node Docs") and [cluster](http://nodejs.org/api/cluster.html "Cluster - Node Docs") documentation pages, as they are really short and extremely valuable for whoever's interested in keeping their servers uptime at a respectable level.

## The Last Stand, Uptime Monitoring ##

Ultimately, even the master cluster _can_ fail. As a last resort, we might set up a process that monitors our application's port. We can determine a _finite number of states_ our application and port might be in.

- Server completely shut down. No port listener
- Server preparing to listen. No port listener
- Server listening

Armed with this knowledge, we could assert whether our application is down, starting, or up. We might set up a _monitoring process_, which would run _in parallel_ with our server process(es).

I created an `npm` package explicitly to deal with this kind of scenario. The [process-finder](https://github.com/bevacqua/process-finder "bevacqua/process-finder on GitHub") package helps us find processes listening on a port, and even more handily, it lets us watch the port for changes!

Here's a tentative `monitor.js` application.

```js
var finder = require('process-finder');
var port = 3000; // port to watch
var watcher = finder.watch(port);
var runner = require('./runner.js');

watcher.on('error', console.error);
watcher.on('listen', function(pid){
    console.log('Cluster Up!', pid);
});
watcher.on('unlisten', function(pid){
    console.log('Cluster Down!', pid);
    runner.start();
});

runner.start();
```

Where `runner.js` would simply [spawn](http://nodejs.org/api/child_process.html "Child Process - Node Docs") a new server. The spawned process would eventually listen in the application's port, and it might even use a **cluster**, as we previously discussed, to improve its resilience.

We got this far in _our efforts to reduce server downtime_, we should go the extra mile here. Most definitely, we would benefit from having our logger _send us an email_ whenever a worker dies, or at the very least, whenever the `monitor` has to restart our `cluster` because a previous cluster _went deaf_.

## Performance Analytics ##

Monitoring your application is great, but you'd probably like charts with that, too. You can use a tool such as [NodeFly](http://nodefly.com/ "NodeFly Monitoring Solution"), or [Nodetime](http://nodetime.com/ "Nodetime Performance Analytics") for this purpose.

These solutions allow you to track CPU usage, server load, database load, perform memory profiling, and _more_. Make sure to check them out. They also allow you to set up alerts when certain thresholds are surpassed.

#### Nodetime ####

Nodetime's [documentation](http://docs.nodetime.com/#alerts "Alerts - Nodetime Documentation") explains:

> It is important to be notified when an application is experiencing performance problems in order to prevent downtime and be able to quickly locate the problem's root cause, while profiling exact problem symptoms, which might disappear later. Nodetime allows users to create threshold and anomaly alerts for many internal metrics of the application - for example, if HTTP response time is continuously high or there are too few requests. It is also possible to set alerts on API call metrics of different supported libraries, such as MongoDB, Redis, and MySQL.

Both solutions are trivially easy to set up.

For [Nodetime](http://nodetime.com/ "Nodetime Performance Analytics"), all you need to do is the following:

#### [Sign Up](http://nodetime.com/signup "Sign up - Nodetime") ####

Sign up with them to get an API key.

#### Install ####

Install their `npm` module.

```bash
$ npm install nodetime --save
```

#### Setup ####

Load and configure the module using the API key linked to your account.

```js
require('nodetime').profile({
    accountKey: 'your_account_key', 
    appName: 'your_application_name'
});
```

And, _that's it!_

  [1]: https://i.imgur.com/gBJfpgY.jpg "Not the most useful kind of logging"
  [2]: https://i.imgur.com/qpxYf8O.jpg "Not quite, but better than nothing!"
