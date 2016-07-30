For context, here's a recap of the events leading up to the short-lived `npm install` issues earlier this week.

- Azer published [an open-source library][kikoss] as the `kik` package on `npm`
- [Kik][kikcom] wanted to use the `kik` package, and they approached Azer about it
- Azer doesn't want to give up the `kik` package name
- Kik approaches `npm` through their conflict resolution policy [about a solution][response]
- `npm` [transfers ownership][npmresp] of `kik` to Kik
- Azer retaliates by [unpublishing all of his packages][retaliation] from `npm`
- One of those packages, [`left-pad`][leftpad], was a popular dependency, underlying many other packages in the ecosystem
- `npm install` started failing across the board for every package that has a dependency on `left-pad@0.0.3`
- A third party published `left-pad@1.0.0` with a copy of `left-pad@0.0.3`, but it didn't help a lot
- `npm` allowed that same third party to re-publish `left-pad@0.0.3` as an exact copy of the unpublished package
- Service was restored

From there, dozens of articles and tweets about the event were dispatched. Some questioned Kik's claim to ownership of the `kik` package name. Some questioned Azer's retaliation causing more damage to the community than to Kik. Some questioned `npm` for allowing a user to un-break the ecosystem by re-publishing an unpublished and fully open-source package. Some questioned `npm` for allowing users to `npm unpublish` in the first place. Some questioned `npm` for defending their interests over Azer's and giving up ownership of `kik`.

The root cause of all of the above is, of course, **trust**. And it runs deep. In this article I want to go over the implications and hopefully trigger a constructive discussion around the topic.

[kikoss]: https://github.com/starters/kik "starters/kik on GitHub"
[kikcom]: https://www.kik.com/
[retaliation]: https://medium.com/@azerbike/i-ve-just-liberated-my-modules-9045c06be67c "I’ve Just Liberated My Modules"
[response]: https://medium.com/@mproberts/a-discussion-about-the-breaking-of-the-internet-3d4d2a83aa4d "A discussion about the breaking of the Internet"
[npmresp]: http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm "kik, left-pad, and npm"
[leftpad]: https://github.com/azer/left-pad "azer/left-pad on GitHub"
