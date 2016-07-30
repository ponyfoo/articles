I just wrote [a little thing][1] that can easily read the standard output of a program, and reproduce it at a later time. This sounds pretty simple business, but having it packaged up in a little script, with a simple API, is pretty important.

Oh, it also respects delays in the original execution, rather than dumping everything at once.

To install it, use `npm`.

```shell
npm i -g rehearsal
```

Now you can prepare an scenario, suppose you want to give a talk on the awesomeness of [grunt-ec2][2], which makes lots of requests over the network, and does stuff. It would be great, being able to demonstrate its abilities without the need for an internet connection.

You can simply run the command as usual, but pipe its output to `rehearsal`, saving the `scenario` to a file on disk.

```shell
grunt ec2_list --color | rehearsal > scenario
```

Note that the command is really executed here, we're just redirecting its output, so make sure that that doesn't blow up your repository, or production servers, or anything.

I got back a series of encoded streams, persisted in the `scenario` file. I had to use the `--color` argument because `chalk` won't produce any color coding, otherwise.


```
{"time":145,"data":":base64:MjR0aCAxNzozMjoyNSAtIExvYWRlZCBjb25maWd1cmF0aW9uIGZvciBncnVudCBlbnZpcm9ubWVudAo="}
{"time":69,"data":":base64:MjR0aCAxNzozMjoyNSAtIGNvbnNvbGUgdHJhbnNwb3J0IGVuYWJsZWQK"}
{"time":0,"data":":base64:MjR0aCAxNzozMjoyNSAtIHB1c2hvdmVyIHRyYW5zcG9ydCBvZmYKMjR0aCAxNzozMjoyNSAtIHBhcGVydHJhaWwgdHJhbnNwb3J0IG9mZgo="}
{"time":111,"data":":base64:MjR0aCAxNzozMjoyNSAtIExvYWRpbmcgZXh0ZXJuYWwgdGFza3MuLi4="}
{"time":1028,"data":":base64:ZG9uZSBpbiAxLjI3NjcwNDQ2cwo="}
{"time":4,"data":":base64:Cg=="}
{"time":0,"data":":base64:G1s0bVJ1bm5pbmcgImVjMl9saXN0IiB0YXNrG1syNG0K"}
{"time":3,"data":":base64:R2V0dGluZyBFQzIgaW5zdGFuY2VzIGZpbHRlcmVkIGJ5IBtbMzZtcnVubmluZxtbMzltIHN0YXRlLi4uCg=="}
{"time":0,"data":":base64:G1s0bRtbMzNtW2NtZF0bWzM5bRtbMjRtIBtbMzVtYXdzIGVjMiBkZXNjcmliZS1pbnN0YW5jZXMgLS1maWx0ZXJzIE5hbWU9aW5zdGFuY2Utc3RhdGUtbmFtZSxWYWx1ZXM9cnVubmluZxtbMzltCg=="}
{"time":1249,"data":":base64:G1szMm0+PiAbWzM5bUZvdW5kIDEgRUMyIEluc3RhbmNlKHMpCg=="}
{"time":1,"data":":base64:G1szNW1pLTRlZTdlMTI4G1szOW0gG1szNW1hbWktYzMwMzYwYWEbWzM5bSAoG1szMm1ydW5uaW5nG1szOW0pIFsbWzM2bXByb2R1Y3Rpb24bWzM5bV0gb24gG1s0bTEwNy4yMC4xOTguMjM5G1syNG0K"}
{"time":1,"data":":base64:Cg=="}
{"time":0,"data":":base64:G1szMm1Eb25lLCB3aXRob3V0IGVycm9ycy4bWzM5bQo="}
{"time":1,"data":":base64:ChtbNG1FbGFwc2VkIHRpbWUbWzI0bQo="}
{"time":1,"data":":base64:ZWMyX2xpc3QgIDFzChtbMW1Ub3RhbCAgICAgMXMbWzIybQo="}
```

The scenario can be reproduced at any time using the `reheasal <scenario>` command, as shown in the screenshot below.

[![rehearsal.png][3]][1]

If you want to take it up a notch, maybe you'd like to create an alias, and then people wouldn't really be able to tell that you were faking it.

```shell
alias realthing="rehearsal scenario"
```

Using spaces in the alias name is a bit tricky, [but it could be done][4]. By not using an alias, but a function instead.

  [1]: https://github.com/bevacqua/rehearsal "bevacqua/rehearsal on GitHub"
  [2]: https://github.com/bevacqua/grunt-ec2 "bevacqua/grunt-ec2 on GitHub"
  [3]: https://i.imgur.com/boNkRem.png 
  [4]: http://superuser.com/q/105375 "Bash: Spaces in alias name"
