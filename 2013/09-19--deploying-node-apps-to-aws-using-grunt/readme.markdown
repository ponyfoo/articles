<div></div>

<h1>Deploying Node apps to AWS using Grunt</h1>

<p><kbd>aws</kbd> <kbd>ec2</kbd> <kbd>grunt</kbd> <kbd>nodejs</kbd> <kbd>grunt-ec2</kbd></p>

<blockquote><p>I&#x2019;ve been toying with <strong>AWS</strong> for a few days now, and I wanted to share my experience and my approach with you. My goal was to set up a deploy flow in Grunt to enable &#x2026;</p></blockquote>

<div><p>I&#x2019;ve been toying with <strong>AWS</strong> for a few days now, and I wanted to share my experience and my approach with you. My goal was to set up a deploy flow in Grunt to enable me to spin up new [EC2][3] instances in the AWS cloud and deploy to them easily.</p></div>

<div></div>

<div><p>The full title should&#x2019;ve been something like: <em>Deploying Node applications to <a href="http://aws.amazon.com/" target="_blank">Amazon Web Services (AWS)</a> through their <a href="https://github.com/aws/aws-cli" target="_blank">CLI tools</a> using Grunt</em>. That was way too long, though. In this post I&#x2019;ll look at being able to do all of the following in my command-line:</p> <ul> <li>Creating <a href="http://en.wikipedia.org/wiki/Secure_Shell#Version_2.x" target="_blank">SSH</a> key-pairs and uploading their public key to AWS</li> <li>Launching EC2 <em>micro</em> instance with an <a href="http://cloud-images.ubuntu.com/releases/raring/release-20130423/" target="_blank"><em>Ubuntu AMI</em></a></li> <li><code class="md-code md-code-inline">ssh</code> into the instance using the private key associated with an instance, and get it set up for future deploys: <ul> <li>Install <strong>Node.js</strong>, duh</li> <li>Install <a href="https://github.com/Unitech/pm2" target="_blank">pm2</a> to keep our application alive</li> </ul> </li> <li>Shut down instances with the same ease as we can launch them, deleting key-pairs</li> <li>Get a list of EC2 instances</li> <li>Get a command to SSH myself to an instance, in case I want to do some heavy lifting myself</li> <li>Deploy to our instance using <code class="md-code md-code-inline">rsync</code>, for <a href="https://help.ubuntu.com/community/rsync" target="_blank">blisteringly fast data transfers</a>, then: <ul> <li>Copy the files somewhere else</li> <li>Install npm packages using <code class="md-code md-code-inline">npm install --production</code></li> <li>Restart our Node application using <code class="md-code md-code-inline">pm2 reload all</code></li> </ul> </li> <li>Refer to instances by a name tag like <code class="md-code md-code-inline">&apos;bambi&apos;</code>, rather than an ID such as <code class="md-code md-code-inline">i-he11f00ba4</code></li> </ul> <p><img src="https://i.imgur.com/Yya9AIy.png" alt="amazon-web-services.png" title="Amazon Web Services"></p> <p>If this sounds like something you&#x2019;d be interested in learning about, read on.</p></div>

