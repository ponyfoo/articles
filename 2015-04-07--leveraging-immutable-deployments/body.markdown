# Process Recap

The approach I propose involves two steps. A `setup` step and a `deploy` step.

You are supposed to run the `setup` script once, when first putting together a new environment. No images are involved for the `setup` process, and it's very fast. Everything is done at the infrastructure level. The `setup` script involves the following steps.

- Create an [ELB][3]
- Configure the [ELB][3] with [connection draining][10]
- Set up health checks on the [ELB][3]
- Create a [Route 53][5] `ALIAS` record set pointing at the [ELB][3]

The `deployment` script is a bit longer. This is the one that you will be _(presumably)_ running a few times a day. Most of the time is spent baking images and waiting for [EC2][2] instances to become healthy in the eyes of [ELB][3].

- Bake `primal` image with [Packer][4], if needed _(on occasion)_
- Build static assets and optimize for `NODE_ENV`
- Bake `carnivore` image with [Packer][4]
- Create an [ASG][1] with the desired amount of `carnivore`-ready [EC2][2] instances
- Wait until all instances are marked as `InService` and `Healthy` on [EC2][2]
- Wait until all instances are marked as `InService` on [ELB][3]
- Detach all outdated [EC2][2] instances from [ELB][3]
- Scale any outdated [ASG][1] down to _0 instances_
- Wait until all outdated [EC2][2] instances are terminated
- Delete outdated [ASG][1]
- Deregister `carnivore` AMI from AWS
- Delete `carnivore` AMI snapshot from AWS

Besides [Packer][4], which you should've installed if you already went over [the previous article][7], you'll need to have a few more tools laying around. 

# Requisites

