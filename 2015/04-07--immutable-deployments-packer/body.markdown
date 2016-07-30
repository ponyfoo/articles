# Motivation

The process I described years ago is _notoriously dated_. It doesn't handle autoscaling, immutability, or unobtrusive deployments. Given that the process **isn't immutable**, it takes a long time between the moment you decide to launch an instance and the moment it becomes able to start serving responses.

> That being said, most people _(at least most of those who don't rely on a PaaS provider to handle their deployments on a `git push`)_ use some sort of mutable deployment script, often **believing it really is a better strategy than immutable deployments**.

I decided to revisit the topic because I've come across the need to design a robust deployment process that scales well for a project I'm working on, and I naturally came back to what I originally had. Back when I redesigned Pony Foo, I redid [grunt-ec2][2] using just Bash. It definitely has [made it easier on the DevOps side of things][3], but it was still as mutable as ever.

## Why stop using something that works?

There were quite a few problems with [the approach I was taking][1]. To name a few:

- Instances took a long time to boot, and deployments would reuse an existing instance
- Scaling would be hard because there was **no clear cut way to scale horizontally** _(add more hardware)_
- Deployments could effectively [_"brick"_][4] an instance that **took a long time to spin into existence** in the first place
- Thus, deployments could be zero-downtime or "a-few-minutes-_(until a new instance is up and running)_-downtime"
- Configuring a new environment, adding a DNS record set, or [conjuring up a load balancer][16], was carried out by hand

In contrast, I now have more serious requirements than simply running a blog _(which hardly gets updated, too)_. I wanted the system to be able to make deployments safer, without downtime, and without meddling with currently running instances. My experience with mutable deployments in the last two years was _nothing short of disappointing_. Mutable deployments are disappointing because they're super hard to scale, and also because **they make it super easy to disrupt the state of a perfectly healthy production web server**.

I wanted the system to be able to scale out on its own _(to a certain degree)_. While I certainly wouldn't want the system to suddenly boot up tens of [c4.8xlarge][5] instances and get an electricity bill worthy of an operation the size of a nuclear power plant, I wanted something that would operate on a range and was able to **handle mounting traffic** load patterns.

What I really needed was to make server instances **disposable** and make it faster to add extra instances behind the load balancer. The best way to accomplish this is by creating an image. You always use images. Amazon has a huge repository of bare OS images you can use. So far, [as explained in my old article][1], I had been using a Ubuntu base image, and then running my mutable deployments on top of that, over and over.

Instead of creating your EC2 instance with a bare OS image, we'll take a multi-step approach.

1. Create a base image _(code-named `primal`)_ to be **reused in every deploy**
  - Based on the bare OS image
  - Make basic OS networking tweaks
  - Install and configure `nginx` and `node`
  - Install your application's `npm` dependencies so that the next step takes less time
  - Once `primal` is created it can be reused in every subsequent deployment
2. Build your application
  - Compile static assets, running tools like `browserify`
  - Optimize them _(if target environment is **production**)_
3. Create a deployment image _(code-named `carnivore`)_
  - Based on `primal`
  - Upload latest changes to your application
  - Upload latest changes to `nginx` configuration
  - Install latest `npm` dependencies _(faster if "pre-filled" in previous step)_
  - Register the web application as a service in `upstart` or `init.d`
4. Launch an instance based on `carnivore`
  - It'll be **ready to use as soon as it boots** and the application starts
  - Launching extra instances can skip provisioning steps `1`,  `2`, and `3`

As you can see, this workflow is _a bit more involved_ than what you need to do to get mutable deployments up and running, but it will definitely be worth your while.

When your **infrastructure becomes disposable**, you can supervise your instances. As soon as one of them seems unhealthy you could create a replacement, and then eventually terminate the unhealthy instance, with the replacement taking its place. This process is fast for immutable systems, but it's a nightmare if you actively rely on specific instances that take a long time to boot up. What happens when _"Dependency Server 3"_ is down? You're unable to scale.

The new approach doesn't get rid of the fixation on third party dependency management systems, but it contains that to the image creation steps. This is already an improvement, because you'll then be able to scale out your infrastructure even when dependency services are down _(as an image would have been already created once)_.

Better yet, you can leave the disposing and scaling to an [Auto Scaling Group][13] in Amazon, as we'll uncover in the next article.

# Using Packer

I've only recently stumbled upon [Packer][6]. Here's how they describe it.

> **Packer** is a tool for creating identical machine images for multiple platforms from a single source configuration.

First off, you'll need to [install Packer][7]. If you're using OSX and like `brew`ing things, you're in luck.

```shell
brew tap homebrew/binary
brew install packer
```

The _"single source configuration"_ they speak of is really a small_ish_ JSON manifest that describes user variables, how the image is going to be built, and how it's going to be provisioned. In our case, we'll have two of these manifests. One is for the `primal` image I described in step 1 above, and the other is for the `carnivore` image described in step 3.

# Defining the base image

Below you'll find the `primal` manifest used to bake images for **Pony Foo**. You'll note that it's mostly readable, but we'll tear it apart anyways.

```json
{
  "variables": {
    "INSTANCE_USER": "admin",
    "NVM_VERSION": "v0.24.0",
    "NODE_VERSION": "0.10"
  },
  "builders": [{
    "type": "amazon-ebs",
    "region": "us-east-1",
    "source_ami": "ami-22de994a",
    "instance_type": "t1.micro",
    "ssh_username": "{{user `INSTANCE_USER`}}",
    "ami_name": "ponyfoo-primal {{timestamp}}"
  }],
  "provisioners": [{
    "type": "file",
    "source": "deploy/mailtube",
    "destination": "/tmp/mailtube"
  }, {
    "type": "shell",
    "environment_vars": [
      "INSTANCE_USER={{user `INSTANCE_USER`}}",
      "NVM_VERSION={{user `NVM_VERSION`}}",
      "NODE_VERSION={{user `NODE_VERSION`}}"
    ],
    "script": "deploy/templates/primal"
  }]
}
```

The first section of the manifest has a series of `variables`. 

```json
{
  "variables": {
    "INSTANCE_USER": "admin",
    "NVM_VERSION": "v0.24.0",
    "NODE_VERSION": "0.10"
  }
}
```

Variables in packer can be modified by the user who is creating an image. We'll just allow them to change the version of [nvm][8] and the version of `node` that they want installed. `nvm` is a "Node Version Manager" that makes it very easy to switch between versions of `node` _(and `io.js`)_, which makes it ideal if we want to leave the version of `node` up to the image builder.

Next up we have the `builders` section.

```json
{
  "builders": [{
    "type": "amazon-ebs",
    "region": "us-east-1",
    "source_ami": "ami-22de994a",
    "instance_type": "t1.micro",
    "ssh_username": "{{user `INSTANCE_USER`}}",
    "ami_name": "ponyfoo-primal {{timestamp}}"
  }]
}
```

First off, I should point out that `ami-22de994a` is a **Debian Wheezy** base image. The `ssh_username` is what will be used to `ssh` onto the instance while provisioning, and the `{{user VARIABLE}}` expression allows us to make the user configurable through `variables`. The AMI name contains a `{{timestamp}}` expression so that the resulting AMI has a unique name, a requirement of AWS. The rest of the configuration speaks for itself: we want a `t1.micro` instance backed by [EBS storage][9] on the `us-east-1` region.

Oh, also, I'm prefixing the AMI with `ponyfoo-` so that it doesn't collide with `primal` images for other applications in my AWS account. Throughout the series, you'll notice that resources are tagged with the application name, and sometimes also with the execution environment _(`staging`, `production`, etc.)_ This helps us to both quickly identify what application an instance belongs to, as well prevent naming collisions across our AWS resources.

We've saved the best for last, and we're now getting to the `provisioners` in our [Packer][6] manifest. There's two provisioners, let's start with the `file` one.

```json
{
  "type": "file",
  "source": "deploy/mailtube",
  "destination": "/tmp/mailtube"
}
```

There's quite a bit going on here. The AMI image builder in Packer spins up an Amazon EC2 instance, where you make your adjustments _(provisioning)_ and then it gets frozen into an AMI. Provisioners come into play once the instance is accessible via `ssh`.

We're going to upload an entire directory to `/tmp/mailtube`. [Packer][6] will figure out on its own whether to use `rsync`, `scp`, or something else entirely. _Not our problem!_

![A mail tube][10]

We're going to send a few precious things up the tube.

- A template to set up our service in `init.d`
- Templates to provision our application in `nginx`
- The initial `package.json` so we can install some preliminary packages

The `package.json` step seems hacky, but it's also crucial. Installing dependencies takes _a long time_, and having as many of them pre-installed in `primal` allows us to keep the build time for `carnivore` at a minimum. Considering we're going to build `primal` far less often, it's a good thing to do. _We'll worry about copying our `package.json` over to the mailtube later._

There's also the `shell` provisioner. This one uploads and runs a script on the instance we're using to define our AMI.

```json
{
  "type": "shell",
  "environment_vars": [
    "INSTANCE_USER={{user `INSTANCE_USER`}}",
    "NVM_VERSION={{user `NVM_VERSION`}}",
    "NODE_VERSION={{user `NODE_VERSION`}}"
  ],
  "script": "deploy/templates/primal"
}
```

Here again you see a few `{{user VARIABLE}}` expressions, used to expose those user variables to the shell script described below. This is probably where your provisioning of `primal` is going to differ the most from mine, so let's take it slow.

# Provisioning `primal` with Bash

First off, this is a Bash script [_(full script here)_][11], not a lot of mystery.

```bash
#!/bin/bash
```

We kick off our provisioning by updating `apt-get`, and installing essentials such as `git`, `curl`, and `gm`.

```bash
echo "packer: updating aptitude"
sudo apt-key update
sudo apt-get update
sudo apt-get remove apt-listchanges
sudo apt-get install git make g++ graphicsmagick curl -y
```

Right after, we'll install `nginx`. We need to `chown` the logs or else we'll run into trouble when starting `nginx`.

```bash
echo "packer: nginx"
sudo mkdir -p /var/log/nginx
sudo chown $INSTANCE_USER /var/log/nginx
sudo chmod -R 755 /var/log/nginx
sudo apt-get install nginx -y
```

We install `nvm` right off of GitHub, using `NVM_VERSION` as provided.

```bash
echo "packer: nvm"
curl https://raw.githubusercontent.com/creationix/nvm/$NVM_VERSION/install.sh | bash
. ~/.nvm/nvm.sh
```

Time to install `node` at the agreed upon `NODE_VERSION` version, and update `npm` to the latest possible version. Updating `npm` seems unnecessary but it actually resolves a lot of potential issues.

```bash
echo "packer: nodejs"
nvm install $NODE_VERSION
nvm alias default $NODE_VERSION
npm update -g npm
```

Make sure we source `nvm` if we manually `ssh` into the instance, for convenience.

```bash
echo '[[ -s $HOME/.nvm/nvm.sh ]] && . $HOME/.nvm/nvm.sh' >> ~/.bashrc
```

The next command makes TCP a tad faster for bursty connections _(HTTP is bursty)_.

```bash
echo "packer: tweaking tcp"
sudo sysctl -w net.ipv4.tcp_slow_start_after_idle=0
```

The next batch of commands enables port forwarding, forwards port `80`
 to port `8080`, and instructs the instance to remember the new port forwarding rules across restarts. This is so that our application doesn't have to bind on port `80`, which would require elevated privileges.

```bash
echo "packer: ipv4 forwarding"
cp /etc/sysctl.conf /tmp/
echo "net.ipv4.ip_forward = 1" >> /tmp/sysctl.conf
sudo cp /tmp/sysctl.conf /etc/
sudo sysctl -p /etc/sysctl.conf

echo "packer: forward port 80 to 8080"
sudo iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -A INPUT -p tcp -m tcp --sport 80 -j ACCEPT
sudo iptables -A OUTPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables-save > /tmp/iptables-store.conf
sudo mv /tmp/iptables-store.conf /etc/iptables-store.conf

echo "packer: remember port forwarding rule across reboots"
echo "#!/bin/sh" > /tmp/iptables-ifupd
echo "iptables-restore < /etc/iptables-store.conf" >> /tmp/iptables-ifupd
chmod +x /tmp/iptables-ifupd
sudo mv /tmp/iptables-ifupd /etc/network/if-up.d/iptables
```

The time has come to pre-fill `node_modules`. Since we don't actually have the application yet _(the tube only had a `package.json`)_, we'll install these dependencies somewhere and then move them later. We only care about `--production` modules, of course.

```bash
echo "packer: precaching server dependencies"
mkdir -p ~/app/precache
cp -r /tmp/mailtube ~/app/mailtube
cp ~/app/mailtube/package.json ~/app/precache
npm install --prefix ~/app/precache --production
```

Finally, we tell `init.d` to run `nginx` at system startup.

```bash
echo "packer: installing nginx at system startup"
sudo update-rc.d nginx defaults
```

**Sweet!** We're done with the first step! Running `packer build primal.json` will produce the `primal` image and give us the ID for the AMI, _`ami-xxxxxx`_.

![Screenshot of the terminal output when building primal image][12]

That took us a bunch of time, but to be quite fair we did all of this in [our old setup][1]. The only difference is that now you can _"take a shortcut"_, and create instances based on this AMI, saving you the time of installing all those lumps of dependencies over and over again.

We can now build `carnivore` based on the `primal` AMI.

# Building `carnivore` with Packer

Now that we have `primal`, and assuming you'll take it upon yourself to build and optimize your static assets, it's time for step 3, burning `carnivore`. Given that `carnivore` is built on virtually every deployment, it has more configurability built into it, and at the same time its provisioning script is smaller.

First off, some variables. Arguably you'll rarely want to change the amount of `nginx` workers, but you'll definitely want to be able to set the `NODE_ENV`, `SERVER_NAME`, and whatnot.

```json
{
  "variables": {
    "NODE_ENV": "staging",
    "SERVER_NAME": "ponyfoo.com",
    "INSTANCE_USER": "admin",
    "NGINX_WORKERS": "4",
    "SOURCE_ID": null
  }
}
```

As you can see below, the `SOURCE_ID` variable is used as input for the base AMI to be extended by `carnivore`. This will be configured by our deployment script to use the ID provided by Packer when building `primal`. The AMI in this case is also tagged with `NODE_ENV`, because you built your static assets specifically for that environment, right before building the image.

```json
{
  "builders": [{
    "type": "amazon-ebs",
    "region": "us-east-1",
    "instance_type": "t1.micro",
    "ssh_username": "{{user `INSTANCE_USER`}}",
    "ami_name": "ponyfoo-carnivore-{{user `NODE_ENV`}} {{timestamp}}",
    "source_ami": "{{user `SOURCE_ID`}}"
  }]
}
```

Once again we have two provisioners. The first one is a directory upload, just like last time. This time, we're uploading everything that's needed to run our application in its current state. Given that we've built static assets on our local environment, we'll copy the essentials to run the application onto `tmp/appserver` and upload that to build the image.

```json
{
  "type": "file",
  "source": "tmp/appserver",
  "destination": "/tmp/appserver"
}
```

We have the base `primal` image as well as the uploaded `/tmp/appserver` application. Our last step is to upload and execute the `carnivore` provisioning script.

```json
{
  "type": "shell",
  "environment_vars": [
    "INSTANCE_USER={{user `INSTANCE_USER`}}",
    "NGINX_WORKERS={{user `NGINX_WORKERS`}}",
    "SERVER_NAME={{user `SERVER_NAME`}}",
    "NODE_ENV={{user `NODE_ENV`}}",
    "NAME=ponyfoo-{{user `NODE_ENV`}}"
  ],
  "script": "deploy/templates/carnivore"
}
```

Once the script is uploaded, Packer will execute it and our final AMI will be ready.

# Priming `carnivore` with Bash

Again, we'll use a Bash script. You can also [skip ahead to the full script on GitHub][19].

```bash
#!/bin/bash
```

For some awkward reason, I can't figure out why I need to source `nvm` again _(after adding it to `~/.bashrc`)_. It works when I `ssh` into the instance, but not here, unless I explicitly source it again.

```bash
echo "packer: sourcing nvm"
. ~/.nvm/nvm.sh
```

We now take the `nginx.conf` and `site.conf` templates made available to `primal` and copy them over to our application directory. We use `sed` to replace a bunch of variables in the template, and we link up `/etc/nginx/nginx.conf` to what we did. I'm using `~/app/server/.bin/public` as the static root just because that's where I end up compiling static assets, no other particular reason.

```bash
echo "packer: updating nginx configuration"
cp -r ~/app/mailtube/nginx ~/app/nginx

sed -i "s#{NGINX_USER}#$INSTANCE_USER#g" ~/app/nginx/nginx.conf
sed -i "s#{NGINX_WORKERS}#$NGINX_WORKERS#g" ~/app/nginx/nginx.conf
sed -i "s#{SERVER_NAME}#$SERVER_NAME#g" ~/app/nginx/site.conf
sed -i "s#{STATIC_ROOT}#$HOME/app/server/.bin/public#g" ~/app/nginx/site.conf

sudo ln -sfn ~/app/nginx/nginx.conf /etc/nginx/nginx.conf
sudo ln -sfn ~/app/nginx/site.conf /etc/nginx/sites-enabled/$NAME.conf
sudo rm /etc/nginx/sites-enabled/default
```

Once that's done, we make sure `nginx` is up _(or whine about it.)_

```bash
sudo service nginx restart || sudo service nginx start || (sudo cat /var/log/nginx/error.log && exit 1)
```

We take the `appserver` directory that we've just uploaded and move it to the application directory. We also take the modules we've pre-installed during `primal` provisioning. Then, we install whatever dependencies are missing or outdated. Again on a personal note, I always upload a specially crafted `deploy/env/$NODE_ENV.json` with environment secrets, for convenience.

```bash
echo "packer: moving uploaded server code"
mv /tmp/appserver ~/app/server

echo "packer: installing server dependencies"
mv ~/app/server/deploy/env/$NODE_ENV.json ~/app/server/.env.json
mv ~/app/precache/node_modules ~/app/server/node_modules
npm install --prefix ~/app/server --production
```

Lastly we register a service on `init.d` for the web application to run at startup. Again, I believe I shouldn't be sourcing `nvm` if I'm already using `bash` for an environment, but it won't show up unless I do. The `init.d` service configuration template was previously uploaded through the tube, and it's configurable mostly so that I can reuse it across applications using a similar deployment strategy.

```bash
echo "packer: installing appserver daemon..."
echo "#!/bin/bash" > ~/app/start
echo ". ~/.nvm/nvm.sh" >> ~/app/start
echo "node ~/app/server/cluster.js" >> ~/app/start
chmod +x ~/app/start
cp ~/app/mailtube/init.d/appserver.conf ~/app/$NAME.conf
sed -i "s#{NAME}#$NAME#g" ~/app/$NAME.conf
sed -i "s#{DESCRIPTION}#Web application daemon service for $NAME#g" ~/app/$NAME.conf
sed -i "s#{USER}#$INSTANCE_USER#g" ~/app/$NAME.conf
sed -i "s#{COMMAND}#$HOME/app/start#g" ~/app/$NAME.conf
sudo mv ~/app/$NAME.conf /etc/init.d/$NAME
sudo chmod +x /etc/init.d/$NAME
sudo touch /var/log/$NAME.log
sudo chown $INSTANCE_USER /var/log/$NAME.log
sudo update-rc.d $NAME defaults

echo "packer: booting appserver daemon..."
sudo service $NAME start
```

Note that, given that this is a hosted environment, I play it safe and always run the `node` application off of `cluster`.

**That's it!** The command below will bring your per-deployment AMI to life. `PRIMAL_ID` should be the ID of your `primal` AMI.

```bash
packer build \
  -var NODE_ENV=$NODE_ENV \
  -var SOURCE_ID=$PRIMAL_ID \
  carnivore.json
```

As far as building images for immutable deployments goes, that's all you need.

The next article will describe the process through which new environments are set up and deployed to with zero-downtime, all the while leveraging autoscaling groups.

> Or in **Amazon Web Services acronym speak**: "In the next article we'll learn how to leverage [AWS ASG][13] to deploy [EC2][15] instances behind an [ELB][16] that's connected to [Route53][14] changing the [LC][18] and the [AMI][17] every time."

[1]: /articles/deploying-node-apps-to-aws-using-grunt "Deploying Node apps to AWS using Grunt on Pony Foo"
[2]: http://github.com/bevacqua/grunt-ec2 "grunt-ec2 on GitHub"
[3]: https://github.com/ponyfoo/ponyfoo/tree/master/build/ec2 "Bash EC2 deployment scripts on GitHub"
[4]: http://en.wikipedia.org/wiki/Brick_%28electronics%29 "Brick (electronics) on Wikipedia"
[5]: http://aws.amazon.com/ec2/instance-types/#c4 "Amazon Web Services Instance Types"
[6]: http://packer.io/ "Packer by HashiCorp"
[7]: https://packer.io/intro/getting-started/setup.html "Installing Packer"
[8]: https://github.com/creationix/nvm "creationix/nvm on GitHub"
[9]: http://aws.amazon.com/ebs/ "Elastic Block Store on Amazon"
[10]: https://i.imgur.com/u4Vdu3R.jpg
[11]: https://github.com/bevacqua/baal/blob/master/deploy/templates/primal "Primal provisioning script on GitHub"
[12]: https://i.imgur.com/LW1gBuj.png
[13]: http://aws.amazon.com/autoscaling/ "Amazon Web Services Auto Scaling"
[14]: http://aws.amazon.com/route53/ "Amazon Web Services Route 53"
[15]: http://aws.amazon.com/ec2/ "Amazon Web Services Elastic Cloud Compute"
[16]: http://aws.amazon.com/elasticloadbalancing/ "Amazon Web Services Elastic Load Balancer"
[17]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html "Amazon Web Services Machine Images"
[18]: http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WorkingWithLaunchConfig.html "Amazon Web Services Launch Configurations"
[19]: https://github.com/bevacqua/baal/blob/master/deploy/templates/carnivore "Carnivore provisioning script on GitHub"