<div><h5 id="what-is-missing-in-this-installment">What is missing in this installment?</h5> <ul> <li>In the future, I&#x2019;d also like to install <a href="http://nginx.com/" target="_blank" aria-label="nginx HTTP proxy server">nginx</a></li> <li>I&#x2019;m not creating individual users for <code class="md-code md-code-inline">rsync</code>, <code class="md-code md-code-inline">node</code>, as I&#x2019;m not entirely sure of how to manage that</li> <li>I&#x2019;m not setting up port forwarding, not giving my instance a public IP</li> <li>I didn&#x2019;t establish a sensible way to swap configuration based on the environment, yet</li> </ul> <p><strong>Yet&#x2026;</strong> This is a <em>work in progress</em>, and I expect to do all of the above, soon. I&#x2019;ll write a follow-up article to keep you updated.</p> <h2 id="introduction">Introduction</h2> <p>In the past, I&#x2019;ve been using <a href="https://www.heroku.com/" target="_blank" aria-label="Heroku Cloud Application Platform">Heroku</a> to host my Node applications, <em>(this blog is hosted on Heroku)</em>. For convenience, and because I didn&#x2019;t really know how to interact with AWS, I&#x2019;d never even tried it before. Now that <strong>I&#x2019;m writing a book on build processes and architecture for Node applications</strong>, I deemed it necessary to teach myself <em>how to use AWS directly</em>, rather than through a <a href="http://en.wikipedia.org/wiki/Platform_as_a_service" target="_blank" aria-label="Platform as a Service">PaaS</a> provider such as Heroku.</p> <p>However, I didn&#x2019;t want to lose the ability to deploy from my command-line. In fact, I wanted to do one better, and become able to spin up new environments with just a <a href="http://gruntjs.com/" target="_blank" aria-label="Grunt.js JavaScript Task Runner">Grunt</a> command. A little while ago, Amazon <a href="https://github.com/aws/aws-cli/releases/tag/1.0.0" target="_blank" aria-label="aws-cli Release 1.0.0 on GitHub">released version 1.0.0</a> of their CLI tools, my understanding: a huge improvement over their previous command-line tooling, but I never really used their previous version before.</p> <p>Some of the tasks took a little iterating to get right, but considering AWS gives you free access to their services for a year when you first sign up, this wasn&#x2019;t really an issue. Keeping in mind the fact that <em>these are automated tasks</em>, the goal for me was to get it right once, and then be _able to re-use it in any other projects I might want to host on the AWS platform in the future.</p> <h2 id="requirements">Requirements</h2> <p>I started off by reading up on the requirements for the <strong>aws-cli</strong> tool. I wanted to do as little manual labor as possible, after researching a little I figured out my requirements to use the <code class="md-code md-code-inline">aws</code> CLI tool unobtrusively.</p> <ul> <li>Have an <strong>AWS</strong> account</li> <li>Get an Access Key, write down <code class="md-code md-code-inline">AWS_ACCESS_KEY_ID</code> and <code class="md-code md-code-inline">AWS_SECRET_ACCESS_KEY</code></li> <li>Pick a default region for <code class="md-code md-code-inline">AWS_DEFAULT_REGION</code></li> <li>Install <code class="md-code md-code-inline">pip</code>, the Python <a href="http://www.pip-installer.org/en/latest/installing.html" target="_blank" aria-label="pip installation instructions">package manager</a>, if it&#x2019;s not present</li> <li>Use it to install the CLI: <code class="md-code md-code-inline">pip install awscli --upgrade</code></li> </ul> <p>I actually created a Grunt task to install the <code class="md-code md-code-inline">aws</code> CLI, but that&#x2019;s not required. Next, the CLI requires you to provide some <em>environment variables</em>. Luckily, I can provide an <code class="md-code md-code-inline">env</code> programmatically to <a href="http://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback" target="_blank" aria-label="Child Process Node.js Documentation">processes I spawn</a> with Node, so I&#x2019;ll do that instead of touching <a href="http://en.wikipedia.org/wiki/Environment_variable" target="_blank" aria-label="Environment Variables on Wikipedia">environment variables</a>.</p> <p>Let&#x2019;s first look at the JSON file I set up to deal with all my AWS-related &#x201C;environment&#x201D; variables.</p> <pre class="md-code-block"><code class="md-code md-lang-json">{
    &quot;<span class="md-code-attribute">AWS_ACCESS_KEY_ID</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;redacted&quot;</span></span>,
    &quot;<span class="md-code-attribute">AWS_SECRET_ACCESS_KEY</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;redacted&quot;</span></span>,
    &quot;<span class="md-code-attribute">AWS_DEFAULT_REGION</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;us-east-1&quot;</span></span>,
    &quot;<span class="md-code-attribute">AWS_IMAGE_ID</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;ami-c30360aa&quot;</span></span>,
    &quot;<span class="md-code-attribute">AWS_INSTANCE_TYPE</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;t1.micro&quot;</span></span>,
    &quot;<span class="md-code-attribute">AWS_SECURITY_GROUP_NAME</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;standard&quot;</span></span>,
    &quot;<span class="md-code-attribute">AWS_SSH_USER</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;ubuntu&quot;</span></span>,
    &quot;<span class="md-code-attribute">AWS_RSYNC_USER</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;ubuntu&quot;</span>
