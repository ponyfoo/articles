# Installing the Elastic Stack

I had used *Elasticsearch 2.3* for [my previous blog post][setup]. This represented a problem when it came to using the Elastic Stack, because I wanted to use the latest version of Kibana _-- `5.0.0-alpha3`, at the time of this writing._

As mentioned in the earlier article, we'll have to download and install Java 8 to get Elasticsearch up and running properly. Below is the piece of code we used to install Java 8. Keep in mind this code was tested on a Debian Jessie environment, but it should mostly work in Ubuntu or similar systems.

```bash
# install java 8
export JAVA_DIST=8u92-b14
export JAVA_RESOURCE=jdk-8u92-linux-x64.tar.gz
export JAVA_VERSION=jdk1.8.0_92
wget -nv --header "Cookie: oraclelicense=accept-securebackup-cookie" -O /tmp/java-jdk.tar.gz http://download.oracle.com/otn-pub/java/jdk/$JAVA_DIST/$JAVA_RESOURCE
sudo mkdir /opt/jdk
sudo tar -zxf /tmp/java-jdk.tar.gz -C /opt/jdk
sudo update-alternatives --install /usr/bin/java java /opt/jdk/$JAVA_VERSION/bin/java 100
sudo update-alternatives --install /usr/bin/javac javac /opt/jdk/$JAVA_VERSION/bin/javac 100
```

Next, we'll install the entire pre-release Elastic Stack _-- formerly ELK Stack_, for **E**lasticsearch, **L**ogstash and **K**ibana. The following piece of code pulls all three packages from `packages.elastic.co` and installs them.

```bash
# install elasticsearch + logstash + kibana
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://packages.elastic.co/elasticsearch/5.x/debian stable main" | sudo tee -a /etc/apt/sources.list
echo "deb https://packages.elastic.co/logstash/5.0/debian stable main" | sudo tee -a /etc/apt/sources.list
echo "deb https://packages.elastic.co/kibana/5.0.0-alpha/debian stable main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo apt-get -y install elasticsearch logstash kibana
```

After installing all three, we should turn on their services so that they run at startup. This is particularly desirable in production systems. Debian Jessie relies on `systemd` for services, so that's what we'll use to enable these services.

```bash
# elasticsearch + logstash + kibana as services
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
sudo /bin/systemctl enable logstash.service
sudo /bin/systemctl enable kibana.service
```

> You want to reduce your "hands on" 👏 production experience as much as possible. Automation is king, etc. Otherwise, why go through the trouble?

Lastly, we start all of the services and log their current status. We pause for ten seconds after booting Elasticsearch in order to give it time to start listening for connections, so that Logstash doesn't run into any trouble.

```bash
# firing up elasticsearch
sudo service elasticsearch restart || sudo service elasticsearch start
sudo service elasticsearch status

# giving logstash some runway
sleep 10s

# firing up logstash
sudo service logstash restart || sudo service logstash start
sudo service logstash status

# firing up kibana
sudo service kibana restart || sudo service kibana start
sudo service kibana status
```

Great, now we'll need to tweak our `nginx` server.

# Setting up `nginx`

First, I exposed Kibana on my `nginx` server configuration. I chose to password protect my Kibana instance using basic authentication. To do that, you need an `htpasswd` file like so, where `$USERNAME` would contain the basic authentication username and `$PASSWORD` would be the password for that user.

```bash
htpasswd -nb $USERNAME $PASSWORD > $FILENAME
```

That'll create a file such as the following, where we have `user/pass` for authentication and `pass` was encrypted.

```
user:$apr1$MZLc648z$MtfQUP9Kz5216Wafq5j9/1
```

When setting up the `nginx` proxy server, you need to use the `auth_basic` and `auth_basic_user_file` directives, as shown below. Make sure to replace `$FILENAME` with the contents of the `$FILENAME` variable used above.

```
server {
  auth_basic "kibana_restricted";
  auth_basic_user_file <mark>$FILENAME</mark>;
}
```

