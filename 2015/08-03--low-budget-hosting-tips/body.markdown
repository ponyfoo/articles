# Add Some Swap Space!

In AWS, [`t1.micro`][1] instances have around **8GB** of disk space while they only have around **800M** of RAM. That's very low RAM. I've incorporated the snippet below into my deployments to add **2GB** of _swap space_, so that when the instance runs out of RAM, it can fall back to using the disk. Albeit slower, it's often enough to get you there -- considering you're using a `t1.micro` instance or equivalent on another platform, you're probably hosting a blog such as this one, a personal website, or a similarly low-budget project.

The script below will add **2GB** of swap space on disk, mount it for immediate use, and make it so that it _works across restarts_.

```bash
echo "creating swap space"
sudo dd if=/dev/zero of=/media/swapfile.img bs=1024 count=2M
sudo mkswap /media/swapfile.img
sudo chmod 0600 /media/swapfile.img
echo "/media/swapfile.img swap swap sw 0 0" | sudo tee -a /etc/fstab
sudo swapon /media/swapfile.img
```

I've struggled with this one for quite a bit, because Pony Foo didn't originally consume as much memory, but after throwing in `nginx`, `io.js`, a cluster of Node instances, and a `cron` job that runs every 6 minutes, memory ramped up. It took me a while to figure out, and the biggest issue was that AWS CloudWatch _(their monitoring system)_ doesn't track memory consumption by default, so the _problem didn't immediately stand out_. The `node` processes would just <mark>**hang, no error message or anything, and stop serving responses**</mark>. Meanwhile CPU would ramp up to _**100% utilization**_, rendering the instance useless for a couple of minutes.

![CloudWatch displaying CPU utilization spikes][2]

> Luckily, adding those **2GB** of swap memory promptly fixed the issue!

# Keep your Cluster in Your Cores

A similar issue was rooted in the `cron` job. After fixing the memory issues, I added in a very simple `cron` job written in Node that would figure out whether there's any articles scheduled for publication and publishes them if their publication date was in the past.

As soon as I deployed, I noticed that the CPU would spike every six minutes. Suspiciously enough, that's how often I had set up the `cron` job to run. After some thinking, I realized what was going on. As I mentioned earlier, I was already running a cluster on the instance, with [`os.cpus().length`][3] workers or `2`, whichever was largest. Throwing in **an extra Node instance crippled the CPU**. Yes, that _surprised me a bit_ too, the job runs for like 6 seconds most of the time. My work around was to place the scheduler in an API endpoint and use `curl` against the cluster in the `cron` job.

This had _a couple of benefits_.

- CPU usage doesn't spike to 100% anymore
- Job runs faster because it doesn't have to parse and compile any JavaScript, nor open any database connections

Nowadays CPU utilization hardly ever goes over **30%**. Considering this is a [`t1.micro`][4] instance, that's low utilization. _High fives all around!_

![CPU utilization today for ponyfoo.com][5]

It does have the drawback that the job **could be executed by an unwanted party**, but that could be easily mitigated by doing some awkward dance where we verify that the requester already had prior access to the server file system anyways. That being said, this job in particular happens to be idempotent -- _running it over and over again won't change the outcome_ -- and thus, **not a big deal that people can help me** check if an article slated for publication can already be published.

# Free-tier*ize* Your One-offs

If you have one-offs that need to be executed every once in a while, such as the `cron` job I've just described, maybe you could spin up _free-tier instances_ on a PaaS provider to run those jobs. It's _a little extra work_, but it lightens the load on your application servers and **spares the CPU**!

# Monitor All the Things

When you're running a website on a budget, monitoring plays an important role. Your site might go out of memory, grind to a halt and no longer be able to serve any responses, crash, time out on the database connection, and many other <mark>risk factors are always lurking around</mark>.

I have a **love/hate** relationship with [Uptime Robot][6]. On the one hand, it's a great service that pings my sites every few minutes and emails me when they go down _(and back up)_. On the other hand, getting those _"ponyfoo.com is down"_ emails **sucks**. Just look at those slick graphs, though!

[![Uptime Robot Dashboard for ponyfoo.com][7]][8]

Of course, you shouldn't rely solely on [Uptime Robot][9]. Add logging to your application. Use [`winston`][10] to log messages to your database whenever an error occurs during a request, **a worker in your cluster crashes**, and whenever _similarly scary events_ occur.

If you're on AWS, you might want to hook into the [CloudWatch API][11], or just head over to your dashboard and see if anything seems amiss.

![CloudWatch Metrics in the AWS Dashboard][12]

If you're not on AWS, you could always look to [New Relic] and similar "embedded" **APM** _(Application Performance Management)_ solutions. These usually take their own toll on performance -- much like _"anti-virus"_ software (which I hate) -- but they do have a noble purpose in thoroughly monitoring and providing you with real-time statistics and insight into the current state and load of your application.

> Hope that helps, I probably forgot to mention some obvious advice here, but this is mostly just off the top of my head!

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][13]._ Remember to **subscribe** if you got this far!

  [1]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts_micro_instances.html "T1 Micro Instances on Amazon Web Services"
  [2]: https://i.imgur.com/T67aDvn.png
  [3]: https://nodejs.org/api/os.html#os_os_cpus "Refer to os module documentation in the node.js manual"
  [4]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts_micro_instances.html "T1 Micro Instances on Amazon Web Services"
  [5]: https://i.imgur.com/fyqrqbH.png
  [6]: https://uptimerobot.com "Uptime Robot"
  [7]: https://i.imgur.com/bLZg1OU.png
  [8]: https://uptimerobot.com "Uptime Robot"
  [9]: https://uptimerobot.com "Uptime Robot"
  [10]: https://github.com/winstonjs/winston "winstonjs/winston on GitHub"
  [11]: http://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/Welcome.html "Amazon CloudWatch API Documentation"
  [12]: https://i.imgur.com/AxdQ1TO.png "Screen Shot 2015-08-04 at 13.14.27.png"
  [13]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
  [14]: http://newrelic.com/ "New Relic APM"