</span>}
</code></pre> <p>The one thing I <em>didn&#x2019;t automate</em> was <strong>security group creation</strong>, plainly because this was something I figured you rarely have to deal with, and it&#x2019;s one of the first steps you have to take, so I was eager to do other stuff, rather than deal with security groups. It&#x2019;s no mystery either, all you have to do is enable ports 22, for <strong>SSH</strong>, and 80, for <strong>HTTP</strong>.</p> <p>The <code class="md-code md-code-inline">t1.micro</code> instance type is the one to use if you want to be eligible for the free tier benefits (that reads: if you don&#x2019;t want to pay up). Ideally, the <code class="md-code md-code-inline">rsync</code> user should be a different one from the default SSH user, but like most things, it&#x2019;s an easily changeable setting that I didn&#x2019;t want to waste a bunch of time on. Take into account any user other than <code class="md-code md-code-inline">ubuntu</code> would need to be created the first time we <code class="md-code md-code-inline">ssh</code> into the instance, so that it&#x2019;s available to future sessions.</p> <h2 id="executing-commands-against-the-aws-api">Executing commands against the AWS API</h2> <p>I won&#x2019;t be boring you with the details of how I got to the current state of affairs, and I&#x2019;ll jump to the latest versions of my code, instead. We&#x2019;ve barely touched the surface now, though. The first thing to set up is a reusable piece of code that&#x2019;ll make running commands, as if we were in a terminal, pretty easy. This module is used pretty much everywhere else in the task suite I developed to interact with <strong>AWS</strong>.</p> <p>Keep in mind all of this code is meant to be run in development machines, with <code class="md-code md-code-inline">grunt</code> available to them, and these tasks will be executed from the command-line directly. Thus, throwing exceptions with <code class="md-code md-code-inline">grunt.fatal</code> is the correct approach, rather than using <code class="md-code md-code-inline">function(err, result)</code> type callbacks.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-pi">&apos;use strict&apos;</span>;

<span class="md-code-keyword">var</span> grunt = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;grunt&apos;</span>);
<span class="md-code-keyword">var</span> util = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;util&apos;</span>);
<span class="md-code-keyword">var</span> exec = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;child_process&apos;</span>).exec;

<span class="md-code-built_in">module</span>.exports = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(command, args, done, print)</span></span>{
    args.unshift(command);

    <span class="md-code-keyword">var</span> cmd = util.format.apply(util, args);

    grunt.verbose.writeln(cmd);

    exec(cmd, { env: conf() }, callback);

    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">callback</span> <span class="md-code-params">(err, stdout, stderr)</span> </span>{
        <span class="md-code-keyword">if</span> (err) { grunt.fatal(err); }
        <span class="md-code-keyword">if</span> (stderr) { grunt.fatal(stderr); }

        <span class="md-code-keyword">if</span> (print !== <span class="md-code-literal">false</span>) {
            grunt.log.writeln(stdout);
            done();
        } <span class="md-code-keyword">else</span> {
            done(stdout);
        }
    }
};
</code></pre> <p>This module quite simply allows me to execute commands with a very simple API, and then either print them to <code class="md-code md-code-inline">stdout</code> or fiddle with the data. Whenever there&#x2019;s a hiccup, I&#x2019;ll throw and halt execution. This is important because I don&#x2019;t want to continue randomly throwing calls into a sensitive API, that might even end up costing me money, because I didn&#x2019;t preemptively end the process.</p> <p>At this point you might have realized this doesn&#x2019;t even compile.</p> <blockquote> <p>What the hell is <code class="md-code md-code-inline">conf()</code>?</p> </blockquote> <p>Yeah, about that. <code class="md-code md-code-inline">conf</code> is a <a href="http://nodejs.org/api/globals.html" target="_blank" aria-label="Node Globals">global variable</a>, and it was set by the line below, <em>somewhere else</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">global.conf = <span class="md-code-built_in">module</span>.exports = nconf.get.bind(nconf);
</code></pre> <p>The <a href="https://github.com/flatiron/nconf" target="_blank" aria-label="flatiron/nconf on GitHub">nconf</a> module is a nice little utility that allows you to keep your environment configuration in one place. This is particularly convenient for our use case, where I configured <code class="md-code md-code-inline">nconf</code> more or less like this:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-pi">&apos;use strict&apos;</span>;

