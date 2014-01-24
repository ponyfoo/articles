# Deploying Node apps to AWS using Grunt

![amazon-web-services.png][1]

The full title should've been something like: _Deploying Node applications to [Amazon Web Services (AWS)](http://aws.amazon.com/ "Amazon Web Services") through their [CLI tools](https://github.com/aws/aws-cli "aws/aws-cli on GitHub") using Grunt_. That was way too long, though.

To summarize, I've been toying with **AWS** for a few days now, and I wanted to share my experience and my approach with you. My goal was to set up a deploy flow in Grunt to enable me to spin up new [EC2](http://aws.amazon.com/ec2 "Elastic Cloud Compute") instances in the AWS cloud and deploy to them easily. In this post I'll look at being able to do all of the following in my command-line:

- Creating [SSH](http://en.wikipedia.org/wiki/Secure_Shell#Version_2.x "Secure Shell 2") key-pairs and uploading their public key to AWS
- Launching EC2 _micro_ instance with an [_Ubuntu AMI_](http://cloud-images.ubuntu.com/releases/raring/release-20130423/ "Ubuntu 13.04 (Raring Ringtail)")
- `ssh` into the instance using the private key associated with an instance, and get it set up for future deploys:
  - Install **Node.js**, duh
  - Install [pm2](https://github.com/Unitech/pm2 "pm2 on GitHub") to keep our application alive
- Shut down instances with the same ease as we can launch them, deleting key-pairs
- Get a list of EC2 instances
- Get a command to SSH myself to an instance, in case I want to do some heavy lifting myself
- Deploy to our instance using `rsync`, for [blisteringly fast data transfers](https://help.ubuntu.com/community/rsync "rsync, an article by the Ubuntu Community"), then:
  - Copy the files somewhere else
  - Install npm packages using `npm install --production`
  - Restart our Node application using `pm2 reload all`
- Refer to instances by a name tag like `'bambi'`, rather than an ID such as `i-he11f00ba4`

If this sounds like something you'd be interested in learning about, read on.

  [1]: http://i.imgur.com/Yya9AIy.png "Amazon Web Services"

##### What is missing in this installment?

- In the future, I'd also like to install [nginx](http://nginx.com/ "nginx HTTP proxy server")
- I'm not creating individual users for `rsync`, `node`, as I'm not entirely sure of how to manage that
- I'm not setting up port forwarding, not giving my instance a public IP
- I didn't establish a sensible way to swap configuration based on the environment, yet

**Yet...** This is a _work in progress_, and I expect to do all of the above, soon. I'll write a follow-up article to keep you updated.

## Introduction

In the past, I've been using [Heroku](https://www.heroku.com/ "Heroku Cloud Application Platform") to host my Node applications, _(this blog is hosted on Heroku)_. For convenience, and because I didn't really know how to interact with AWS, I'd never even tried it before. Now that **I'm writing a book on build processes and architecture for Node applications**, I deemed it necessary to teach myself _how to use AWS directly_, rather than through a [PaaS](http://en.wikipedia.org/wiki/Platform_as_a_service "Platform as a Service") provider such as Heroku.

However, I didn't want to lose the ability to deploy from my command-line. In fact, I wanted to do one better, and become able to spin up new environments with just a [Grunt](http://gruntjs.com/ "Grunt.js JavaScript Task Runner") command. A little while ago, Amazon [released version 1.0.0](https://github.com/aws/aws-cli/releases/tag/1.0.0 "aws-cli Release 1.0.0 on GitHub") of their CLI tools, my understanding: a huge improvement over their previous command-line tooling, but I never really used their previous version before.

Some of the tasks took a little iterating to get right, but considering AWS gives you free access to their services for a year when you first sign up, this wasn't really an issue. Keeping in mind the fact that _these are automated tasks_, the goal for me was to get it right once, and then be _able to re-use it in any other projects I might want to host on the  AWS platform in the future.

## Requirements

I started off by reading up on the requirements for the **aws-cli** tool. I wanted to do as little manual labor as possible, after researching a little I figured out my requirements to use the `aws` CLI tool unobtrusively.

- Have an **AWS** account
- Get an Access Key, write down `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
- Pick a default region for `AWS_DEFAULT_REGION`
- Install `pip`, the Python [package manager](http://www.pip-installer.org/en/latest/installing.html "pip installation instructions"), if it's not present
- Use it to install the CLI: `pip install awscli --upgrade`

I actually created a Grunt task to install the `aws` CLI, but that's not required. Next, the CLI requires you to provide some _environment variables_. Luckily, I can provide an `env` programmatically to [processes I spawn](http://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback "Child Process Node.js Documentation") with Node, so I'll do that instead of touching [environment variables](http://en.wikipedia.org/wiki/Environment_variable "Environment Variables on Wikipedia").

Let's first look at the JSON file I set up to deal with all my AWS-related "environment" variables.

```json
{
    "AWS_ACCESS_KEY_ID": "redacted",
    "AWS_SECRET_ACCESS_KEY": "redacted",
    "AWS_DEFAULT_REGION": "us-east-1",
    "AWS_IMAGE_ID": "ami-c30360aa",
    "AWS_INSTANCE_TYPE": "t1.micro",
    "AWS_SECURITY_GROUP_NAME": "standard",
    "AWS_SSH_USER": "ubuntu",
    "AWS_RSYNC_USER": "ubuntu"
}
```

The one thing I _didn't automate_ was **security group creation**, plainly because this was something I figured you rarely have to deal with, and it's one of the first steps you have to take, so I was eager to do other stuff, rather than deal with security groups. It's no mystery either, all you have to do is enable ports 22, for **SSH**, and 80, for **HTTP**.

The `t1.micro` instance type is the one to use if you want to be eligible for the free tier benefits (that reads: if you don't want to pay up). Ideally, the `rsync` user should be a different one from the default SSH user, but like most things, it's an easily changeable setting that I didn't want to waste a bunch of time on. Take into account any user other than `ubuntu` would need to be created the first time we `ssh` into the instance, so that it's available to future sessions.

## Executing commands against the AWS API

I won't be boring you with the details of how I got to the current state of affairs, and I'll jump to the latest versions of my code, instead. We've barely touched the surface now, though. The first thing to set up is a reusable piece of code that'll make running commands, as if we were in a terminal, pretty easy. This module is used pretty much everywhere else in the task suite I developed to interact with **AWS**.

Keep in mind all of this code is meant to be run in development machines, with `grunt` available to them, and these tasks will be executed from the command-line directly. Thus, throwing exceptions with `grunt.fatal` is the correct approach, rather than using `function(err, result)` type callbacks.

```js
'use strict';

var grunt = require('grunt');
var util = require('util');
var exec = require('child_process').exec;

module.exports = function(command, args, done, print){
    args.unshift(command);

    var cmd = util.format.apply(util, args);

    grunt.verbose.writeln(cmd);

    exec(cmd, { env: conf() }, callback);

    function callback (err, stdout, stderr) {
        if (err) { grunt.fatal(err); }
        if (stderr) { grunt.fatal(stderr); }

        if (print !== false) {
            grunt.log.writeln(stdout);
            done();
        } else {
            done(stdout);
        }
    }
};
```

This module quite simply allows me to execute commands with a very simple API, and then either print them to `stdout` or fiddle with the data. Whenever there's a hiccup, I'll throw and halt execution. This is important because I don't want to continue randomly throwing calls into a sensitive API, that might even end up costing me money, because I didn't preemptively end the process.

At this point you might have realized this doesn't even compile.

> What the hell is `conf()`?

Yeah, about that. `conf` is a [global variable](http://nodejs.org/api/globals.html "Node Globals"), and it was set by the line below, _somewhere else_.

```js
global.conf = module.exports = nconf.get.bind(nconf);
```

The [nconf](https://github.com/flatiron/nconf "flatiron/nconf on GitHub") module is a nice little utility that allows you to keep your environment configuration in one place. This is particularly convenient for our use case, where I configured `nconf` more or less like this:

```js
'use strict';

var nconf = require('nconf');
var path = require('path');
var aws = path.join(__dirname, 'private/aws.json');

nconf.argv();
nconf.env();
nconf.file('aws', aws);

global.conf = module.exports = nconf.get.bind(nconf);
```

I believe **this is the one and only respectable use of a global variable in Node**, and I don't use them for anything else, _and neither should you_.

Fine, that was a lot of code and we're still not interacting with **AWS** any more than we were five minutes ago. Fair enough, here's a `grunt` task you could use immediately:

```js
'use strict'

var exec = require('./exec.js');

module.exports = function (grunt) {
    grunt.registerTask('ec2_lookup', function(name){
        exec('aws ec2 describe-instances --filters Name=tag:Name,Values=%s', [name], this.async());
    });
};
```

Sweet, huh? That's really concise! Now we can run `grunt ec2_lookup:foobar` and get some JSON describing the metadata for an EC2 instance tagged with the name `foobar`. That's nice syntactic sugar, but barely any better than using the `aws` CLI tool directly. Well, the power of this approach reveals itself when we start combining different commands in new and useful ways. Let's start heading in the direction of launching a new EC2 instance. That's pretty easy to do using their [web interface](https://console.aws.amazon.com/console/home "Amazon Web Services Console Home"), but we can spend a bunch of time typing away at our terminal and it'll be pretty hard to get it right, besides: we came for the automation.

Here's a similar task in action, which allows us to look up instances based on their state:

![ec2-list.png][1]

## Launching an EC2 instance

The first step we have to take for launching an EC2 instance, is generating an SSH key-pair so that we can access it from the command-line. Then we have to actually run the instance, providing a bunch of parameters to the CLI. Our launch task looks pretty simple:

```js
'use strict';

module.exports = function(grunt){

    grunt.registerTask('ec2_launch', function(name){

        grunt.task.run([
            'ec2_create_keypair:' + name,
            'ec2_run_instance:' + name,
            'ec2_setup:' + name
        ]);
    });
};
```

We'll get into the `ec2_setup` task later on. For now, let's focus on creating a key pair.

## The `ec2_create_keypair` Task

There are two different ways you can do this for AWS EC2 instances. You could either [use the CLI to create one](http://docs.aws.amazon.com/cli/latest/userguide/cli-ec2-keypairs.html "Using Key Pairs on AWS CLI documentation"), or you could generate one yourself, using `ssh-keygen` and then [importing it into AWS](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws "Generate your own key and import it to AWS"). In the beginning, I tried to use their CLI to create the key-pairs, but I ran into a few issues, and moved away from that approach, ultimately focusing on `ssh-keygen`, which was better anyways since the private key never travels over the network (in the other approach, _Amazon sends you the private key_).

So how do I generate the private key? Easy!

```js
exec('ssh-keygen -t rsa -b 2048 -N "" -f %s', [file], then);
```

Where file is the path where I want to persist my private key, a public key will also be created at the same place, with a `.pub` extension appended to it. Then, I give amazon the public key.

```js
exec('aws ec2 import-key-pair --public-key-material %s --key-name %s', [
    'file://' + file + '.pub', name
], done);
```

I'll admit, the `'file://'` thing is really weird, but I haven't really found any other solution, and [this one worked](https://github.com/aws/aws-cli/issues/41 "Related issue on GitHub"), so I stick to it. Let me know if you've found any alternatives to that. The `name` variable is one used in almost every one of these **Grunt** tasks I've implemented, and it's the tag I use to identify instances, as we'll see shortly. The public key also gets uploaded to **AWS** with that name.

#### A matching `ec2_delete_keypair`

This task reverts what was done in `ec2_create_keypair`, and it also takes a `name` argument. We simply execute the [delete-key-pair](http://docs.aws.amazon.com/cli/latest/reference/ec2/delete-key-pair.html "delete-key-pair on AWS CLI documentation") command, and then physically delete the key-pair from our machine.

```js
exec('aws ec2 delete-key-pair --key-name %s', [name], removeFromDisk);
```

If you're reading this, _I'll go ahead and assume_ you know how to delete files using Node.

## Spinning up an instance in `ec2_run_instance`

This one is both one of the easiest tasks to put together, and the easiest to drastically fuck up when running, due to its sensitivity in that it affects our billing.

```js
exec('aws ec2 run-instances --image-id %s --instance-type %s --count %s --key-name %s --security-groups %s', [
    conf('AWS_IMAGE_ID'), conf('AWS_INSTANCE_TYPE'), 1, name, conf('AWS_SECURITY_GROUP_NAME')
], createTag, false);
```

Nothing we haven't really seen before. We're creating an instance that is:

- Just one instance
- Based on the [AMI](https://aws.amazon.com/amis "Amazon Machine Images"), `"ami-c30360aa"`, as configured in `aws.json`
- Of the `"t1.micro"` [instance type](http://aws.amazon.com/ec2/instance-types "Amazon EC2 Instance Types")
- In the security group we've previously created for all of our instances, `"standard"` in my case
- In the region we chose as default, `"us-east-1"`
- Configured to use the SSH key with the name we're provided

Wow, that's a lot for just one command. Yeah, but what's next? We're not done. We need to create a tag for this instance, so that we can very easily refer to it in all other Grunt tasks we have. Below is what the `createTag` callback is doing.

```js
function createTag (stdout) {
    var result = JSON.parse(stdout);
    var id = result.Instances[0].InstanceId;
    var task = util.format('ec2_create_tag:%s:%s', id, name);

    grunt.task.run(task);
    done();
}
```

Nothing that amusing, I'm getting the `id`, putting it together with the `name` we want, and passing those along to the `ec2_create_tag` task. That task will create a tag on our instance, like so:

```js
exec('aws ec2 create-tags --resources %s --tags Key=Name,Value=%s', [id, name], done);
```

Phew, that was a lot of work. However, we're now able to spin as many instances as we want from our command-line, how cool is that?!

#### Shutting instances down

Before proceeding, though, I'll share what `ec2_shutdown` does for us: it'll delete the key-pair with the task we talked about earlier, after [terminating the instance](http://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html "Terminate Instances with AWS CLI") through **AWS CLI**.

```js
exec('aws ec2 terminate-instances --instance-ids %s', [id], done);
```

One caveat you might've realized, is that this command uses the instance ID, which we don't want to provide by hand, since that's messy. Remember the first task we wrote to find instances tagged with a name? Well, that's what we're using to find the `InstanceId`!

# To `ssh`, and beyond!

After our instance is up, and tagged, we have to wait a bit before we can access its DNS, and then we have to wait a bit more to be able to make an SSH connection. I didn't want to wait myself, so I wrote a task to wait for me.

```js
function waitForDNS () {
    getCredentials(name, function (c) {
        if (!c) {
            grunt.log.writeln('DNS still cold, retrying...');
            wait(waitForDNS);
            return;
        }
        grunt.log.ok('Instance accessible at: %s', c.host);
        waitForSSH();
    });
}

function waitForSSH () {
    grunt.log.writeln('attempting to connect...');
    var connection = ssh([], name, function(){}, false);

    connection.on('error', function () {
        grunt.log.writeln('connection refused, retrying');
        wait(waitForSSH);
    });

    connection.on('ready', function () {
        grunt.log.ok('success, proceeding in 10s');
        wait(done, 10); // apt-get failed if we didn't wait a bit.
    });
}
waitForDNS(); // first attempt
```

Let me explain the two functions that are not described here, `getCredentials` and `ssh`. The former just uses a command similar to our `ec2_lookup` from earlier, and checks whether the public DNS field is set. The latter is a little more complicated, it queues commands to be executed over SSH, but since we're passing an empty array of commands, we're just testing if it can make a connection. When we're done, we wait a few seconds for good measure, because otherwise the system might not be _entirely_ operational. Before adding that 10 second wait there, my following task was consistently failing to install anything using [apt-get](https://help.ubuntu.com/community/AptGet/Howto "Package Management with APT"). After the wait, the problem evaporated.

## Installing things in our new instance!

Now that the wait is finally over, we can start using our instance. Yeah, about that... we need to install things first, it's a brand new machine! _(well, sort of)_

Using the same `ssh` queueing mechanism I talked about in the last section _(fine! I'll show it to you in a minute, hang on)_, I set up a bunch of things on the new box, over SSH.

```js
var project = conf('PROJECT_ID');
var tasks = [[ // rsync
    util.format('sudo mkdir -p /srv/rsync/%s/latest', project),
    util.format('sudo mkdir -p /srv/apps/%s/v', project),
    util.format('sudo chown ubuntu /srv/rsync/%s/latest', project)
], [ // node.js
    'sudo apt-get install python-software-properties',
    'sudo add-apt-repository ppa:chris-lea/node.js -y',
    'sudo apt-get update',
    'sudo apt-get install nodejs -y'
], [ // pm2
    'sudo apt-get install make g++ -y',
    'sudo npm install -g pm2',
    'sudo pm2 startup'
]];

var commands = _.flatten(tasks);
ssh(commands, name, done);
```

All I'm really doing is installing `node`, `npm`, and `pm2`. After this, all that remains is deploying through `rsync`, and using `pm2` to load our changes on each deploy. Let's stand back for a minute and look at the `ssh.js` module I wrote.

```js
var Connection = require('ssh2');

function(commands, name, done){
    var c = new Connection();

    c.on('ready', next);
    c.on('close', done);
    c.on('error', grunt.fatal);

    getCredentials(name, c.connect);

    function next () {
        if (commands.length === 0) {
            done();
        } else {
            var command = commands.shift();

            grunt.log.writeln(command);

            c.exec(command, function (err, stream) {
                if (err) { grunt.fatal(err); }

                stream.on('data', function (data, extended) {
                    grunt.log.write(String(data));
                });

                stream.on('exit', function (code) {
                    if (code === 0) { next(); }
                    grunt.fatal(command, chalk.red('exit code ' + code));
                });
            });
        }
    }
    return c;
}
```

See? not that complicated, just some sort of asynchronous queue. A fancy way of running commands over SSH.

## Deploying with `rsync` and `pm2`

Lastly we'll use [rsync](https://help.ubuntu.com/community/rsync "rsync, an article by the Ubuntu Community") over SSH to push our current version to the **AWS EC2** instance. We'll then _copy_ with `cp` to a folder named using the current build version; we don't use `mv`: we want it to stay there to be able to exploit the benefits of `rsync` in the future. Then, we just update a symbolic link on the `current` folder, to make sure it maps to the last deployed version of our code, and lastly, we reload our `node` process using `pm2`. Done!

```js
exec('rsync -vaz --stats --progress --delete %s -e "ssh -o StrictHostKeyChecking=no -i %s" %s %s@%s:%s', [
    excludeFrom, c.privateKeyFile, local, user, c.host, remote
], deploy);

function deploy () {
    var dest = util.format('/srv/apps/%s/v/%s', project, v);
    var target = util.format('/srv/apps/%s/current', project);
    var commands = [
        util.format('sudo cp -r %s %s', remoteSync, dest),
        util.format('sudo npm --prefix %s install --production', dest),
        util.format('sudo ln -sfn %s %s', dest, target), [
            util.format('sudo pm2 start %s/%s -i 2 --name %s', target, conf('NODE_SCRIPT'), name),
            'sudo pm2 reload all'
        ].join(' || ') // start or reload
    ];
    ssh(commands, name, done);
}
```

This works the same for the first deploy, where it'll short-circuit on `pm2 start /srv/apps/example/current/app.js -i 2 --name staging`, rather than reloading.

# Conclusions and a Repository

I polished what I wrote about in the article and separated it from my original code-base, making it into [a reusable grunt plugin I called grunt-ec2](https://github.com/bevacqua/grunt-ec2 "grunt-ec2 on GitHub"), it's pretty easy to set up, and it doesn't pollute the global namespace or any of that, you simply have to pass a few configuration variables through Grunt as usual. You can read it's documentation on GitHub.

I also added some pretty screenshots I took for that repository.

Lastly, please **let me know if this article sounded like a lot of gibberish glued together**, or if you _actually learned anything_ from it.

  [1]: http://i.imgur.com/ecFsa4b.png