> ### Notes About X-Pack
>
> _A more robust alternative to proxying via nginx in order to password-protect Kibana would be using [X-Pack and Shield][v5]. X-Pack is a commercial offering that introduces an extra protection layer in front of Elasticsearch. That same authentication layer can then be leveraged to password-protect Kibana._  
> _Other X-Pack features include cluster health monitoring, data change alerts, reporting, and relationship graphs._
>
> I've set it up myself for my blog, and I must say that X-Pack is pretty awesome. Check out [my deployment scripts][deploys] to learn more about setting it up. In this article, I've decided against providing a more detailed description of the installation process and features in X-Pack. If you want to give it a shot, we offer [a month-long trial][trial] where you can take all of these extra awesome features for a ride.

It's important to note that we'll be expecting the `access.log` file to have a specific format that Logstash understands, so that we can consume it and split it into discrete fields. To configure `nginx` with the pattern we'll call metrics, add this to the `http` section of your `nginx.conf` file.

```
log_format metrics '$http_host '
                   '$proxy_protocol_addr [$time_local] '
                   '"$request" $status $body_bytes_sent '
                   '"$http_referer" "$http_user_agent" '
                   '$request_time '
                   '$upstream_response_time';
```

Next, in the `server` section of your `site.conf` for the site you want to be parsing logs for, just tell `nginx` to use the `metrics` logging format we've defined earlier.

```
access_log /var/log/nginx/ponyfoo.com.access.log metrics;
```

After exposing the Kibana web app on my `nginx` server and pointing a DNS `CNAME` entry for [analytics.ponyfoo.com][analyt] _(currently password protected)_ to my load balancer, I was good to go.

# 🚿 From `nginx` to Elasticsearch via Logstash

Logstash relies on inputs, filters applied on those inputs, and outputs. We configure all of this in a `logstash.conf` config file that we can then feed logstash using the `logstash -f $FILE` command.

Getting Logstash to read `nginx` access logs is easy. You specify an event type that's later going to be used when indexing events into Elasticsearch, and the path to the file where your logs are recorded. Logstash takes care of tailing the file and streaming those logs to whatever outputs you indicate.

```
input {
  file {
    type => "nginx_access"
    path => "/var/log/nginx/ponyfoo.com.access.log"
  }
}
```

When it comes to configuring Logstash to pipe its output into Elasticsearch, the configuration *couldn't be any simpler*.

```
output {
  elasticsearch {}
}
```

An important aspect of using Logstash is using filters to determine the fields and field types you'll use when indexing log events into Elasticsearch. The following snippet of code tells Logstash to look for `grok` patterns in `/opt/logstash/patterns`, and to match the `message` field to the `NGINX_ACCESS` grok pattern, which is defined in a patterns file that lives in the provided directory.

```
filter {
  grok {
    patterns_dir => "/opt/logstash/patterns"
    match => { "message" => "%{NGINX_ACCESS}" }
  }
}
```

The contents of the file containing the grok patterns are outlined below, and they should be placed in a file in `/opt/logstash/patterns`, as specified in the `patterns_dir` directive above. Logstash comes with [quite a few built-in patterns][logpat] and the general way of consuming a pattern is `%{PATTERN:name}` where some patterns such as `NUMBER` also take a type argument, such as `%{NUMBER:bytes:int}`, in the piece of code below.

```
NGINX_USERNAME [a-zA-Z\.\@\-\+_%]+
NGINX_USER %{NGINX_USERNAME}
NGINX_ACCESS %{IPORHOST:http_host} %{IPORHOST:client_ip} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:http_version:float})?|%{DATA:raw_request})\" %{NUMBER:status:int} (?:%{NUMBER:bytes:int}|-) %{QS:referrer} %{QS:agent} %{NUMBER:request_time:float} %{NUMBER:upstream_time:float}
NGINX_ACCESS %{IPORHOST:http_host} %{IPORHOST:client_ip} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:http_version:float})?|%{DATA:raw_request})\" %{NUMBER:status:int} (?:%{NUMBER:bytes:int}|-) %{QS:referrer} %{QS:agent} %{NUMBER:request_time:float}
```