<span class="md-code-keyword">var</span> nconf = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;nconf&apos;</span>);
<span class="md-code-keyword">var</span> path = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;path&apos;</span>);
<span class="md-code-keyword">var</span> aws = path.join(__dirname, <span class="md-code-string">&apos;private/aws.json&apos;</span>);

nconf.argv();
nconf.env();
nconf.file(<span class="md-code-string">&apos;aws&apos;</span>, aws);

global.conf = <span class="md-code-built_in">module</span>.exports = nconf.get.bind(nconf);
</code></pre> <p>I believe <strong>this is the one and only respectable use of a global variable in Node</strong>, and I don&#x2019;t use them for anything else, <em>and neither should you</em>.</p> <p>Fine, that was a lot of code and we&#x2019;re still not interacting with <strong>AWS</strong> any more than we were five minutes ago. Fair enough, here&#x2019;s a <code class="md-code md-code-inline">grunt</code> task you could use immediately:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-pi">&apos;use strict&apos;</span>

<span class="md-code-keyword">var</span> exec = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./exec.js&apos;</span>);

<span class="md-code-built_in">module</span>.exports = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(grunt)</span> </span>{
    grunt.registerTask(<span class="md-code-string">&apos;ec2_lookup&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(name)</span></span>{
        exec(<span class="md-code-string">&apos;aws ec2 describe-instances --filters Name=tag:Name,Values=%s&apos;</span>, [name], <span class="md-code-keyword">this</span>.async());
    });
};
</code></pre> <p>Sweet, huh? That&#x2019;s really concise! Now we can run <code class="md-code md-code-inline">grunt ec2_lookup:foobar</code> and get some JSON describing the metadata for an EC2 instance tagged with the name <code class="md-code md-code-inline">foobar</code>. That&#x2019;s nice syntactic sugar, but barely any better than using the <code class="md-code md-code-inline">aws</code> CLI tool directly. Well, the power of this approach reveals itself when we start combining different commands in new and useful ways. Let&#x2019;s start heading in the direction of launching a new EC2 instance. That&#x2019;s pretty easy to do using their <a href="https://console.aws.amazon.com/console/home" target="_blank" aria-label="Amazon Web Services Console Home">web interface</a>, but we can spend a bunch of time typing away at our terminal and it&#x2019;ll be pretty hard to get it right, besides: we came for the automation.</p> <p>Here&#x2019;s a similar task in action, which allows us to look up instances based on their state:</p> <p><img alt="ec2-list.png" class="" src="https://i.imgur.com/ecFsa4b.png"></p> <h2 id="launching-an-ec2-instance">Launching an EC2 instance</h2> <p>The first step we have to take for launching an EC2 instance, is generating an SSH key-pair so that we can access it from the command-line. Then we have to actually run the instance, providing a bunch of parameters to the CLI. Our launch task looks pretty simple:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-pi">&apos;use strict&apos;</span>;

