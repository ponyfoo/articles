I've blogged very little about [Taunus][1] since I first released it, [roughly a year ago][2]. Back then, _it only powered [ponyfoo.com][3]_, but now there's a few cases in the wild where it's being used, and I even got to do some consulting in a project where they're using Taunus! In the year since its release, it has had [a whooping 174 releases][4], but not a whole lot has changed, and its API has remained stable for the most part. Its feature-set grew quite a bit, although it remains fairly light-weight at `18.8kB` after gzip and minification. Today, it's able to figure out how to [submit forms via AJAX automatically][5], as long as they already work as plain HTML, and it has support for _WebSockets_ [via a plugin][6].

[1]: https://github.com/taunus/taunus
[2]: /articles/taunus-micro-isomorphic-mvc-framework
[3]: /
[4]: https://www.npmjs.com/package/taunus
[5]: https://github.com/taunus/formium
[6]: https://github.com/taunus/skyrocket
