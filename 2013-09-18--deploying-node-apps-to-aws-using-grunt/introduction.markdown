The full title should've been something like: _Deploying Node applications to [Amazon Web Services (AWS)][1] through their [CLI tools][2] using Grunt_. That was way too long, though. In this post I'll look at being able to do all of the following in my command-line:

* Creating [SSH][4] key-pairs and uploading their public key to AWS
* Launching EC2 _micro_ instance with an [_Ubuntu AMI_][5]
* `ssh` into the instance using the private key associated with an instance, and get it set up for future deploys:
  * Install **Node.js**, duh
  * Install [pm2][6] to keep our application alive
* Shut down instances with the same ease as we can launch them, deleting key-pairs
* Get a list of EC2 instances
* Get a command to SSH myself to an instance, in case I want to do some heavy lifting myself
* Deploy to our instance using `rsync`, for [blisteringly fast data transfers][7], then:
  * Copy the files somewhere else
  * Install npm packages using `npm install --production`
  * Restart our Node application using `pm2 reload all`
* Refer to instances by a name tag like `'bambi'`, rather than an ID such as `i-he11f00ba4`

![amazon-web-services.png][8]

If this sounds like something you'd be interested in learning about, read on.

[1]: http://aws.amazon.com/
[2]: https://github.com/aws/aws-cli
[3]: http://aws.amazon.com/ec2
[4]: http://en.wikipedia.org/wiki/Secure_Shell#Version_2.x
[5]: http://cloud-images.ubuntu.com/releases/raring/release-20130423/
[6]: https://github.com/Unitech/pm2
[7]: https://help.ubuntu.com/community/rsync
[8]: https://i.imgur.com/Yya9AIy.png "Amazon Web Services"