<span class="md-code-built_in">module</span>.exports = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(grunt)</span></span>{

    grunt.registerTask(<span class="md-code-string">&apos;ec2_launch&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(name)</span></span>{

        grunt.task.run([
            <span class="md-code-string">&apos;ec2_create_keypair:&apos;</span> + name,
            <span class="md-code-string">&apos;ec2_run_instance:&apos;</span> + name,
            <span class="md-code-string">&apos;ec2_setup:&apos;</span> + name
        ]);
    });
};
</code></pre> <p>We&#x2019;ll get into the <code class="md-code md-code-inline">ec2_setup</code> task later on. For now, let&#x2019;s focus on creating a key pair.</p> <h2 id="the-ec2-create-keypair-task">The <code class="md-code md-code-inline">ec2_create_keypair</code> Task</h2> <p>There are two different ways you can do this for AWS EC2 instances. You could either <a href="http://docs.aws.amazon.com/cli/latest/userguide/cli-ec2-keypairs.html" target="_blank" aria-label="Using Key Pairs on AWS CLI documentation">use the CLI to create one</a>, or you could generate one yourself, using <code class="md-code md-code-inline">ssh-keygen</code> and then <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws" target="_blank" aria-label="Generate your own key and import it to AWS">importing it into AWS</a>. In the beginning, I tried to use their CLI to create the key-pairs, but I ran into a few issues, and moved away from that approach, ultimately focusing on <code class="md-code md-code-inline">ssh-keygen</code>, which was better anyways since the private key never travels over the network (in the other approach, <em>Amazon sends you the private key</em>).</p> <p>So how do I generate the private key? Easy!</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">exec(<span class="md-code-string">&apos;ssh-keygen -t rsa -b 2048 -N &quot;&quot; -f %s&apos;</span>, [file], then);
</code></pre> <p>Where file is the path where I want to persist my private key, a public key will also be created at the same place, with a <code class="md-code md-code-inline">.pub</code> extension appended to it. Then, I give amazon the public key.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">exec(<span class="md-code-string">&apos;aws ec2 import-key-pair --public-key-material %s --key-name %s&apos;</span>, [
    <span class="md-code-string">&apos;file://&apos;</span> + file + <span class="md-code-string">&apos;.pub&apos;</span>, name
], done);
</code></pre> <p>I&#x2019;ll admit, the <code class="md-code md-code-inline">&apos;file://&apos;</code> thing is really weird, but I haven&#x2019;t really found any other solution, and <a href="https://github.com/aws/aws-cli/issues/41" target="_blank" aria-label="Related issue on GitHub">this one worked</a>, so I stick to it. Let me know if you&#x2019;ve found any alternatives to that. The <code class="md-code md-code-inline">name</code> variable is one used in almost every one of these <strong>Grunt</strong> tasks I&#x2019;ve implemented, and it&#x2019;s the tag I use to identify instances, as we&#x2019;ll see shortly. The public key also gets uploaded to <strong>AWS</strong> with that name.</p> <h4 id="a-matching-ec2-delete-keypair">A matching <code class="md-code md-code-inline">ec2_delete_keypair</code></h4> <p>This task reverts what was done in <code class="md-code md-code-inline">ec2_create_keypair</code>, and it also takes a <code class="md-code md-code-inline">name</code> argument. We simply execute the <a href="http://docs.aws.amazon.com/cli/latest/reference/ec2/delete-key-pair.html" target="_blank" aria-label="delete-key-pair on AWS CLI documentation">delete-key-pair</a> command, and then physically delete the key-pair from our machine.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">exec(<span class="md-code-string">&apos;aws ec2 delete-key-pair --key-name %s&apos;</span>, [name], removeFromDisk);
</code></pre> <p>If you&#x2019;re reading this, <em>I&#x2019;ll go ahead and assume</em> you know how to delete files using Node.</p> <h2 id="spinning-up-an-instance-in-ec2-run-instance">Spinning up an instance in <code class="md-code md-code-inline">ec2_run_instance</code></h2> <p>This one is both one of the easiest tasks to put together, and the easiest to drastically fuck up when running, due to its sensitivity in that it affects our billing.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">exec(<span class="md-code-string">&apos;aws ec2 run-instances --image-id %s --instance-type %s --count %s --key-name %s --security-groups %s&apos;</span>, [
    conf(<span class="md-code-string">&apos;AWS_IMAGE_ID&apos;</span>), conf(<span class="md-code-string">&apos;AWS_INSTANCE_TYPE&apos;</span>), <span class="md-code-number">1</span>, name, conf(<span class="md-code-string">&apos;AWS_SECURITY_GROUP_NAME&apos;</span>)
], createTag, <span class="md-code-literal">false</span>);
</code></pre> <p>Nothing we haven&#x2019;t really seen before. We&#x2019;re creating an instance that is:</p> <ul> <li>Just one instance</li> <li>Based on the <a href="https://aws.amazon.com/amis" target="_blank" aria-label="Amazon Machine Images">AMI</a>, <code class="md-code md-code-inline">&quot;ami-c30360aa&quot;</code>, as configured in <code class="md-code md-code-inline">aws.json</code></li> <li>Of the <code class="md-code md-code-inline">&quot;t1.micro&quot;</code> <a href="http://aws.amazon.com/ec2/instance-types" target="_blank" aria-label="Amazon EC2 Instance Types">instance type</a></li> <li>In the security group we&#x2019;ve previously created for all of our instances, <code class="md-code md-code-inline">&quot;standard&quot;</code> in my case</li> <li>In the region we chose as default, <code class="md-code md-code-inline">&quot;us-east-1&quot;</code></li> <li>Configured to use the SSH key with the name we&#x2019;re provided</li> </ul> <p>Wow, that&#x2019;s a lot for just one command. Yeah, but what&#x2019;s next? We&#x2019;re not done. We need to create a tag for this instance, so that we can very easily refer to it in all other Grunt tasks we have. Below is what the <code class="md-code md-code-inline">createTag</code> callback is doing.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">createTag</span> <span class="md-code-params">(stdout)</span> </span>{
    <span class="md-code-keyword">var</span> result = <span class="md-code-built_in">JSON</span>.parse(stdout);
    <span class="md-code-keyword">var</span> id = result.Instances[<span class="md-code-number">0</span>].InstanceId;
    <span class="md-code-keyword">var</span> task = util.format(<span class="md-code-string">&apos;ec2_create_tag:%s:%s&apos;</span>, id, name);

    grunt.task.run(task);
    done();
}
</code></pre> <p>Nothing that amusing, I&#x2019;m getting the <code class="md-code md-code-inline">id</code>, putting it together with the <code class="md-code md-code-inline">name</code> we want, and passing those along to the <code class="md-code md-code-inline">ec2_create_tag</code> task. That task will create a tag on our instance, like so:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">exec(<span class="md-code-string">&apos;aws ec2 create-tags --resources %s --tags Key=Name,Value=%s&apos;</span>, [id, name], done);
</code></pre> <p>Phew, that was a lot of work. However, we&#x2019;re now able to spin as many instances as we want from our command-line, how cool is that?!</p> <h4 id="shutting-instances-down">Shutting instances down</h4> <p>Before proceeding, though, I&#x2019;ll share what <code class="md-code md-code-inline">ec2_shutdown</code> does for us: it&#x2019;ll delete the key-pair with the task we talked about earlier, after <a href="http://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html" target="_blank" aria-label="Terminate Instances with AWS CLI">terminating the instance</a> through <strong>AWS CLI</strong>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">exec(<span class="md-code-string">&apos;aws ec2 terminate-instances --instance-ids %s&apos;</span>, [id], done);
</code></pre> <p>One caveat you might&#x2019;ve realized, is that this command uses the instance ID, which we don&#x2019;t want to provide by hand, since that&#x2019;s messy. Remember the first task we wrote to find instances tagged with a name? Well, that&#x2019;s what we&#x2019;re using to find the <code class="md-code md-code-inline">InstanceId</code>!</p> <h1 id="to-ssh-and-beyond">To <code class="md-code md-code-inline">ssh</code>, and beyond!</h1> <p>After our instance is up, and tagged, we have to wait a bit before we can access its DNS, and then we have to wait a bit more to be able to make an SSH connection. I didn&#x2019;t want to wait myself, so I wrote a task to wait for me.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">waitForDNS</span> <span class="md-code-params">()</span> </span>{
    getCredentials(name, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(c)</span> </span>{
        <span class="md-code-keyword">if</span> (!c) {
            grunt.log.writeln(<span class="md-code-string">&apos;DNS still cold, retrying...&apos;</span>);
            wait(waitForDNS);
            <span class="md-code-keyword">return</span>;
        }
        grunt.log.ok(<span class="md-code-string">&apos;Instance accessible at: %s&apos;</span>, c.host);
        waitForSSH();
    });
}

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">waitForSSH</span> <span class="md-code-params">()</span> </span>{
    grunt.log.writeln(<span class="md-code-string">&apos;attempting to connect...&apos;</span>);
    <span class="md-code-keyword">var</span> connection = ssh([], name, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{}, <span class="md-code-literal">false</span>);

    connection.on(<span class="md-code-string">&apos;error&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
        grunt.log.writeln(<span class="md-code-string">&apos;connection refused, retrying&apos;</span>);
        wait(waitForSSH);
    });

    connection.on(<span class="md-code-string">&apos;ready&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
        grunt.log.ok(<span class="md-code-string">&apos;success, proceeding in 10s&apos;</span>);
        wait(done, <span class="md-code-number">10</span>); <span class="md-code-comment">// apt-get failed if we didn&apos;t wait a bit.</span>
    });
}
waitForDNS(); <span class="md-code-comment">// first attempt</span>
</code></pre> <p>Let me explain the two functions that are not described here, <code class="md-code md-code-inline">getCredentials</code> and <code class="md-code md-code-inline">ssh</code>. The former just uses a command similar to our <code class="md-code md-code-inline">ec2_lookup</code> from earlier, and checks whether the public DNS field is set. The latter is a little more complicated, it queues commands to be executed over SSH, but since we&#x2019;re passing an empty array of commands, we&#x2019;re just testing if it can make a connection. When we&#x2019;re done, we wait a few seconds for good measure, because otherwise the system might not be <em>entirely</em> operational. Before adding that 10 second wait there, my following task was consistently failing to install anything using <a href="https://help.ubuntu.com/community/AptGet/Howto" target="_blank" aria-label="Package Management with APT">apt-get</a>. After the wait, the problem evaporated.</p> <h2 id="installing-things-in-our-new-instance">Installing things in our new instance!</h2> <p>Now that the wait is finally over, we can start using our instance. Yeah, about that&#x2026; we need to install things first, it&#x2019;s a brand new machine! <em>(well, sort of)</em></p> <p>Using the same <code class="md-code md-code-inline">ssh</code> queueing mechanism I talked about in the last section <em>(fine! I&#x2019;ll show it to you in a minute, hang on)</em>, I set up a bunch of things on the new box, over SSH.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> project = conf(<span class="md-code-string">&apos;PROJECT_ID&apos;</span>);
<span class="md-code-keyword">var</span> tasks = [[ <span class="md-code-comment">// rsync</span>
    util.format(<span class="md-code-string">&apos;sudo mkdir -p /srv/rsync/%s/latest&apos;</span>, project),
    util.format(<span class="md-code-string">&apos;sudo mkdir -p /srv/apps/%s/v&apos;</span>, project),
    util.format(<span class="md-code-string">&apos;sudo chown ubuntu /srv/rsync/%s/latest&apos;</span>, project)
], [ <span class="md-code-comment">// node.js</span>
    <span class="md-code-string">&apos;sudo apt-get install python-software-properties&apos;</span>,
    <span class="md-code-string">&apos;sudo add-apt-repository ppa:chris-lea/node.js -y&apos;</span>,
    <span class="md-code-string">&apos;sudo apt-get update&apos;</span>,
    <span class="md-code-string">&apos;sudo apt-get install nodejs -y&apos;</span>
], [ <span class="md-code-comment">// pm2</span>
    <span class="md-code-string">&apos;sudo apt-get install make g++ -y&apos;</span>,
    <span class="md-code-string">&apos;sudo npm install -g pm2&apos;</span>,
    <span class="md-code-string">&apos;sudo pm2 startup&apos;</span>
]];

