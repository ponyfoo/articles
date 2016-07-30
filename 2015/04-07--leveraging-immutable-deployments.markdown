<h1>Leveraging Immutable Deployments</h1>

<blockquote><p>Last time around, we discussed <a href="https://ponyfoo.com/articles/immutable-deployments-packer">how to create an AMI for every deployment</a>: a crucial step in enabling you to leverage deployment immutability. This time around &#x2026;</p></blockquote>

<div><kbd>bash</kbd> <kbd>aws</kbd> <kbd>ami</kbd> <kbd>immutable-deployments</kbd></div>

<div><p>Last time around, we discussed <a href="https://ponyfoo.com/articles/immutable-deployments-packer">how to create an AMI for every deployment</a>: a crucial step in enabling you to leverage deployment immutability. This time around we&#x2019;ll learn what it takes to automate autoscaling immutable deployments with zero-downtime, while sitting behind a load balancer and using Route 53 for DNS.</p></div>

<div></div>

<div><p>Given the <a href="https://ponyfoo.com/articles/immutable-deployments-packer">immutable images we&#x2019;re now able to build</a>, thanks to the last article, in this article we&#x2019;ll lay out an architecture, based on <strong>Amazon Web Services</strong>, that&#x2019;s able to take advantage of those immutable images.</p> <h1 id="architecture-overview">Architecture Overview</h1> <p>Our journey starts at <a href="http://aws.amazon.com/route53/" target="_blank">Route 53</a>, a DNS provider service from Amazon. You&#x2019;ll set up the initial DNS configuration by hand. I&#x2019;ll leave the <code class="md-code md-code-inline">NS</code>, <code class="md-code md-code-inline">SOA</code>, and <code class="md-code md-code-inline">TXT</code> records up to you. The feature that interests me in Route 53 is <a href="http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html" target="_blank">creating <code class="md-code md-code-inline">ALIAS</code> record sets</a>. These special record sets allow you to bind a domain name to an <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">Elastic Load Balancer <em>(ELB)</em></a>, which can then distribute traffic among your <a href="http://aws.amazon.com/ec2/" target="_blank">Elastic Cloud Compute <em>(EC2)</em></a> server instances.</p> <p>The eye candy in <code class="md-code md-code-inline">ALIAS</code> record sets is that you&#x2019;re able to use them at the naked, or apex domain level <em>(e.g <code class="md-code md-code-inline">ponyfoo.com</code>)</em>, and not just sub-domains <em>(e.g <code class="md-code md-code-inline">blog.ponyfoo.com</code>)</em>. Our scripts will set up these <code class="md-code md-code-inline">ALIAS</code> records on our behalf whenever we want to deploy to a new environment, also a very enticing feature.</p> <blockquote> <p>When you finish reading this article, you&#x2019;ll be able to run the commands below and have your application listening at <code class="md-code md-code-inline">dev01.example.com</code> when they complete, without the need for any manual actions.</p> <pre class="md-code-block"><code class="md-code">NODE_ENV=dev01 npm run setup
NODE_ENV=dev01 npm run deploy
</code></pre> </blockquote> <p>We don&#x2019;t merely use <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">ELB</a> because of its ability to route traffic from naked domains, but also because it enables us to have many web servers behind a single domain, which makes our application more robust and better able to handle load, becoming <em>highly available</em>.</p> <p>In order to better manage the instances behind our <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">ELB</a>, we&#x2019;ll use <a href="http://aws.amazon.com/autoscaling/" target="_blank">Auto Scaling Groups <em>(ASG)</em></a>. These come at no extra cost, you just pay for the <a href="http://aws.amazon.com/ec2/" target="_blank">EC2</a> instances in them. The <a href="http://aws.amazon.com/autoscaling/" target="_blank">ASG</a> will automatically detect <em>&#x201C;unhealthy&#x201D;</em> <a href="http://aws.amazon.com/ec2/" target="_blank">EC2</a> instances, or instances that the <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">ELB</a> has deemed unreachable after pinging them with <code class="md-code md-code-inline">GET</code> requests. Unhealthy instances are automatically disconnected from the <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">ELB</a>, meaning they&#x2019;ll no longer receive traffic. However, we&#x2019;ll enable <a href="https://aws.amazon.com/blogs/aws/elb-connection-draining-remove-instances-from-service-with-care/" target="_blank">connection draining</a> so that they gracefully respond to pending requests before shutting down.</p> <p>When a new deployment is requested, we&#x2019;ll create a new <a href="http://aws.amazon.com/autoscaling/" target="_blank">ASG</a>, and provision it with the new <code class="md-code md-code-inline">carnivore</code> AMI, <a href="https://ponyfoo.com/articles/immutable-deployments-packer">which you may recall from our last encounter</a>. The new <a href="http://aws.amazon.com/autoscaling/" target="_blank">ASG</a> will spin as many <a href="http://aws.amazon.com/ec2/" target="_blank">EC2</a> instances as desired. We&#x2019;ll wait until <a href="http://aws.amazon.com/ec2/" target="_blank">EC2</a> reports every one of those instances are properly initialized and healthy. We&#x2019;ll then wait until <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">ELB</a> reports every one of those instances is reachable via <code class="md-code md-code-inline">HTTP</code>. This ensures that no downtime occurs during our deployment. When every new instance is reachable on <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">ELB</a>, we&#x2019;ll remove the outdated <a href="http://aws.amazon.com/ec2/" target="_blank">EC2</a> instances from <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank">ELB</a> first, and <em>downscale the outdated ASG to 0</em>. This will cause the ASG to allow connection draining to kick in on the outdated instances, and terminate them afterwards. Once all of that is over, we delete the outdated ASG.</p> <blockquote> <p>This approach might not be blazing fast, but I&#x2019;ll take a speed bump over downtime any day.</p> </blockquote> <p>Of course, none of this would be feasible if spinning a new instance took 15 minutes while installing software that should&#x2019;ve been baked into an image. This is why creating those images was crucial. In a slow process such as this, baking images saves us much appreciated startup time.</p></div>