Each match in the pattern above means a field will be created for that bit of output, with its corresponding name and type as indicated in the pattern file.

Going back to the `filter` section of our `logstash.conf` file, we'll also want a [`date`][date] filter that parses the `timestamp` field into a proper date. a [`geoip`][geo] filter so that the `client_ip` field gets the geolocation treatment.

```
filter {
  grok {
    patterns_dir => "/opt/logstash/patterns"
    match => { "message" => "%{NGINX_ACCESS}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
  }
  geoip {
    source => "client_ip"
  }
}
```

> You can run `logstash -f $FILE` to test the configuration file and ensure your pattern works. There's also a website where you can [test your `grok` patterns online][grokon].

If you're running Logstash as a service, or intend to, you'll need to place the `logstash.conf` file in `/etc/logstash/conf.d` _-- or change the default configuration directory._

My preference when configuring globally-installed tools such as Logstash is to keep configuration files in a centralized location and then create symbolic links in the places where the tool is looking for those files. In the case of Logstash there's two types of files we'll be using: `grok` patterns and Logstash configuration files. I'll be keeping them in `$HOME/app/logstash` and creating links to the places where Logstash looks for each of these kinds of files.

```bash
sudo ln -sfn $HOME/app/logstash/config/*.conf /etc/logstash/conf.d/
sudo ln -sfn $HOME/app/logstash/patterns/* /opt/logstash/patterns/
```

By the way, I had an issue where `nginx` logs wouldn't be piped into Elasticsearch, and Logstash would fail silently, **but only when run as a service**. It turned out to be a file permissions issue, one that was easily fixed by ensuring that the `logstash` user had execute permissions on every directory from `/` to `/var/log/nginx`.

You can verify that by executing the following command, which interpolates [`getfacl /`][getfacl] all the way through to the `/var/log/nginx` directory.

```bash
getfacl /{,var/{,log/{,nginx}}}
```

You'll get output such as this:

```
getfacl: Removing leading '/' from absolute path names
# file: var/log/nginx
# owner: www-data
# group: adm
user::rwx
group::r-x
other::---
```

You'll note that in this case, user `logstash` of group `logstash` wouldn't have execute rights on `/var/log/nginx`. This can be easily fixed with the following command, but it gave me quite the headache until I stumbled upon this particular oddity.

```
sudo chmod o+x /var/log/nginx
```

> <sub>_Naturally, you'll also need to grant read access to the relevant log files._</sub>

I didn't just want to stream `nginx` logs to Elasticsearch, I also wanted `node` application level logs to be here. Let's do this!

# 🌀 Throwing `node` Logs into the Mix

To add Node logs into the mix, we don't need to change the `output` section of our Logstash file. It has all we needed. We do need to change the `input` and `filter` sections. First, we'll add another `file` to the `input` section. This should point to the file where your Node.js app is streaming its logs into. In my case that file was `/var/log/ponyfoo-production.log`, which is where the service running my `node` app redirected its output. 

```
input {
  file {
    type => "node_app"
    path => "/var/log/ponyfoo-production.log"
  }
}
```

Now, the `node_app` log is a bit different. Sometimes, there are multi-line log entries. That happens when I log a stack trace. That's easily fixed by adding [a `multiline` codec][ml] to my input processor, where each log entry must start with a `NODE_TIME` pattern-matching string, or otherwise is merged into the current log entry. Effectively, this means that lines that don't start with a timestamp will be treated as part of the last event, until a new line that matches a timestamp comes along, creating a new entry.

```
input {
  file {
    type => "node_app"
    path => "/var/log/ponyfoo-production.log"
    <mark>codec => multiline</mark> {
      patterns_dir => "/opt/logstash/patterns"
      pattern => "^%{NODE_TIME} "
      what => "previous"
      negate => true
    }
  }
}
```

My Node.js logs are quite compact, containing a date and time, a `level` marker and the logged message.

