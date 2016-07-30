[Integration Testing][1] is a must if we're to build a reasonably resilient web application. It helps us figure out evident errors in the "happy path" of execution. However, when it comes to automated tools that enable us to do integration testing **on real browsers**, _the options available to us are pretty underwhelming_.

WebDriver was [introduced by Google in 2009][2].

> WebDriver is a clean, fast framework for automated testing of webapps.

To my knowledge, _documentation for WebDriver is scarce_, at best. Attempts to interact with it are going to be painful for you, particularly if you're attempting to use one of the even less documented implementations, such as the [wd][3] package, built to run these tests using Node. The worst of it is that there **doesn't seem to be a better alternative** if you want to test with real browsers, like Chrome _(which you should)_. This is paired with the popularity of partially-implemented libraries which _"sort of do what you want"_, but aren't able to do really basic stuff like handling file uploads.

I wish I had the time to invest effort in a Kickstarter project to improve the current state of affairs. I'd love to see a better integration testing solution which can be implemented in any language through an API like Selenium does, and supports any browser, **which just works**, and whose consuming libraries were a lot better (API wise), and much better documented as well. Good documentation is _vastly underestimated_ nowadays.

[![selenium.png][5]][4]

[1]: http://en.wikipedia.org/wiki/Integration_testing
[2]: http://google-opensource.blogspot.com.ar/2009/05/introducing-webdriver.html
[3]: https://github.com/admc/wd
[4]: http://docs.seleniumhq.org/projects/webdriver/
[5]: https://i.imgur.com/T5uFEMC.png