<div><h1 id="process-recap">Process Recap</h1> <p>The approach I propose involves two steps. A <code class="md-code md-code-inline">setup</code> step and a <code class="md-code md-code-inline">deploy</code> step.</p> <p>You are supposed to run the <code class="md-code md-code-inline">setup</code> script once, when first putting together a new environment. No images are involved for the <code class="md-code md-code-inline">setup</code> process, and it&#x2019;s very fast. Everything is done at the infrastructure level. The <code class="md-code md-code-inline">setup</code> script involves the following steps.</p> <ul> <li>Create an <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank" aria-label="Amazon Elastic Load Balancers">ELB</a></li> <li>Configure the <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank" aria-label="Amazon Elastic Load Balancers">ELB</a> with <a href="https://aws.amazon.com/blogs/aws/elb-connection-draining-remove-instances-from-service-with-care/" target="_blank" aria-label="ELB Connection Draining &#x2013; Remove Instances From Service With Care">connection draining</a></li> <li>Set up health checks on the <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank" aria-label="Amazon Elastic Load Balancers">ELB</a></li> <li>Create a <a href="http://aws.amazon.com/route53/" target="_blank" aria-label="Amazon Web Services Route 53">Route 53</a> <code class="md-code md-code-inline">ALIAS</code> record set pointing at the <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank" aria-label="Amazon Elastic Load Balancers">ELB</a></li> </ul> <p>The <code class="md-code md-code-inline">deployment</code> script is a bit longer. This is the one that you will be <em>(presumably)</em> running a few times a day. Most of the time is spent baking images and waiting for <a href="http://aws.amazon.com/ec2/" target="_blank" aria-label="Amazon Elastic Cloud Compute">EC2</a> instances to become healthy in the eyes of <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank" aria-label="Amazon Elastic Load Balancers">ELB</a>.</p> <ul> <li>Bake <code class="md-code md-code-inline">primal</code> image with <a href="https://packer.io/" target="_blank">Packer</a>, if needed <em>(on occasion)</em></li> <li>Build static assets and optimize for <code class="md-code md-code-inline">NODE_ENV</code></li> <li>Bake <code class="md-code md-code-inline">carnivore</code> image with <a href="https://packer.io/" target="_blank">Packer</a></li> <li>Create an <a href="http://aws.amazon.com/autoscaling/" target="_blank" aria-label="Amazon Auto Scaling Groups">ASG</a> with the desired amount of <code class="md-code md-code-inline">carnivore</code>-ready <a href="http://aws.amazon.com/ec2/" target="_blank" aria-label="Amazon Elastic Cloud Compute">EC2</a> instances</li> <li>Wait until all instances are marked as <code class="md-code md-code-inline">InService</code> and <code class="md-code md-code-inline">Healthy</code> on <a href="http://aws.amazon.com/ec2/" target="_blank" aria-label="Amazon Elastic Cloud Compute">EC2</a></li> <li>Wait until all instances are marked as <code class="md-code md-code-inline">InService</code> on <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank" aria-label="Amazon Elastic Load Balancers">ELB</a></li> <li>Detach all outdated <a href="http://aws.amazon.com/ec2/" target="_blank" aria-label="Amazon Elastic Cloud Compute">EC2</a> instances from <a href="http://aws.amazon.com/elasticloadbalancing/" target="_blank" aria-label="Amazon Elastic Load Balancers">ELB</a></li> <li>Scale any outdated <a href="http://aws.amazon.com/autoscaling/" target="_blank" aria-label="Amazon Auto Scaling Groups">ASG</a> down to <em>0 instances</em></li> <li>Wait until all outdated <a href="http://aws.amazon.com/ec2/" target="_blank" aria-label="Amazon Elastic Cloud Compute">EC2</a> instances are terminated</li> <li>Delete outdated <a href="http://aws.amazon.com/autoscaling/" target="_blank" aria-label="Amazon Auto Scaling Groups">ASG</a></li> <li>Deregister <code class="md-code md-code-inline">carnivore</code> AMI from AWS</li> <li>Delete <code class="md-code md-code-inline">carnivore</code> AMI snapshot from AWS</li> </ul> <p>Besides <a href="https://packer.io/" target="_blank">Packer</a>, which you should&#x2019;ve installed if you already went over <a href="https://ponyfoo.com/articles/immutable-deployments-packer" aria-label="Immutable Deployments and Packer">the previous article</a>, you&#x2019;ll need to have a few more tools laying around.</p> <h1 id="requisites">Requisites</h1> <p>The first of them is the <a href="http://aws.amazon.com/cli/" target="_blank" aria-label="Amazon Web Services Command-Line Interface">AWS CLI</a> <em>(needs Python if you don&#x2019;t have it already in your system)</em>, obviously.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">pip install awscli
aws configure
</code></pre> <p>We use two different tools to manipulate JSON data. One is <a href="http://stedolan.github.io/jq/" target="_blank" aria-label="jq is a lightweight and flexible command-line JSON processor">jq</a>, which is great for most use cases where we just need to select some data deep in a JSON structure. If you are on OSX, you can use <code class="md-code md-code-inline">brew</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">brew install jq
</code></pre> <p>The other JSON tool is <a href="https://github.com/ddopson/underscore-cli" target="_blank" aria-label="underscore-cli on GitHub">underscore-cli</a>, which is better suited for complex filtering, mapping, and selects.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm install underscore-cli --save-dev
</code></pre> <p>In the next couple of sections we&#x2019;ll go over each of the two scripts and how they work.</p> <h1 id="putting-together-the-setup-script">Putting together the <code class="md-code md-code-inline">setup</code> script</h1> <p>Okay, so it&#x2019;s Bash again!</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-shebang">#!/bin/bash
</span></code></pre> <p>Not sure how this is <em>not</em> the default behavior, but this command makes sure that your script doesn&#x2019;t behave as if <a href="http://www.vbforums.com/showthread.php?448401-Classic-VB-What-is-wrong-with-using-quot-On-Error-Resume-Next-quot" target="_blank" aria-label="What is wrong with using &apos;On Error Resume Next&apos;?">On Error Resume Next</a> was on <em>(for those unfamiliar with Basic, <code class="md-code md-code-inline">set -e</code> will make sure that errors are treated as such)</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-built_in">set</span> <span class="md-code-operator">-e</span>
</code></pre> <p>The following bit of code defaults <code class="md-code md-code-inline">NODE_ENV</code> to <code class="md-code md-code-inline">staging</code>, unless it&#x2019;s user-provided.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">: <span class="md-code-string">&quot;<span class="md-code-variable">${NODE_ENV:=&quot;staging&quot;}</span>&quot;</span>
</code></pre> <p>First off, let&#x2019;s pull up the <a href="https://console.aws.amazon.com/console/home" target="_blank" aria-label="AWS Management Console">Amazon Web Services Console</a> and find the <em>&#x201C;Hosted Zone ID&#x201D;</em> for the domain you want to use with this application. Then, paste it in the next line.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">HOSTED_ZONE=<span class="md-code-string">&quot;{{YOUR_HOSTED_ZONE_ID}}&quot;</span>
</code></pre> <p>Next we declare the top level domain that we&#x2019;re going to be using. This should&#x2019;ve already been configured on the Route 53 hosted zone. In my case, that&#x2019;s <code class="md-code md-code-inline">ponyfoo.com</code>, obviously.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">TLD=<span class="md-code-string">&quot;ponyfoo.com&quot;</span>
</code></pre> <p>As I <a href="https://ponyfoo.com/articles/immutable-deployments-packer" aria-label="Immutable Deployments and Packer">mentioned in the previous article</a>, <em>resources are named conventionally</em> so that names don&#x2019;t clash with each other. Let&#x2019;s set up a bunch of names we are going to use.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">NAME=<span class="md-code-string">&quot;ponyfoo-<span class="md-code-variable">$NODE_ENV</span>&quot;</span>
ELB_NAME=<span class="md-code-string">&quot;elb-<span class="md-code-variable">$NAME</span>&quot;</span>
ASG_NAME=<span class="md-code-string">&quot;asg-<span class="md-code-variable">$NAME</span>&quot;</span>
LC_NAME=<span class="md-code-string">&quot;lc-<span class="md-code-variable">$NAME</span>-initial&quot;</span>
KEYFILE=<span class="md-code-string">&quot;deploy/keys/<span class="md-code-variable">$NODE_ENV</span>&quot;</span>
</code></pre> <p>What <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html" target="_blank" aria-label="Regions and Availability Zones">availability zones</a> should be used by the load balancer? This is configurable, but stick to something that works! I originally wanted to pull these from the ELB, but <em>it seems not every instance type is available in every availability zone</em>, so your mileage may vary.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">AVAILABILITY_ZONES=<span class="md-code-string">&quot;us-east-1a us-east-1b&quot;</span>
</code></pre> <p>My convention is to use the naked domain when <code class="md-code md-code-inline">$NODE_ENV</code> is <code class="md-code md-code-inline">production</code> <em>(<code class="md-code md-code-inline">example.com</code> vs <code class="md-code md-code-inline">staging.example.com</code>)</em>, but you can easily change that.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">HOST_NAME=<span class="md-code-variable">$NODE_ENV</span><span class="md-code-string">&quot;.&quot;</span>