```
8 Jun 21:53:27 - info: Database connection to {db:ponyfoo-cluster} established.
08 Jun 21:53:27 - info: Worker 756 executing app@1.0.37
08 Jun 21:53:40 - info: elasticsearch: Adding connection to http://localhost:9200/
08 Jun 21:53:40 - info: cluster listening on port 3000.
08 Jun 21:56:07 - info: Database connection to {db:ponyfoo-cluster} established.
08 Jun 21:56:07 - info: Worker 913 executing app@1.0.37
08 Jun 21:56:07 - info: elasticsearch: Adding connection to http://localhost:9200/
08 Jun 21:56:08 - info: Database connection to {db:ponyfoo-cluster} established.
08 Jun 21:56:08 - info: Worker 914 executing app@1.0.37
08 Jun 21:56:08 - info: elasticsearch: Adding connection to http://localhost:9200/
08 Jun 21:56:08 - info: app listening on ip-10-0-69-59:3000
08 Jun 21:56:09 - info: app listening on ip-10-0-69-59:3000
```

As a result, my Node.js patterns file is much simpler. The `NODE_TIME` was exacted into its own pattern for convenience.

```
NODE_TIME %{MONTHDAY} %{MONTH} %{TIME}
NODE_APP %{NODE_TIME:timestamp} - %{LOGLEVEL:level}: %{GREEDYDATA:description}
```

In order for the `filter` section to play nice with two different input files under different formats, we'll have to update it using conditionals. The following piece of code is the same `filter` section we had earlier, wrapped in an `if` that checks whether we are dealing with an event of type `nginx_access`.

```
filter {
  <mark>if [type] == "nginx_access" {</mark>
    grok {
      patterns_dir => "/opt/logstash/patterns"
      match => { "message" => "%{NGINX_ACCESS}" }
    }
    date {
      match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
    }
    geoip {
      source => "client_ip"
    }
  <mark>}</mark>
}
```

For `node_app` events, we'll do something quite similar, except there's no geolocation going on, and our main `message` pattern is named `NODE_APP` instead of `NGINX_ACCESS`. Also, make sure that the filter matches the new document type, `node_type`.

```
filter {
  if [type] == "node_app" {
    grok {
      patterns_dir => "/opt/logstash/patterns"
      match => { "message" => "%{NODE_APP}" }
    }
    date {
      match => [ "timestamp" , "dd MMM HH:mm:ss" ]
    }
  }
}
```

Purrrfect! 🐯

If you restart `logstash`, -- either using the service or by executing the `logstash -f logstash.conf` command again -- it should start streaming logs into Elasticsearch, from both `nginx` and `node`, as soon as the Logstash service comes online.

How do we interact with those logs in a friendly manner then? While it's okay for search to be mostly accessible through the web interface of the blog, it'd be pretty pointless to go through all of this trouble just to dump the logs into a table again on the web front-end.

That's where *Kibana* comes in, **helping us visualize the data.**

# Getting Started with Kibana

Having already installed the Elastic Stack, and after having set up Logstash to channel our logs into Elasticsearch, we're now ready to fire up Kibana and gain some insights into all our data.

The default port Kibana listens on is `5601`. I use `nginx` as a reverse proxy in front of Kibana. You can use `proxy_pass` to forward requests to the back-end Kibana `node` app. Here's the full `server` entry for `nginx` I use for [ponyfoo.com][pf].

```
server {
  listen 8080 proxy_protocol;

  server_name analytics.ponyfoo.com;
  set_real_ip_from 172.31.0.0/20;
  real_ip_header proxy_protocol;
  access_log /var/log/nginx/analytics.ponyfoo.com.access.log metrics;

  auth_basic "kibana_restricted";
  auth_basic_user_file /path/to/htpasswd/file;

  location / {
    proxy_pass http://127.0.0.1:5601;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_cache one;
    proxy_cache_key nx$request_uri$scheme;
  }
}
```

