* Lint my source code
* Run unit tests
* Clean my distribution directory
* Build the distribution files, minified and otherwise
* Get the file size of both the regular and minified versions
* Bump the package version for `npm` and `bower`
* Push a new tag to `git`, to update the Bower version
* Publish the updated version to `npm`

In this article I aim to explain what I did, how I did it, and the reasons why I made some of the choices that I did. The only real problem I had had to do with synchronicity. I felt it would be interesting walking you through the process. It may help you get started with Gulp!

![rocket.png][1]

[1]: https://i.imgur.com/ApIcjlI.png "The Gulp Rocket!"