<span class="md-code-keyword">if</span> [ <span class="md-code-string">&quot;<span class="md-code-variable">$HOST_NAME</span>&quot;</span> == <span class="md-code-string">&quot;production.&quot;</span> ]
<span class="md-code-keyword">then</span>
  HOST_NAME=<span class="md-code-string">&quot;&quot;</span>
<span class="md-code-keyword">fi</span>
</code></pre> <p>One last bit of set up before we start to interact with Amazon. Most commands will be logged so that you can diagnose what happened in case something goes wrong. Here&#x2019;s where to look for them.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">rm -rf deploy/<span class="md-code-built_in">log</span>
mkdir deploy/<span class="md-code-built_in">log</span>
</code></pre> <p>Onto the exciting business. We start by creating the load balancer on the designated availability zones, and telling it to listen on port <code class="md-code md-code-inline">80</code>. We name the ELB according to our conventions so we can easily dig it up later on <em>(or inspect it in the AWS management console)</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: creating <span class="md-code-variable">$ELB_NAME</span> load balancer (<span class="md-code-variable">$AVAILABILITY_ZONES</span>)...&quot;</span>
aws elb create-load-balancer \
  --load-balancer-name <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_NAME</span>&quot;</span> \
  --listeners Protocol=HTTP,LoadBalancerPort=<span class="md-code-number">80</span>,InstanceProtocol=HTTP,InstancePort=<span class="md-code-number">80</span> \
  --availability-zones <span class="md-code-variable">$AVAILABILITY_ZONES</span> &gt; deploy/<span class="md-code-built_in">log</span>/elb-create.log
</code></pre> <p>Turn on <a href="https://aws.amazon.com/blogs/aws/elb-connection-draining-remove-instances-from-service-with-care/" target="_blank" aria-label="ELB Connection Draining &#x2013; Remove Instances From Service With Care">connection draining</a> on the load balancer we just created. Instances are given 300 seconds <em>(5 minutes)</em> after being detached from ELB to finish off outstanding requests. This is a crucial aspect of managing deployments gracefully. We&#x2019;ll shut off old instances from ELB but they&#x2019;ll still be allowed to complete those outstanding requests on their own terms.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: enabling connection draining on elb...&quot;</span>
aws elb modify-load-balancer-attributes \
  --load-balancer-name <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_NAME</span>&quot;</span> \
  --load-balancer-attributes <span class="md-code-string">&quot;{\&quot;ConnectionDraining\&quot;:{\&quot;Enabled\&quot;:true,\&quot;Timeout\&quot;:300}}&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/elb-draining.log
</code></pre> <p>Lastly, we&#x2019;ll set up health checks on ELB. You should set up a <code class="md-code md-code-inline">GET /api/status/health</code> endpoint in your application. All it needs to do is return a <code class="md-code md-code-inline">200 OK</code> status code in under 4 seconds <em>(<code class="md-code md-code-inline">Timeout</code>)</em>. Instance health is checked twice every minute <em>(<code class="md-code md-code-inline">Interval</code>)</em>. If an instance fails the health check twice <em>(<code class="md-code md-code-inline">UnhealthyThreshold</code>)</em>, it&#x2019;ll become unhealthy. When an instance is marked as unhealthy the autoscaling group will probably take it down, but nevertheless we&#x2019;ve configured a <em><code class="md-code md-code-inline">HealthyThreshold</code></em> of 2, which would bring it back to <em>Healthy</em> after two consecutive successful health checks.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: configuring health checks on elb...&quot;</span>
aws elb configure-health-check \
  --load-balancer-name <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_NAME</span>&quot;</span> \
  --health-check Target=HTTP:<span class="md-code-number">80</span>/api/status/health,Interval=<span class="md-code-number">30</span>,UnhealthyThreshold=<span class="md-code-number">2</span>,HealthyThreshold=<span class="md-code-number">2</span>,Timeout=<span class="md-code-number">4</span> &gt; deploy/<span class="md-code-built_in">log</span>/elb-health.log
