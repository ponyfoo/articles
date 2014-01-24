# Module Packaging in JavaScript Today #

It's time for me to revisit my opinions on [npm](https://npmjs.org "Node Packaged Modules"), node's module packaging system. At the same time, I want to explore a popular asset management option that are becoming more and more popular, in [Browserify](http://browserify.org/ "npm modules in the browser - Browserify").

In the past, I wrote a [couple of posts](/search/tagged/npm "Articles tagged npm") regarding `npm`. In particular, an article explained [how to publish](/2013/01/23/publishing-nodejs-packages-with-npm "Publishing NodeJS Packages with npm"), and I gave my [opinion](/2013/01/18/asset-management-in-node "Asset Management in Node") about the current state of asset management in node, where I introduced [assetify](https://github.com/bevacqua/node-assetify "assetify for Node on GitHub"), my take on the matter.

I made some comments about `npm` back then, namely:

> - Node.JS is just a few years old (it was born in _2009_)
> - Some areas, such as client-side asset managers, don't have a well defined **best library**
> - Publishing packages on [npm](https://npmjs.org/ "npmjs.org") is astonishingly easy
> - Large amounts of packages are available

I've been meaning to update my insights into the matter, as my previous post no longer reflects my thoughts on `npm`. Recently, I read [this post](http://blog.nodejitsu.com/npm-innovation-through-modularity "npm: innovation through modularity") on **nodejitsu**, and I've been meaning to come back to the subject since then.

http://blog.nodejitsu.com/npm-innovation-through-modularity
http://dontkry.com/posts/code/browserify-and-the-universal-module-definition.html
http://dontkry.com/posts/code/using-npm-on-the-client-side.html
http://www.futurealoof.com/posts/nodemodules-in-git.html