Alternatively you can just skip `nginx` altogether, that's up to you. I've set it up in front of `nginx` because I wanted basic authentication and I had already set up `nginx` for the main blog application. Now you should be able to visit your Kibana dashboard at either `http://localhost:5601` or your designated port and hostname.

```bash
open http://localhost:5601
```

The first time you load Kibana, you will be greeted with a message about configuring a default index pattern. Kibana suggests we try the `logstash-*` index pattern. Logstash defaults to storing events on indices named according to the day the event is stored on, such as `logstash-2016-06-17`, and so on. The `*` in the pattern acts as a wildcard so that every index starting with `logstash-` will match.

![Set up a default index pattern.][defix]

Once we've set up the index pattern, we might want to visit the *Discover* tab. Here, we'll be able to pull up Elasticsearch records as data rows, as well as a simple time-based chart displaying the amount of events registered over time. You can use the Elasticsearch query DSL to perform any queries and you can also apply filters on top of that query.

![Events over time][discover]

Data tables are nowhere near the most exciting kind of thing you can do in Kibana, though. Let's take a look at a few different charts I've set up, and while we're at it I'll explain the terminology used by the Kibana interface as well as a few statistics terms in layman's language. Articles have been written before that describe in detail [everything you can do with Kibana visualizations][details], but I'll favor *a real-world example-based approach.*

Let's head to the *Visualize* tab for our first visualization.

# Charting Status Codes and Request Volume

Create a pie chart and select `logstash-*` for the index pattern. You'll note that the default pie chart visualization is a whole pie containing every matching document and displaying the total count. Fair enough.

![All requests in a pie][disc]

Add a *Split Slices* bucket with a *Terms* aggregation and choose the `status` field *-- then hit Enter.* That sentence was overcharged with technical terms. Let's break it down. We are splitting the pie into several slices, using the Terms aggregation type to indicate that we want to identify distinct terms in the `status` field, such as `200`, `302`, `404`, and grouping documents by that aggregation. We'll end up with an slice for documents with a status code of `200`, another slice for documents with a `302` status code, and so on.

Generally speaking, a healthy application should be returning mostly `2xx` and `3xx` responses. This chart makes it easy to visualize whether that's the case.

![Splitting slices gives us an slice per status code][split]

You can further specialize the graph by adding a sub-bucket, aggregating again using Terms, this time by the `request.raw` field. That way, you'll not only get a general ratio of status codes, but you'll also gain insight into which requests are the most popular, as well as which ones are the most problematic, all in one chart.

In the screenshot found below, you can see that the `/articles/feed` route is getting even more traffic than the home page. 

![Splitting slices all the way down][subbuck]

Maybe a little bit of caching is in order, since the RSS feeds don't change all that often. In the same light, earlier on these pie charts helped me identify and fix a bunch of requests for icons and images that were yielding `404` errors. That's something that would've been harder to spot by looking at the dense and raw nginx logs, but it's really easy to take action based on visualizations that help us work with the data.

Save the pie chart using the *Save* link on the top bar and give it a name such as "Response Stats".

# Plotting Responses Over Time, By Status Code

Go back to the Visualize tab and create a new line chart. You'll see that we only get a single dot graphed, because there's no `x` axis by default. Let's add an *X-Axis*, using a *Date Histogram* of `@timestamp`. This is a fancy way of saying we want to group our data points into 30 minute long buckets. All events within a 30-minute window are grouped _-- aggregated --_ into a single bucket displaying the Count _(that's the `y` axis)_. A *Date Histogram* makes it possible to render a large dataset into a line chart without having to render several thousands of individual data points. The Date Histogram aggregation allows you to define the time lapse you want to use to group events, or you can use the default *Auto* value of 30 minutes, as with the example below.

![Responses plotted over time][overtime]

While it's useful to render requests over time, particularly when it comes to determining whether our server is functioning properly under load, experiencing a sudden surge of requests, or not responding to a single request, it's even more useful to understand the underlying sub-segments of data. What are the status codes for each of those responses over time? Was the spike that we saw in the earlier chart the product of `500` errors or just a spike in traffic?