</code></pre> <p>The next block of code queries Amazon about the ELB, getting the metadata we need to set up the <code class="md-code md-code-inline">ALIAS</code> record on our Route 53 hosted zone. This is necessary because the command to create the ELB doesn&#x2019;t yield information such as the hosted zone ID or the hosted zone name. Afterwards, we use that metadata to upsert an <code class="md-code md-code-inline">ALIAS</code> record on Route 53. We indicate that we want Route 53 to take the ELB health check into consideration.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: describing load balancer to create route53 alias recordset...&quot;</span>
aws elb describe-load-balancers \
  --load-balancer-name <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_NAME</span>&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/elb-describe-lb.log

ELB_ZONE_ID=$(jq -r <span class="md-code-string">&apos;.LoadBalancerDescriptions[0].CanonicalHostedZoneNameID&apos;</span> &lt; deploy/<span class="md-code-built_in">log</span>/elb-describe-lb.log)
ELB_ZONE_NAME=$(jq -r <span class="md-code-string">&apos;.LoadBalancerDescriptions[0].CanonicalHostedZoneName&apos;</span> &lt; deploy/<span class="md-code-built_in">log</span>/elb-describe-lb.log)

<span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: creating route53 alias recordset on <span class="md-code-variable">$HOST_NAME</span><span class="md-code-variable">$TLD</span>...&quot;</span>
<span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;{
  \&quot;Changes\&quot;: [{
    \&quot;Action\&quot;: \&quot;UPSERT\&quot;,
    \&quot;ResourceRecordSet\&quot;: {
      \&quot;Type\&quot;: \&quot;A\&quot;,
      \&quot;Name\&quot;: \&quot;<span class="md-code-variable">$HOST_NAME</span><span class="md-code-variable">$TLD</span>.\&quot;,
      \&quot;AliasTarget\&quot;: {
        \&quot;HostedZoneId\&quot;: \&quot;<span class="md-code-variable">$ELB_ZONE_ID</span>\&quot;,
        \&quot;DNSName\&quot;: \&quot;<span class="md-code-variable">$ELB_ZONE_NAME</span>\&quot;,
        \&quot;EvaluateTargetHealth\&quot;: true
      }
    }
  }]
}&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/route53-record-set-changes.log

aws route53 change-resource-record-sets \
  --hosted-zone-id <span class="md-code-string">&quot;<span class="md-code-variable">$HOSTED_ZONE</span>&quot;</span> \
  --change-batch <span class="md-code-string">&quot;file://deploy/log/route53-record-set-changes.log&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/route53-change-recordset.log
</code></pre> <p>Finally, we check to see if an <code class="md-code md-code-inline">ssh</code> key file was already generated and uploaded to Amazon, and if not, we upload one. The key file will grant <code class="md-code md-code-inline">ssh</code> access to every EC2 instance assigned to the current <code class="md-code md-code-inline">$NODE_ENV</code> environment. You are welcome to omit this step if you don&#x2019;t plan on needing to <code class="md-code md-code-inline">ssh</code> into instances to debug an issue. I don&#x2019;t usually do it, but I like knowing I can <em>(so that I can pinpoint what is going on, directly from a production instance)</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-keyword">if</span> [ <span class="md-code-operator">-f</span> <span class="md-code-string">&quot;<span class="md-code-variable">$KEYFILE</span>&quot;</span> ]
<span class="md-code-keyword">then</span>
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: ssh key file already exists on aws.&quot;</span>
<span class="md-code-keyword">else</span>
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: ssh key file doesn&apos;t exist yet. creating...&quot;</span>
  mkdir -p deploy/keys
  ssh-keygen -t rsa -b <span class="md-code-number">4096</span> -N <span class="md-code-string">&quot;&quot;</span> <span class="md-code-operator">-f</span> <span class="md-code-string">&quot;<span class="md-code-variable">$KEYFILE</span>&quot;</span>
  aws ec2 import-key-pair \
    --key-name <span class="md-code-string">&quot;<span class="md-code-variable">$NAME</span>&quot;</span> \
    --public-key-material <span class="md-code-string">&quot;file://<span class="md-code-variable">$KEYFILE</span>.pub&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/ec2-upload-keypair.log
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: ssh key file uploaded to aws.&quot;</span>
<span class="md-code-keyword">fi</span>

<span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;setup: done.&quot;</span>
</code></pre> <blockquote> <p><strong>&#x201C;If you need to <code class="md-code md-code-inline">ssh</code> into an instance that means your automation has failed&#x201D;</strong></p> <p>This is something I&#x2019;ve read many times, and while it&#x2019;s a great idea not to depend on <code class="md-code md-code-inline">ssh</code> for any of our process, it&#x2019;s still valuable being <em>able</em> to <code class="md-code md-code-inline">ssh</code> in case something goes wrong. You wouldn&#x2019;t want to be left stranded on principle, would you?</p> </blockquote> <p>That&#x2019;s all there is to the <code class="md-code md-code-inline">setup</code> script. We now have an ELB, and it&#x2019;s linked to our hosted zone on Route 53. At this point if we were to navigate to <code class="md-code md-code-inline">ponyfoo.com</code> we&#x2019;d see a blank page, served by ELB when no healthy instances are available.</p> <h1 id="a-immutable-deploy-script">A immutable <code class="md-code md-code-inline">deploy</code> script</h1> <p>Again, this is Bash. We don&#x2019;t tolerate mistakes. And we default <code class="md-code md-code-inline">NODE_ENV</code> to <code class="md-code md-code-inline">staging</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-shebang">#!/bin/bash
</span>
<span class="md-code-built_in">set</span> <span class="md-code-operator">-e</span>