<span class="md-code-keyword">var</span> commands = _.flatten(tasks);
ssh(commands, name, done);
</code></pre> <p>All I&#x2019;m really doing is installing <code class="md-code md-code-inline">node</code>, <code class="md-code md-code-inline">npm</code>, and <code class="md-code md-code-inline">pm2</code>. After this, all that remains is deploying through <code class="md-code md-code-inline">rsync</code>, and using <code class="md-code md-code-inline">pm2</code> to load our changes on each deploy. Let&#x2019;s stand back for a minute and look at the <code class="md-code md-code-inline">ssh.js</code> module I wrote.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> Connection = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;ssh2&apos;</span>);

<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(commands, name, done)</span></span>{
    <span class="md-code-keyword">var</span> c = <span class="md-code-keyword">new</span> Connection();

    c.on(<span class="md-code-string">&apos;ready&apos;</span>, next);
    c.on(<span class="md-code-string">&apos;close&apos;</span>, done);
    c.on(<span class="md-code-string">&apos;error&apos;</span>, grunt.fatal);

    getCredentials(name, c.connect);

    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">next</span> <span class="md-code-params">()</span> </span>{
        <span class="md-code-keyword">if</span> (commands.length === <span class="md-code-number">0</span>) {
            done();
        } <span class="md-code-keyword">else</span> {
            <span class="md-code-keyword">var</span> command = commands.shift();

            grunt.log.writeln(command);

            c.exec(command, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(err, stream)</span> </span>{
                <span class="md-code-keyword">if</span> (err) { grunt.fatal(err); }

                stream.on(<span class="md-code-string">&apos;data&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(data, extended)</span> </span>{
                    grunt.log.write(<span class="md-code-built_in">String</span>(data));
                });

                stream.on(<span class="md-code-string">&apos;exit&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(code)</span> </span>{
                    <span class="md-code-keyword">if</span> (code === <span class="md-code-number">0</span>) { next(); }
                    grunt.fatal(command, chalk.red(<span class="md-code-string">&apos;exit code &apos;</span> + code));
                });
            });
        }
    }
    <span class="md-code-keyword">return</span> c;
}
</code></pre> <p>See? not that complicated, just some sort of asynchronous queue. A fancy way of running commands over SSH.</p> <h2 id="deploying-with-rsync-and-pm2">Deploying with <code class="md-code md-code-inline">rsync</code> and <code class="md-code md-code-inline">pm2</code></h2> <p>Lastly we&#x2019;ll use <a href="https://help.ubuntu.com/community/rsync" target="_blank" aria-label="rsync, an article by the Ubuntu Community">rsync</a> over SSH to push our current version to the <strong>AWS EC2</strong> instance. We&#x2019;ll then <em>copy</em> with <code class="md-code md-code-inline">cp</code> to a folder named using the current build version; we don&#x2019;t use <code class="md-code md-code-inline">mv</code>: we want it to stay there to be able to exploit the benefits of <code class="md-code md-code-inline">rsync</code> in the future. Then, we just update a symbolic link on the <code class="md-code md-code-inline">current</code> folder, to make sure it maps to the last deployed version of our code, and lastly, we reload our <code class="md-code md-code-inline">node</code> process using <code class="md-code md-code-inline">pm2</code>. Done!</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">exec(<span class="md-code-string">&apos;rsync -vaz --stats --progress --delete %s -e &quot;ssh -o StrictHostKeyChecking=no -i %s&quot; %s %s@%s:%s&apos;</span>, [
    excludeFrom, c.privateKeyFile, local, user, c.host, remote
], deploy);

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">deploy</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-keyword">var</span> dest = util.format(<span class="md-code-string">&apos;/srv/apps/%s/v/%s&apos;</span>, project, v);
    <span class="md-code-keyword">var</span> target = util.format(<span class="md-code-string">&apos;/srv/apps/%s/current&apos;</span>, project);
    <span class="md-code-keyword">var</span> commands = [
        util.format(<span class="md-code-string">&apos;sudo cp -r %s %s&apos;</span>, remoteSync, dest),
        util.format(<span class="md-code-string">&apos;sudo npm --prefix %s install --production&apos;</span>, dest),
        util.format(<span class="md-code-string">&apos;sudo ln -sfn %s %s&apos;</span>, dest, target), [
            util.format(<span class="md-code-string">&apos;sudo pm2 start %s/%s -i 2 --name %s&apos;</span>, target, conf(<span class="md-code-string">&apos;NODE_SCRIPT&apos;</span>), name),
            <span class="md-code-string">&apos;sudo pm2 reload all&apos;</span>
        ].join(<span class="md-code-string">&apos; || &apos;</span>) <span class="md-code-comment">// start or reload</span>
    ];
    ssh(commands, name, done);
}
</code></pre> <p>This works the same for the first deploy, where it&#x2019;ll short-circuit on <code class="md-code md-code-inline">pm2 start /srv/apps/example/current/app.js -i 2 --name staging</code>, rather than reloading.</p> <h1 id="conclusions-and-a-repository">Conclusions and a Repository</h1> <p>I polished what I wrote about in the article and separated it from my original code-base, making it into <a href="https://github.com/bevacqua/grunt-ec2" target="_blank" aria-label="grunt-ec2 on GitHub">a reusable grunt plugin I called grunt-ec2</a>, it&#x2019;s pretty easy to set up, and it doesn&#x2019;t pollute the global namespace or any of that, you simply have to pass a few configuration variables through Grunt as usual. You can read it&#x2019;s documentation on GitHub.</p> <p>I also added some pretty screenshots I took for that repository.</p> <p>Lastly, please <strong>let me know if this article sounded like a lot of gibberish glued together</strong>, or if you <em>actually learned anything</em> from it.</p></div>