Add a *Split Lines* aggregation by Terms using the `status` field. After hitting Enter *(or the Play button)*, we'll note that the chart is now divided into several different lines, each line representing all responses that ended with a given status code. Now it'll be much easier to spot error spikes and discern them from spikes in traffic.

![Responses plotted over time splitted by status code][splitreq]

Save this chart as *"Response Times"*, and let's create something else.

# Charting Geographic Data

It's really easy to chart geographic data into a map using Kibana, provided that you've set up the `geoip` filter in Logstash as indicated earlier, so that the IP addresses found in nginx logs are transformed into geolocation data that can be charted into a map. Given that, it's just a matter of going to the Kibana Visualize tab, creating a Tile Map, and adding a Geo Coordinates bucket using the field computed by Logstash.

![Geolocation map using Kibana and Logstash][map]

Save the map as *"Request Locations"*.

# Stitching It All Together

Now that we've created three different visualizations, where we have a rough overview of status codes and the hottest endpoints, time-based plots of each of the status codes, and a map showing where requests are coming from, it'd be nice to put all of that in a single dashboard.

Visit the *Dashboard* tab in Kibana, pick *Add* from the top-bar navigation and add all of our saved charts. After some rearranging of the charts, you'll get a dashboard like the one below. You can save this one as well, naming it something like *"Overview"*. Whenever you want to get a glimpse into your operations now you can simply visit the Overview dashboard and you'll be able to collect insights from it.

![Our first dashboard][all]

I've also included an "average response time" over time chart in my dashboard, which wasn't discussed earlier. You can plot that on a line chart with a `y` axis of a *Date Histogram* with minute or second resolution, and an `x` axis using an *Average* aggregation of `request_time`.

> Make sure to save it afterwards!

[analyt]: https://analytics.ponyfoo.com "Kibana for Pony Foo"
[geo]: https://www.elastic.co/guide/en/logstash/current/plugins-filters-geoip.html "Logstash Geoip Filter"
[date]: https://www.elastic.co/guide/en/logstash/current/plugins-filters-data.html "Logstash Date Filter"
[logpat]: https://github.com/logstash-plugins/logstash-patterns-core/tree/02fc1c47e094cbbe3e482754cda95088827da327/patterns "Grok patterns shipped with Logstash"
[grokon]: http://grokconstructor.appspot.com/do/match "Test grok patterns online"
[ml]: https://www.elastic.co/guide/en/logstash/current/plugins-codecs-multiline.html "Multi-line Logstash Codec"
[getfacl]: http://www.linuxcommand.org/man_pages/getfacl1.html "The getfacl command explained"
[pf]: https://ponyfoo.com "That's my blog!"
[defix]: https://i.imgur.com/HcJ1IW2.png
[discover]: https://i.imgur.com/gvbDjts.png
[details]: https://www.timroes.de/2015/02/07/kibana-4-tutorial-part-3-visualize/ "Kibana Tutorial – Visualize"
[disc]: https://i.imgur.com/H8e4QVy.png
[split]: https://i.imgur.com/akJ57DG.png
[subbuck]: https://i.imgur.com/v59RwhY.png
[overtime]: https://i.imgur.com/yGwfCmd.png
[splitreq]: https://i.imgur.com/z71sMH5.png
[map]: https://i.imgur.com/Um1NzC4.png
[all]: https://i.imgur.com/coOul7v.png
[setup]: /articles/setting-up-elasticsearch-for-a-blog "Setting Up Elasticsearch for a Blog on Pony Foo"
[v5]: https://www.elastic.co/v5 "Say Heya to the Elastic Stack"
[deploys]: https://github.com/ponyfoo/ponyfoo/blob/2b4bd84cad9aa20e2a326d715259d1c9b499b68e/deploy/templates/primal#L13-L41 "Deploying X-Pack for Pony Foo"
[trial]: https://www.elastic.co/downloads/x-pack "Download X-Pack for the Elastic Stack"