: <span class="md-code-string">&quot;<span class="md-code-variable">${NODE_ENV:=&quot;staging&quot;}</span>&quot;</span>
</code></pre> <p>The 3 variables below allow us to make the deployment process faster. Setting <code class="md-code md-code-inline">PRIMAL_ID</code> means we&#x2019;ll skip the creation of a <code class="md-code md-code-inline">primal</code> image. Most of the time we don&#x2019;t want to create a new <code class="md-code md-code-inline">primal</code> image, but sometimes we might want to <em>(especially during debugging)</em>. Setting <code class="md-code md-code-inline">IMAGE_ID</code> means we&#x2019;ll also skip the creation of a <code class="md-code md-code-inline">carnivore</code> image. Most of the time we want to <strong>create new <code class="md-code md-code-inline">carnivore</code> images for each deployment</strong>, but it&#x2019;s useful to skip that step during debugging as <em>it&#x2019;ll save considerable time</em>.</p> <p>Finally, <code class="md-code md-code-inline">CLEANUP=&quot;no&quot;</code> means that the <code class="md-code md-code-inline">IMAGE_ID</code> AMI shouldn&#x2019;t be removed after a deployment. By default, those AMI are deleted only if they were produced by the deployment script, but you can set <code class="md-code md-code-inline">CLEANUP=&quot;no&quot;</code> to make sure that they&#x2019;re never deleted.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-comment"># PRIMAL_ID=&quot;ami-xxxxxxxx&quot;</span>
<span class="md-code-comment"># IMAGE_ID=&quot;ami-xxxxxxxx&quot;</span>
<span class="md-code-comment"># CLEANUP=&quot;no&quot;</span>
</code></pre> <p>Some more configuration follows. We choose the type of instances we want, how many we&#x2019;d like, the minimum and maximum capacity to be defined for the <a href="http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WorkingWithLaunchConfig.html" target="_blank" aria-label="Creating Launch Configurations">ASG launch configuration</a>, and whatnot.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">INSTANCE_TYPE=<span class="md-code-string">&quot;t1.micro&quot;</span>
INSTANCE_USER=<span class="md-code-string">&quot;admin&quot;</span>
SECURITY_GROUP=<span class="md-code-string">&quot;default&quot;</span>
MIN_CAPACITY=<span class="md-code-string">&quot;1&quot;</span>
MAX_CAPACITY=<span class="md-code-string">&quot;2&quot;</span>
DESIRED_CAPACITY=<span class="md-code-string">&quot;1&quot;</span>
</code></pre> <p>A couple of helper methods to get a timestamp like <code class="md-code md-code-inline">20150407181937</code> and to sleep for a short while and print a single dot.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">timestamp</span></span> () {
  date +<span class="md-code-string">&quot;%Y%m%d%H%M%S&quot;</span>
}

<span class="md-code-function"><span class="md-code-title">zzz</span></span> () {
  sleep <span class="md-code-number">2</span>
  <span class="md-code-built_in">printf</span> <span class="md-code-string">&quot;.&quot;</span>
}
</code></pre> <p>This may not be your convention, but you should do whatever static asset building you need inside this function. In my case I have <code class="md-code md-code-inline">npm run</code> scripts like <code class="md-code md-code-inline">build-staging</code> and <code class="md-code md-code-inline">build-production</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">build_app</span></span> () {
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: building app for <span class="md-code-variable">$NODE_ENV</span> environment&quot;</span>
  npm run build-<span class="md-code-variable">$NODE_ENV</span>
}
</code></pre> <p>The <code class="md-code md-code-inline">copy_over</code> method copies everything that&#x2019;s needed to run the application in a hosted environment, so this will probably be different from case to case. This should probably include the static assets you built in the <code class="md-code md-code-inline">build_app</code> method. Going back to <a href="https://ponyfoo.com/articles/immutable-deployments-packer" aria-label="Immutable Deployments and Packer">the previous article</a>, you might remember that <code class="md-code md-code-inline">carnivore</code> images took an entire directory from <code class="md-code md-code-inline">tmp/appserver</code>. In <code class="md-code md-code-inline">copy_over</code>, we prepare that directory with everything the image will need.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">copy_over</span></span> () {
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: copying files over to tmp/appserver&quot;</span>
  rm -rf tmp/appserver
  mkdir -p tmp/appserver/deploy/env
  cp -r {.bin,client,controllers,lib,models,resources,services,views,.env.defaults.json,.taunusrc,*.js,package.json} tmp/appserver
  cp <span class="md-code-string">&quot;deploy/env/<span class="md-code-variable">$NODE_ENV</span>.json&quot;</span> tmp/appserver/deploy/env
}
</code></pre> <p>The following couple of functions are in charge of conditionally building the base <code class="md-code md-code-inline">primal</code> image with Packer. As you can see we won&#x2019;t be passing <code class="md-code md-code-inline">packer</code> any configuration besides what <a href="https://ponyfoo.com/articles/immutable-deployments-packer" aria-label="Immutable Deployments and Packer">we set up last time around</a>. The only condition that&#x2019;s checked is whether a <code class="md-code md-code-inline">PRIMAL_ID</code> variable is set. The <code class="md-code md-code-inline">tee</code> command writes to a file while still printing to standard output.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">build_primal_image</span></span> () {
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: building primal image with packer...&quot;</span>

  cp package.json deploy/mailtube
  packer build \
    deploy/templates/primal.json | tee deploy/<span class="md-code-built_in">log</span>/packer-primal.log

  PRIMAL_ID=$(tail -<span class="md-code-number">1</span> &lt; deploy/<span class="md-code-built_in">log</span>/packer-primal.log | cut <span class="md-code-operator">-d</span> <span class="md-code-string">&apos; &apos;</span> <span class="md-code-operator">-f</span> <span class="md-code-number">2</span>)

  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: built image <span class="md-code-variable">$PRIMAL_ID</span>&quot;</span>
}

<span class="md-code-function"><span class="md-code-title">maybe_build_primal_image</span></span> () {
  <span class="md-code-keyword">if</span> [ -z <span class="md-code-variable">${PRIMAL_ID+x}</span> ]
  <span class="md-code-keyword">then</span>
    build_primal_image
  <span class="md-code-keyword">else</span>
    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: skipping primal image build, using <span class="md-code-variable">$PRIMAL_ID</span>&quot;</span>
  <span class="md-code-keyword">fi</span>
}
</code></pre> <p>Building <code class="md-code md-code-inline">carnivore</code> needs us to prepare the application for the relevant environment, first. This time we&#x2019;ll forward a couple of variables to <code class="md-code md-code-inline">packer</code>. The <code class="md-code md-code-inline">NODE_ENV</code> variable will be used when booting up our application so that the image knows what environment it should expect to be running on. The <code class="md-code md-code-inline">SOURCE_ID</code> variable will be used in the template to base our deployment image off of the <code class="md-code md-code-inline">SOURCE_ID</code> AMI.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">build_deployment_image</span></span> () {
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: building carnivore image with packer...&quot;</span>

  packer build \
    -var NODE_ENV=<span class="md-code-variable">$NODE_ENV</span> \
    -var SOURCE_ID=<span class="md-code-variable">$PRIMAL_ID</span> \
    deploy/templates/carnivore.json | tee deploy/<span class="md-code-built_in">log</span>/packer-carnivore.log

  IMAGE_ID=$(tail -<span class="md-code-number">1</span> &lt; deploy/<span class="md-code-built_in">log</span>/packer-carnivore.log | cut <span class="md-code-operator">-d</span> <span class="md-code-string">&apos; &apos;</span> <span class="md-code-operator">-f</span> <span class="md-code-number">2</span>)

  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: built image <span class="md-code-variable">$IMAGE_ID</span>&quot;</span>
}