The first of them is the [AWS CLI][12] _(needs Python if you don't have it already in your system)_, obviously.

```bash
pip install awscli
aws configure
```

We use two different tools to manipulate JSON data. One is [jq][13], which is great for most use cases where we just need to select some data deep in a JSON structure. If you are on OSX, you can use `brew`.

```bash
brew install jq
```

The other JSON tool is [underscore-cli][14], which is better suited for complex filtering, mapping, and selects.

```bash
npm install underscore-cli --save-dev
```

In the next couple of sections we'll go over each of the two scripts and how they work.

# Putting together the `setup` script

Okay, so it's Bash again!

```bash
#!/bin/bash
```

Not sure how this is _not_ the default behavior, but this command makes sure that your script doesn't behave as if [On Error Resume Next][6] was on _(for those unfamiliar with Basic, `set -e` will make sure that errors are treated as such)_.

```bash
set -e
```

The following bit of code defaults `NODE_ENV` to `staging`, unless it's user-provided.

```bash
: "${NODE_ENV:="staging"}"
```

First off, let's pull up the [Amazon Web Services Console][8] and find the _"Hosted Zone ID"_ for the domain you want to use with this application. Then, paste it in the next line.

```bash
HOSTED_ZONE="{{YOUR_HOSTED_ZONE_ID}}"
```

Next we declare the top level domain that we're going to be using. This should've already been configured on the Route 53 hosted zone. In my case, that's `ponyfoo.com`, obviously.

```bash
TLD="ponyfoo.com"
```

As I [mentioned in the previous article][7], _resources are named conventionally_ so that names don't clash with each other. Let's set up a bunch of names we are going to use.

```bash
NAME="ponyfoo-$NODE_ENV"
ELB_NAME="elb-$NAME"
ASG_NAME="asg-$NAME"
LC_NAME="lc-$NAME-initial"
KEYFILE="deploy/keys/$NODE_ENV"
```

What [availability zones][9] should be used by the load balancer? This is configurable, but stick to something that works! I originally wanted to pull these from the ELB, but _it seems not every instance type is available in every availability zone_, so your mileage may vary.

```bash
AVAILABILITY_ZONES="us-east-1a us-east-1b"
```

My convention is to use the naked domain when `$NODE_ENV` is `production` _(`example.com` vs `staging.example.com`)_, but you can easily change that.

```bash
HOST_NAME=$NODE_ENV"."

if [ "$HOST_NAME" == "production." ]
then
  HOST_NAME=""
fi
```

One last bit of set up before we start to interact with Amazon. Most commands will be logged so that you can diagnose what happened in case something goes wrong. Here's where to look for them.

```bash
rm -rf deploy/log
mkdir deploy/log
```

Onto the exciting business. We start by creating the load balancer on the designated availability zones, and telling it to listen on port `80`. We name the ELB according to our conventions so we can easily dig it up later on _(or inspect it in the AWS management console)_.

```bash
echo "setup: creating $ELB_NAME load balancer ($AVAILABILITY_ZONES)..."
aws elb create-load-balancer \
  --load-balancer-name "$ELB_NAME" \
  --listeners Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80 \
  --availability-zones $AVAILABILITY_ZONES > deploy/log/elb-create.log
```

Turn on [connection draining][10] on the load balancer we just created. Instances are given 300 seconds _(5 minutes)_ after being detached from ELB to finish off outstanding requests. This is a crucial aspect of managing deployments gracefully. We'll shut off old instances from ELB but they'll still be allowed to complete those outstanding requests on their own terms.

```bash
echo "setup: enabling connection draining on elb..."
aws elb modify-load-balancer-attributes \
  --load-balancer-name "$ELB_NAME" \
  --load-balancer-attributes "{\"ConnectionDraining\":{\"Enabled\":true,\"Timeout\":300}}" > deploy/log/elb-draining.log
```

Lastly, we'll set up health checks on ELB. You should set up a `GET /api/status/health` endpoint in your application. All it needs to do is return a `200 OK` status code in under 4 seconds _(`Timeout`)_. Instance health is checked twice every minute _(`Interval`)_. If an instance fails the health check twice _(`UnhealthyThreshold`)_, it'll become unhealthy. When an instance is marked as unhealthy the autoscaling group will probably take it down, but nevertheless we've configured a _`HealthyThreshold`_ of 2, which would bring it back to _Healthy_ after two consecutive successful health checks.

```bash
echo "setup: configuring health checks on elb..."
aws elb configure-health-check \
  --load-balancer-name "$ELB_NAME" \
  --health-check Target=HTTP:80/api/status/health,Interval=30,UnhealthyThreshold=2,HealthyThreshold=2,Timeout=4 > deploy/log/elb-health.log
```

The next block of code queries Amazon about the ELB, getting the metadata we need to set up the `ALIAS` record on our Route 53 hosted zone. This is necessary because the command to create the ELB doesn't yield information such as the hosted zone ID or the hosted zone name. Afterwards, we use that metadata to upsert an `ALIAS` record on Route 53. We indicate that we want Route 53 to take the ELB health check into consideration.

```bash
echo "setup: describing load balancer to create route53 alias recordset..."
aws elb describe-load-balancers \
  --load-balancer-name "$ELB_NAME" > deploy/log/elb-describe-lb.log

ELB_ZONE_ID=$(jq -r '.LoadBalancerDescriptions[0].CanonicalHostedZoneNameID' < deploy/log/elb-describe-lb.log)
ELB_ZONE_NAME=$(jq -r '.LoadBalancerDescriptions[0].CanonicalHostedZoneName' < deploy/log/elb-describe-lb.log)

echo "setup: creating route53 alias recordset on $HOST_NAME$TLD..."
echo "{
  \"Changes\": [{
    \"Action\": \"UPSERT\",
    \"ResourceRecordSet\": {
      \"Type\": \"A\",
      \"Name\": \"$HOST_NAME$TLD.\",
      \"AliasTarget\": {
        \"HostedZoneId\": \"$ELB_ZONE_ID\",
        \"DNSName\": \"$ELB_ZONE_NAME\",
        \"EvaluateTargetHealth\": true
      }
    }
  }]
}" > deploy/log/route53-record-set-changes.log

aws route53 change-resource-record-sets \
  --hosted-zone-id "$HOSTED_ZONE" \
  --change-batch "file://deploy/log/route53-record-set-changes.log" > deploy/log/route53-change-recordset.log
```

Finally, we check to see if an `ssh` key file was already generated and uploaded to Amazon, and if not, we upload one. The key file will grant `ssh` access to every EC2 instance assigned to the current `$NODE_ENV` environment. You are welcome to omit this step if you don't plan on needing to `ssh` into instances to debug an issue. I don't usually do it, but I like knowing I can _(so that I can pinpoint what is going on, directly from a production instance)_.

```bash
if [ -f "$KEYFILE" ]
then
  echo "setup: ssh key file already exists on aws."
else
  echo "setup: ssh key file doesn't exist yet. creating..."
  mkdir -p deploy/keys
  ssh-keygen -t rsa -b 4096 -N "" -f "$KEYFILE"
  aws ec2 import-key-pair \
    --key-name "$NAME" \
    --public-key-material "file://$KEYFILE.pub" > deploy/log/ec2-upload-keypair.log
  echo "setup: ssh key file uploaded to aws."
fi

echo "setup: done."
```

> **"If you need to `ssh` into an instance that means your automation has failed"**
>
> This is something I've read many times, and while it's a great idea not to depend on `ssh` for any of our process, it's still valuable being _able_ to `ssh` in case something goes wrong. You wouldn't want to be left stranded on principle, would you?

That's all there is to the `setup` script. We now have an ELB, and it's linked to our hosted zone on Route 53. At this point if we were to navigate to `ponyfoo.com` we'd see a blank page, served by ELB when no healthy instances are available.

# A immutable `deploy` script

Again, this is Bash. We don't tolerate mistakes. And we default `NODE_ENV` to `staging`.

```bash
#!/bin/bash

set -e

: "${NODE_ENV:="staging"}"
```

The 3 variables below allow us to make the deployment process faster. Setting `PRIMAL_ID` means we'll skip the creation of a `primal` image. Most of the time we don't want to create a new `primal` image, but sometimes we might want to _(especially during debugging)_. Setting `IMAGE_ID` means we'll also skip the creation of a `carnivore` image. Most of the time we want to **create new `carnivore` images for each deployment**, but it's useful to skip that step during debugging as _it'll save considerable time_.

Finally, `CLEANUP="no"` means that the `IMAGE_ID` AMI shouldn't be removed after a deployment. By default, those AMI are deleted only if they were produced by the deployment script, but you can set `CLEANUP="no"` to make sure that they're never deleted.

```bash
# PRIMAL_ID="ami-xxxxxxxx"
# IMAGE_ID="ami-xxxxxxxx"
# CLEANUP="no"
```

Some more configuration follows. We choose the type of instances we want, how many we'd like, the minimum and maximum capacity to be defined for the [ASG launch configuration][11], and whatnot.

```bash
INSTANCE_TYPE="t1.micro"
INSTANCE_USER="admin"
SECURITY_GROUP="default"
MIN_CAPACITY="1"
MAX_CAPACITY="2"
DESIRED_CAPACITY="1"
```

A couple of helper methods to get a timestamp like `20150407181937` and to sleep for a short while and print a single dot.

```bash
timestamp () {
  date +"%Y%m%d%H%M%S"
}

zzz () {
  sleep 2
  printf "."
}
```

This may not be your convention, but you should do whatever static asset building you need inside this function. In my case I have `npm run` scripts like `build-staging` and `build-production`.

```bash
build_app () {
  echo "deploy: building app for $NODE_ENV environment"
  npm run build-$NODE_ENV
}
```

The `copy_over` method copies everything that's needed to run the application in a hosted environment, so this will probably be different from case to case. This should probably include the static assets you built in the `build_app` method. Going back to [the previous article][7], you might remember that `carnivore` images took an entire directory from `tmp/appserver`. In `copy_over`, we prepare that directory with everything the image will need.

```bash
copy_over () {
  echo "deploy: copying files over to tmp/appserver"
  rm -rf tmp/appserver
  mkdir -p tmp/appserver/deploy/env
  cp -r {.bin,client,controllers,lib,models,resources,services,views,.env.defaults.json,.taunusrc,*.js,package.json} tmp/appserver
  cp "deploy/env/$NODE_ENV.json" tmp/appserver/deploy/env
}
```

The following couple of functions are in charge of conditionally building the base `primal` image with Packer. As you can see we won't be passing `packer` any configuration besides what [we set up last time around][7]. The only condition that's checked is whether a `PRIMAL_ID` variable is set. The `tee` command writes to a file while still printing to standard output.

```bash
build_primal_image () {
  echo "deploy: building primal image with packer..."

  cp package.json deploy/mailtube
  packer build \
    deploy/templates/primal.json | tee deploy/log/packer-primal.log

  PRIMAL_ID=$(tail -1 < deploy/log/packer-primal.log | cut -d ' ' -f 2)

  echo "deploy: built image $PRIMAL_ID"
}

maybe_build_primal_image () {
  if [ -z ${PRIMAL_ID+x} ]
  then
    build_primal_image
  else
    echo "deploy: skipping primal image build, using $PRIMAL_ID"
  fi
}
```

Building `carnivore` needs us to prepare the application for the relevant environment, first. This time we'll forward a couple of variables to `packer`. The `NODE_ENV` variable will be used when booting up our application so that the image knows what environment it should expect to be running on. The `SOURCE_ID` variable will be used in the template to base our deployment image off of the `SOURCE_ID` AMI.

```bash
build_deployment_image () {
  echo "deploy: building carnivore image with packer..."

  packer build \
    -var NODE_ENV=$NODE_ENV \
    -var SOURCE_ID=$PRIMAL_ID \
    deploy/templates/carnivore.json | tee deploy/log/packer-carnivore.log

  IMAGE_ID=$(tail -1 < deploy/log/packer-carnivore.log | cut -d ' ' -f 2)

  echo "deploy: built image $IMAGE_ID"
}

maybe_build_deployment_image () {
  if [ -z ${IMAGE_ID+x} ]
  then
    build_app
    copy_over
    build_deployment_image
  else
    CLEANUP="no"
    echo "deploy: skipping deployment image build, using $IMAGE_ID"
  fi
}
```

The `lookup_existing_asg` method records pre-existing autoscaling groups and launch configurations so that they can be turned off and deleted later on. The conditionals near the bottom are used to avoid creating a file with a single new line, as that would result in problems when reading that file line by line later on in the process.

```bash
lookup_existing_asg () {
  echo "deploy: pulling down list of existing autoscaling groups..."
  aws autoscaling describe-auto-scaling-groups > deploy/log/asg-list.log

  echo "deploy: pulling down list of existing launch configurations..."
  aws autoscaling describe-launch-configurations > deploy/log/asg-lc.log

  EXISTING_GROUP_NAMES=$(underscore process --outfmt text "data.AutoScalingGroups.filter(function (asg) {
    return asg.AutoScalingGroupName.indexOf(\"asg-$NAME\") === 0
  }).map(function (asg) {
    return asg.AutoScalingGroupName
  })" < deploy/log/asg-list.log)

  EXISTING_LAUNCH_CONFIGURATIONS=$(underscore process --outfmt text "data.LaunchConfigurations.filter(function (lc) {
    return lc.LaunchConfigurationName.indexOf(\"lc-$NAME\") === 0
  }).map(function (lc) {
    return lc.LaunchConfigurationName
  })" < deploy/log/asg-lc.log)

  if [ "$EXISTING_GROUP_NAMES" != "" ]
  then
    echo "$EXISTING_GROUP_NAMES" > deploy/log/asg-existing-group-names.log
  else
    touch deploy/log/asg-existing-group-names.log
  fi

  if [ "$EXISTING_LAUNCH_CONFIGURATIONS" != "" ]
  then
    echo "$EXISTING_LAUNCH_CONFIGURATIONS" > deploy/log/asg-existing-lc.log
  else
    touch deploy/log/asg-existing-lc.log
  fi
}
```

In `launch_updated_asg` we'll create [a new launch configuration][11], and use that to create a new [ASG][1]. The [ASG][1] is configured to ask the ELB for health reports, to tag instances according to the current environment, to use the availability zones registered with the ELB, and to launch as many instances as we've configured earlier. The ASG is in charge of creating the instances, so this is the only place where you need to change _how_ they will be created. Generally speaking you won't ever need to create instances yourself unless you are in some kind of obscure hurry. If you want more instances you should **just increase the desired and maximum capacity** in the ASG.

```bash
launch_updated_asg () {
  echo "deploy: describing elb to get availability zones..."
  aws elb describe-load-balancers \
    --load-balancer-name "$ELB_NAME" > deploy/log/elb-describe-lb.log

  AVAILABILITY_ZONES=$(jq -r ".LoadBalancerDescriptions[0].AvailabilityZones[]?" < deploy/log/elb-describe-lb.log)

  echo "deploy: creating $LC_NAME using the latest image..."
  aws autoscaling create-launch-configuration \
    --launch-configuration-name "$LC_NAME" \
    --image-id "$IMAGE_ID" \
    --instance-type "$INSTANCE_TYPE" \
    --key-name "$NAME" \
    --security-groups "$SECURITY_GROUP" > deploy/log/asg-lc-creation.log

  echo "deploy: creating $ASG_NAME autoscaling group..."
  aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name "$ASG_NAME" \
    --launch-configuration-name "$LC_NAME" \
    --availability-zones $AVAILABILITY_ZONES \
    --health-check-type "ELB" \
    --health-check-grace-period 300 \
    --load-balancer-names "$ELB_NAME" \
    --min-size "$MIN_CAPACITY" \
    --max-size "$MAX_CAPACITY" \
    --desired-capacity "$DESIRED_CAPACITY" \
    --tags ResourceId=$ASG_NAME,Key=Name,Value=$NAME ResourceId=$ASG_NAME,Key=Role,Value=web > deploy/log/asg-create-group.log
}
```

Once the new [ASG][1] has been created, we need to wait for EC2 and ELB to report that all of the instances are healthy and reachable through the load balancer. We mix polling and short naps to avoid spamming the API, while not taking too long to react to health status changes either.

```bash
wait_on_health () {
  EC2_HEALTH="0"
  while [ "$EC2_HEALTH" != "$DESIRED_CAPACITY" ]
  do
    printf "deploy: ensuring new instance(s) are healthy at ec2"
    zzz;zzz
    aws autoscaling describe-auto-scaling-groups \
      --auto-scaling-group-names "$ASG_NAME" > deploy/log/asg-description.log

    EC2_HEALTH=$(underscore process --outfmt text "data.AutoScalingGroups[0].Instances.filter(function (i) {
      return i.LifecycleState === 'InService' && i.HealthStatus === 'Healthy'
    }).length" < deploy/log/asg-description.log)

    echo " ($EC2_HEALTH/$DESIRED_CAPACITY are healthy)"
  done

  ELB_INSTANCES=$(jq -r '.AutoScalingGroups[0].Instances[]?.InstanceId' < deploy/log/asg-description.log)
  ELB_HEALTH="0"
  while [ "$ELB_HEALTH" != "$DESIRED_CAPACITY" ]
  do
    printf "deploy: ensuring new instance(s) are healthy at elb"
    zzz;zzz
    aws elb describe-instance-health \
      --load-balancer-name "$ELB_NAME" \
      --instances $ELB_INSTANCES  > deploy/log/elb-health-description.log

    ELB_HEALTH=$(underscore process --outfmt text "data.InstanceStates.filter(function (s) {
      return s.State === 'InService'
    }).length" < deploy/log/elb-health-description.log)

    echo " ($ELB_HEALTH/$DESIRED_CAPACITY are healthy)"
  done
}
```

Now that our new instances are up and running, we can safely remove outdated instances from the ELB. We iterate over all the ASG that predated our deployment, and deregister their instances. Then, we also _downscale those ASG to 0_, effectively asking them politely to gracefully shut down all of their instances. After all of them have been terminated, we shut down the ASG itself. Then, we delete all outdated launch configurations.

```bash
cleanup_outdated_autoscaling_groups () {
  while read EXISTING_GROUP_NAME
  do
    ASG_INSTANCES=$(underscore process --outfmt text "data.AutoScalingGroups.filter(function (asg,i) {
      return asg.AutoScalingGroupName === \"$EXISTING_GROUP_NAME\"
    }).shift().Instances.map(function (i) {
      return i.InstanceId
    })" < deploy/log/asg-list.log)

    echo "deploy: removing instances in outdated $EXISTING_GROUP_NAME from $ELB_NAME..."
    aws elb deregister-instances-from-load-balancer \
      --load-balancer-name $ELB_NAME \
      --instances $ASG_INSTANCES > deploy/log/elb-deregister.log

    echo "deploy: downscaling outdated $EXISTING_GROUP_NAME..."
    aws autoscaling update-auto-scaling-group \
      --auto-scaling-group-name $EXISTING_GROUP_NAME \
      --max-size 0 \
      --min-size 0 > deploy/log/asg-downscale.log

    OPERATIONAL="1"
    while [ "$OPERATIONAL" != "0" ]
    do
      printf "deploy: ensuring outdated instance(s) are terminated"
      zzz;zzz
      aws autoscaling describe-auto-scaling-groups \
        --auto-scaling-group-names "$EXISTING_GROUP_NAME" > deploy/log/asg-existing-description.log

      OPERATIONAL=$(underscore process --outfmt text "data.AutoScalingGroups.filter(function (asg) {
        return asg.AutoScalingGroupName === \"$EXISTING_GROUP_NAME\"
      }).shift().Instances.length" < deploy/log/asg-existing-description.log)

      echo " ($OPERATIONAL are operational)"
   done

    echo "deploy: deleting outdated $EXISTING_GROUP_NAME..."
    aws autoscaling delete-auto-scaling-group \
      --auto-scaling-group-name $EXISTING_GROUP_NAME || echo "deploy: delete failed. maybe it's already deleted."
  done < deploy/log/asg-existing-group-names.log

  while read EXISTING_LC_NAME
  do
    echo "deploy: removing outdated launch configuration $EXISTING_LC_NAME..."
    aws autoscaling delete-launch-configuration \
      --launch-configuration-name "$EXISTING_LC_NAME" >> deploy/log/asg-lc-deletion.log || echo "deploy: delete failed. maybe it's already deleted."
  done < deploy/log/asg-existing-lc.log
}
```

The `cleanup_deployment_image` method deregisters the `$IMAGE_ID` AMI and deletes the associated snapshot, making sure that our deployment leaves no stray artifacts on its way.

```bash
cleanup_deployment_image () {
  if [ "$CLEANUP" != "no" ]
  then
    SNAPSHOT_ID=$(aws ec2 describe-images \
      --image-ids $IMAGE_ID \
      | jq -r .Images[0].BlockDeviceMappings[0].Ebs.SnapshotId)

    echo "deploy: deregistering deployment image $IMAGE_ID"
    aws ec2 deregister-image --image-id $IMAGE_ID

    echo "deploy: deleting snapshot $SNAPSHOT_ID"
    aws ec2 delete-snapshot --snapshot-id $SNAPSHOT_ID
  fi
}
```

Lastly, we run all the methods we've declared. You probably won't have to touch this beyond changing the `$NAME` prefix from `ponyfoo` to `something-else`.

```bash
STAMP="$(timestamp)"
NAME="ponyfoo-$NODE_ENV"
ELB_NAME="elb-$NAME"
KEYFILE="deploy/keys/$NODE_ENV"
ASG_NAME="asg-$NAME-$STAMP"
LC_NAME="lc-$NAME-$STAMP"

rm -rf deploy/log
mkdir deploy/log
maybe_build_primal_image
maybe_build_deployment_image
lookup_existing_asg
launch_updated_asg
wait_on_health
cleanup_outdated_autoscaling_groups
cleanup_deployment_image
```

**Finally!** That's _it_.

# Conclusion

I think one of the best aspects of this deployment process is how easy it is to set up. I designed it for another project but I managed to cram it into `ponyfoo.com` in under 10 minutes. The coolest part is that it really isn't tied to one technology in particular, and besides the environment being defined as `NODE_ENV` there isn't a lot of `node`-specific code in the deployment scripts, as most of that is contained in the Packer image provisioning scripts.

This separation of concerns, where you build your static assets in your local environment, you bake an AMI with everything your hosted environments need, and you perform deployments in an application-agnostic way, makes deployments with this setup a breeze. _At least for me!_

**Happy Bashing!**

<sub>_[You'll find a full copy of the scripts outlined in these articles on GitHub][15]_.</sub>

[1]: http://aws.amazon.com/autoscaling/ "Amazon Auto Scaling Groups"
[2]: http://aws.amazon.com/ec2/ "Amazon Elastic Cloud Compute"
[3]: http://aws.amazon.com/elasticloadbalancing/ "Amazon Elastic Load Balancers"
[4]: https://packer.io/
[5]: http://aws.amazon.com/route53/ "Amazon Web Services Route 53"
[6]: http://www.vbforums.com/showthread.php?448401-Classic-VB-What-is-wrong-with-using-quot-On-Error-Resume-Next-quot "What is wrong with using 'On Error Resume Next'?"
[7]: /articles/immutable-deployments-packer "Immutable Deployments and Packer"
[8]: https://console.aws.amazon.com/console/home "AWS Management Console"
[9]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html "Regions and Availability Zones"
[10]: https://aws.amazon.com/blogs/aws/elb-connection-draining-remove-instances-from-service-with-care/ "ELB Connection Draining – Remove Instances From Service With Care"
[11]: http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WorkingWithLaunchConfig.html "Creating Launch Configurations"
[12]: http://aws.amazon.com/cli/ "Amazon Web Services Command-Line Interface"
[13]: http://stedolan.github.io/jq/ "jq is a lightweight and flexible command-line JSON processor"
[14]: https://github.com/ddopson/underscore-cli "underscore-cli on GitHub"
[15]:  https://github.com/bevacqua/baal "bevacqua/baal on GitHub"