<span class="md-code-function"><span class="md-code-title">maybe_build_deployment_image</span></span> () {
  <span class="md-code-keyword">if</span> [ -z <span class="md-code-variable">${IMAGE_ID+x}</span> ]
  <span class="md-code-keyword">then</span>
    build_app
    copy_over
    build_deployment_image
  <span class="md-code-keyword">else</span>
    CLEANUP=<span class="md-code-string">&quot;no&quot;</span>
    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: skipping deployment image build, using <span class="md-code-variable">$IMAGE_ID</span>&quot;</span>
  <span class="md-code-keyword">fi</span>
}
</code></pre> <p>The <code class="md-code md-code-inline">lookup_existing_asg</code> method records pre-existing autoscaling groups and launch configurations so that they can be turned off and deleted later on. The conditionals near the bottom are used to avoid creating a file with a single new line, as that would result in problems when reading that file line by line later on in the process.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">lookup_existing_asg</span></span> () {
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: pulling down list of existing autoscaling groups...&quot;</span>
  aws autoscaling describe-auto-scaling-groups &gt; deploy/<span class="md-code-built_in">log</span>/asg-list.log

  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: pulling down list of existing launch configurations...&quot;</span>
  aws autoscaling describe-launch-configurations &gt; deploy/<span class="md-code-built_in">log</span>/asg-lc.log

  EXISTING_GROUP_NAMES=$(underscore process --outfmt text <span class="md-code-string">&quot;data.AutoScalingGroups.filter(function (asg) {
    return asg.AutoScalingGroupName.indexOf(\&quot;asg-<span class="md-code-variable">$NAME</span>\&quot;) === 0
  }).map(function (asg) {
    return asg.AutoScalingGroupName
  })&quot;</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-list.log)

  EXISTING_LAUNCH_CONFIGURATIONS=$(underscore process --outfmt text <span class="md-code-string">&quot;data.LaunchConfigurations.filter(function (lc) {
    return lc.LaunchConfigurationName.indexOf(\&quot;lc-<span class="md-code-variable">$NAME</span>\&quot;) === 0
  }).map(function (lc) {
    return lc.LaunchConfigurationName
  })&quot;</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-lc.log)

  <span class="md-code-keyword">if</span> [ <span class="md-code-string">&quot;<span class="md-code-variable">$EXISTING_GROUP_NAMES</span>&quot;</span> != <span class="md-code-string">&quot;&quot;</span> ]
  <span class="md-code-keyword">then</span>
    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;<span class="md-code-variable">$EXISTING_GROUP_NAMES</span>&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/asg-existing-group-names.log
  <span class="md-code-keyword">else</span>
    touch deploy/<span class="md-code-built_in">log</span>/asg-existing-group-names.log
  <span class="md-code-keyword">fi</span>

  <span class="md-code-keyword">if</span> [ <span class="md-code-string">&quot;<span class="md-code-variable">$EXISTING_LAUNCH_CONFIGURATIONS</span>&quot;</span> != <span class="md-code-string">&quot;&quot;</span> ]
  <span class="md-code-keyword">then</span>
    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;<span class="md-code-variable">$EXISTING_LAUNCH_CONFIGURATIONS</span>&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/asg-existing-lc.log
  <span class="md-code-keyword">else</span>
    touch deploy/<span class="md-code-built_in">log</span>/asg-existing-lc.log
  <span class="md-code-keyword">fi</span>
}
</code></pre> <p>In <code class="md-code md-code-inline">launch_updated_asg</code> we&#x2019;ll create <a href="http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WorkingWithLaunchConfig.html" target="_blank" aria-label="Creating Launch Configurations">a new launch configuration</a>, and use that to create a new <a href="http://aws.amazon.com/autoscaling/" target="_blank" aria-label="Amazon Auto Scaling Groups">ASG</a>. The <a href="http://aws.amazon.com/autoscaling/" target="_blank" aria-label="Amazon Auto Scaling Groups">ASG</a> is configured to ask the ELB for health reports, to tag instances according to the current environment, to use the availability zones registered with the ELB, and to launch as many instances as we&#x2019;ve configured earlier. The ASG is in charge of creating the instances, so this is the only place where you need to change <em>how</em> they will be created. Generally speaking you won&#x2019;t ever need to create instances yourself unless you are in some kind of obscure hurry. If you want more instances you should <strong>just increase the desired and maximum capacity</strong> in the ASG.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">launch_updated_asg</span></span> () {
  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: describing elb to get availability zones...&quot;</span>
  aws elb describe-load-balancers \
    --load-balancer-name <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_NAME</span>&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/elb-describe-lb.log

  AVAILABILITY_ZONES=$(jq -r <span class="md-code-string">&quot;.LoadBalancerDescriptions[0].AvailabilityZones[]?&quot;</span> &lt; deploy/<span class="md-code-built_in">log</span>/elb-describe-lb.log)

  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: creating <span class="md-code-variable">$LC_NAME</span> using the latest image...&quot;</span>
  aws autoscaling create-launch-configuration \
    --launch-configuration-name <span class="md-code-string">&quot;<span class="md-code-variable">$LC_NAME</span>&quot;</span> \
    --image-id <span class="md-code-string">&quot;<span class="md-code-variable">$IMAGE_ID</span>&quot;</span> \
    --instance-type <span class="md-code-string">&quot;<span class="md-code-variable">$INSTANCE_TYPE</span>&quot;</span> \
    --key-name <span class="md-code-string">&quot;<span class="md-code-variable">$NAME</span>&quot;</span> \
    --security-groups <span class="md-code-string">&quot;<span class="md-code-variable">$SECURITY_GROUP</span>&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/asg-lc-creation.log

  <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: creating <span class="md-code-variable">$ASG_NAME</span> autoscaling group...&quot;</span>
  aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name <span class="md-code-string">&quot;<span class="md-code-variable">$ASG_NAME</span>&quot;</span> \
    --launch-configuration-name <span class="md-code-string">&quot;<span class="md-code-variable">$LC_NAME</span>&quot;</span> \
    --availability-zones <span class="md-code-variable">$AVAILABILITY_ZONES</span> \
    --health-check-type <span class="md-code-string">&quot;ELB&quot;</span> \
    --health-check-grace-period <span class="md-code-number">300</span> \
    --load-balancer-names <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_NAME</span>&quot;</span> \
    --min-size <span class="md-code-string">&quot;<span class="md-code-variable">$MIN_CAPACITY</span>&quot;</span> \
    --max-size <span class="md-code-string">&quot;<span class="md-code-variable">$MAX_CAPACITY</span>&quot;</span> \
    --desired-capacity <span class="md-code-string">&quot;<span class="md-code-variable">$DESIRED_CAPACITY</span>&quot;</span> \
    --tags ResourceId=<span class="md-code-variable">$ASG_NAME</span>,Key=Name,Value=<span class="md-code-variable">$NAME</span> ResourceId=<span class="md-code-variable">$ASG_NAME</span>,Key=Role,Value=web &gt; deploy/<span class="md-code-built_in">log</span>/asg-create-group.log
}
</code></pre> <p>Once the new <a href="http://aws.amazon.com/autoscaling/" target="_blank" aria-label="Amazon Auto Scaling Groups">ASG</a> has been created, we need to wait for EC2 and ELB to report that all of the instances are healthy and reachable through the load balancer. We mix polling and short naps to avoid spamming the API, while not taking too long to react to health status changes either.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">wait_on_health</span></span> () {
  EC2_HEALTH=<span class="md-code-string">&quot;0&quot;</span>
  <span class="md-code-keyword">while</span> [ <span class="md-code-string">&quot;<span class="md-code-variable">$EC2_HEALTH</span>&quot;</span> != <span class="md-code-string">&quot;<span class="md-code-variable">$DESIRED_CAPACITY</span>&quot;</span> ]
  <span class="md-code-keyword">do</span>
    <span class="md-code-built_in">printf</span> <span class="md-code-string">&quot;deploy: ensuring new instance(s) are healthy at ec2&quot;</span>
    zzz;zzz
    aws autoscaling describe-auto-scaling-groups \
      --auto-scaling-group-names <span class="md-code-string">&quot;<span class="md-code-variable">$ASG_NAME</span>&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/asg-description.log

    EC2_HEALTH=$(underscore process --outfmt text <span class="md-code-string">&quot;data.AutoScalingGroups[0].Instances.filter(function (i) {
      return i.LifecycleState === &apos;InService&apos; &amp;&amp; i.HealthStatus === &apos;Healthy&apos;
    }).length&quot;</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-description.log)

    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot; (<span class="md-code-variable">$EC2_HEALTH</span>/<span class="md-code-variable">$DESIRED_CAPACITY</span> are healthy)&quot;</span>
  <span class="md-code-keyword">done</span>

  ELB_INSTANCES=$(jq -r <span class="md-code-string">&apos;.AutoScalingGroups[0].Instances[]?.InstanceId&apos;</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-description.log)
  ELB_HEALTH=<span class="md-code-string">&quot;0&quot;</span>
  <span class="md-code-keyword">while</span> [ <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_HEALTH</span>&quot;</span> != <span class="md-code-string">&quot;<span class="md-code-variable">$DESIRED_CAPACITY</span>&quot;</span> ]
  <span class="md-code-keyword">do</span>
    <span class="md-code-built_in">printf</span> <span class="md-code-string">&quot;deploy: ensuring new instance(s) are healthy at elb&quot;</span>
    zzz;zzz
    aws elb describe-instance-health \
      --load-balancer-name <span class="md-code-string">&quot;<span class="md-code-variable">$ELB_NAME</span>&quot;</span> \
      --instances <span class="md-code-variable">$ELB_INSTANCES</span>  &gt; deploy/<span class="md-code-built_in">log</span>/elb-health-description.log

    ELB_HEALTH=$(underscore process --outfmt text <span class="md-code-string">&quot;data.InstanceStates.filter(function (s) {
      return s.State === &apos;InService&apos;
    }).length&quot;</span> &lt; deploy/<span class="md-code-built_in">log</span>/elb-health-description.log)

    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot; (<span class="md-code-variable">$ELB_HEALTH</span>/<span class="md-code-variable">$DESIRED_CAPACITY</span> are healthy)&quot;</span>
  <span class="md-code-keyword">done</span>
}
</code></pre> <p>Now that our new instances are up and running, we can safely remove outdated instances from the ELB. We iterate over all the ASG that predated our deployment, and deregister their instances. Then, we also <em>downscale those ASG to 0</em>, effectively asking them politely to gracefully shut down all of their instances. After all of them have been terminated, we shut down the ASG itself. Then, we delete all outdated launch configurations.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">cleanup_outdated_autoscaling_groups</span></span> () {
  <span class="md-code-keyword">while</span> <span class="md-code-built_in">read</span> EXISTING_GROUP_NAME
  <span class="md-code-keyword">do</span>
    ASG_INSTANCES=$(underscore process --outfmt text <span class="md-code-string">&quot;data.AutoScalingGroups.filter(function (asg,i) {
      return asg.AutoScalingGroupName === \&quot;<span class="md-code-variable">$EXISTING_GROUP_NAME</span>\&quot;
    }).shift().Instances.map(function (i) {
      return i.InstanceId
    })&quot;</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-list.log)

    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: removing instances in outdated <span class="md-code-variable">$EXISTING_GROUP_NAME</span> from <span class="md-code-variable">$ELB_NAME</span>...&quot;</span>
    aws elb deregister-instances-from-load-balancer \
      --load-balancer-name <span class="md-code-variable">$ELB_NAME</span> \
      --instances <span class="md-code-variable">$ASG_INSTANCES</span> &gt; deploy/<span class="md-code-built_in">log</span>/elb-deregister.log

    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: downscaling outdated <span class="md-code-variable">$EXISTING_GROUP_NAME</span>...&quot;</span>
    aws autoscaling update-auto-scaling-group \
      --auto-scaling-group-name <span class="md-code-variable">$EXISTING_GROUP_NAME</span> \
      --max-size <span class="md-code-number">0</span> \
      --min-size <span class="md-code-number">0</span> &gt; deploy/<span class="md-code-built_in">log</span>/asg-downscale.log

    OPERATIONAL=<span class="md-code-string">&quot;1&quot;</span>
    <span class="md-code-keyword">while</span> [ <span class="md-code-string">&quot;<span class="md-code-variable">$OPERATIONAL</span>&quot;</span> != <span class="md-code-string">&quot;0&quot;</span> ]
    <span class="md-code-keyword">do</span>
      <span class="md-code-built_in">printf</span> <span class="md-code-string">&quot;deploy: ensuring outdated instance(s) are terminated&quot;</span>
      zzz;zzz
      aws autoscaling describe-auto-scaling-groups \
        --auto-scaling-group-names <span class="md-code-string">&quot;<span class="md-code-variable">$EXISTING_GROUP_NAME</span>&quot;</span> &gt; deploy/<span class="md-code-built_in">log</span>/asg-existing-description.log

      OPERATIONAL=$(underscore process --outfmt text <span class="md-code-string">&quot;data.AutoScalingGroups.filter(function (asg) {
        return asg.AutoScalingGroupName === \&quot;<span class="md-code-variable">$EXISTING_GROUP_NAME</span>\&quot;
      }).shift().Instances.length&quot;</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-existing-description.log)

      <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot; (<span class="md-code-variable">$OPERATIONAL</span> are operational)&quot;</span>
   <span class="md-code-keyword">done</span>

    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: deleting outdated <span class="md-code-variable">$EXISTING_GROUP_NAME</span>...&quot;</span>
    aws autoscaling delete-auto-scaling-group \
      --auto-scaling-group-name <span class="md-code-variable">$EXISTING_GROUP_NAME</span> || <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: delete failed. maybe it&apos;s already deleted.&quot;</span>
  <span class="md-code-keyword">done</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-existing-group-names.log

  <span class="md-code-keyword">while</span> <span class="md-code-built_in">read</span> EXISTING_LC_NAME
  <span class="md-code-keyword">do</span>
    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: removing outdated launch configuration <span class="md-code-variable">$EXISTING_LC_NAME</span>...&quot;</span>
    aws autoscaling delete-launch-configuration \
      --launch-configuration-name <span class="md-code-string">&quot;<span class="md-code-variable">$EXISTING_LC_NAME</span>&quot;</span> &gt;&gt; deploy/<span class="md-code-built_in">log</span>/asg-lc-deletion.log || <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: delete failed. maybe it&apos;s already deleted.&quot;</span>
  <span class="md-code-keyword">done</span> &lt; deploy/<span class="md-code-built_in">log</span>/asg-existing-lc.log
}
</code></pre> <p>The <code class="md-code md-code-inline">cleanup_deployment_image</code> method deregisters the <code class="md-code md-code-inline">$IMAGE_ID</code> AMI and deletes the associated snapshot, making sure that our deployment leaves no stray artifacts on its way.</p> <pre class="md-code-block"><code class="md-code md-lang-bash"><span class="md-code-function"><span class="md-code-title">cleanup_deployment_image</span></span> () {
  <span class="md-code-keyword">if</span> [ <span class="md-code-string">&quot;<span class="md-code-variable">$CLEANUP</span>&quot;</span> != <span class="md-code-string">&quot;no&quot;</span> ]
  <span class="md-code-keyword">then</span>
    SNAPSHOT_ID=$(aws ec2 describe-images \
      --image-ids <span class="md-code-variable">$IMAGE_ID</span> \
      | jq -r .Images[<span class="md-code-number">0</span>].BlockDeviceMappings[<span class="md-code-number">0</span>].Ebs.SnapshotId)

    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: deregistering deployment image <span class="md-code-variable">$IMAGE_ID</span>&quot;</span>
    aws ec2 deregister-image --image-id <span class="md-code-variable">$IMAGE_ID</span>

    <span class="md-code-built_in">echo</span> <span class="md-code-string">&quot;deploy: deleting snapshot <span class="md-code-variable">$SNAPSHOT_ID</span>&quot;</span>
    aws ec2 delete-snapshot --snapshot-id <span class="md-code-variable">$SNAPSHOT_ID</span>
  <span class="md-code-keyword">fi</span>
}
</code></pre> <p>Lastly, we run all the methods we&#x2019;ve declared. You probably won&#x2019;t have to touch this beyond changing the <code class="md-code md-code-inline">$NAME</code> prefix from <code class="md-code md-code-inline">ponyfoo</code> to <code class="md-code md-code-inline">something-else</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">STAMP=<span class="md-code-string">&quot;<span class="md-code-variable">$(timestamp)</span>&quot;</span>
NAME=<span class="md-code-string">&quot;ponyfoo-<span class="md-code-variable">$NODE_ENV</span>&quot;</span>
ELB_NAME=<span class="md-code-string">&quot;elb-<span class="md-code-variable">$NAME</span>&quot;</span>
KEYFILE=<span class="md-code-string">&quot;deploy/keys/<span class="md-code-variable">$NODE_ENV</span>&quot;</span>
ASG_NAME=<span class="md-code-string">&quot;asg-<span class="md-code-variable">$NAME</span>-<span class="md-code-variable">$STAMP</span>&quot;</span>
LC_NAME=<span class="md-code-string">&quot;lc-<span class="md-code-variable">$NAME</span>-<span class="md-code-variable">$STAMP</span>&quot;</span>

rm -rf deploy/<span class="md-code-built_in">log</span>
mkdir deploy/<span class="md-code-built_in">log</span>
maybe_build_primal_image
maybe_build_deployment_image
lookup_existing_asg
launch_updated_asg
<span class="md-code-built_in">wait</span>_on_health
cleanup_outdated_autoscaling_groups
cleanup_deployment_image
</code></pre> <p><strong>Finally!</strong> That&#x2019;s <em>it</em>.</p> <h1 id="conclusion">Conclusion</h1> <p>I think one of the best aspects of this deployment process is how easy it is to set up. I designed it for another project but I managed to cram it into <code class="md-code md-code-inline">ponyfoo.com</code> in under 10 minutes. The coolest part is that it really isn&#x2019;t tied to one technology in particular, and besides the environment being defined as <code class="md-code md-code-inline">NODE_ENV</code> there isn&#x2019;t a lot of <code class="md-code md-code-inline">node</code>-specific code in the deployment scripts, as most of that is contained in the Packer image provisioning scripts.</p> <p>This separation of concerns, where you build your static assets in your local environment, you bake an AMI with everything your hosted environments need, and you perform deployments in an application-agnostic way, makes deployments with this setup a breeze. <em>At least for me!</em></p> <p><strong>Happy Bashing!</strong></p> <p><sub><em><a href="https://github.com/bevacqua/baal" target="_blank" aria-label="bevacqua/baal on GitHub">You&#x2019;ll find a full copy of the scripts outlined in these articles on GitHub</a></em>.</sub></p></div>
