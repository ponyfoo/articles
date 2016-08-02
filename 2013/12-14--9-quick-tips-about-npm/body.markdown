## Manage `node` and `npm` versions easily

Using [`nvm`](https://github.com/creationix/nvm "creationix/nvm on GitHub"), you won't ever have any versioning or sudo issues with `npm` again. I can't recommend this highly enough!

```shell
curl https://raw.github.com/creationix/nvm/master/install.sh | sh
```

Reload your terminal, and then start by installing some other version.

```shell
nvm install 0.10.22
nvm alias default 0.10.22
```

Switching to another version is also really easy, and you can copy modules over when you upgrade, with `nvm copy-packages <previous-version>`.

## Use modules directly from the source

Rather than installing a package every time it gets updated, you can use the [`npm link`](https://npmjs.org/doc/cli/npm-link.html "npm link documentation") command. This command will create a symbolic link which you can later consume with `npm link <pkg>`.

As an example, have the following illustrative shell session.

```shell
git clone https://github.com/bevacqua/grunt-ec2.git
cd grunt-ec2
npm link
cd ../site
npm link grunt-ec2
grunt ec2_deploy:production
```

If you make any changes to `grunt-ec2` in your local version, they will have an immediate effect on the package installed in `site`, as well. This one is also very useful if you're a package author, as you can quickly test your package in a real usage scenario without having to publish it to `npm` first.

## `npm init` in style

This one is rather straightforward. You can set some values in `npm`'s configuration, and then using `npm init` will use them when creating a `package.json` file.

```shell
# add author info to npm
npm set init.author.name "$NAME"
npm set init.author.email "$EMAIL"
npm set init.author.url "$SITE"
npm init
```

## Update any package to latest version

Firstly, you can get the _currently installed version_ of a package by executing:

```js
npm view <pkg> version
```

If you set the version of a few modules in `package.json` to `*`, then running `npm update --save` will update all those modules to the latest stable version, and since we've used the `--save` flag, the version numbers will get persisted _(overwriting the stars)_. This is most useful in recently started projects, but should be treated carefully in solutions which are already in production, as authors might break between updates.

## Clean the `npm` cache

If you're having any issues with an `npm` package, you might have to remove it from the `npm` cache before attempting to install it again.

```shell
npm clean <pkg>
npm install <pkg>
```

## Publish `npm` modules faster!

You can do that using an [optimized npm publish](https://github.com/dominictarr/npm-atomic-publish "npm-atomic-publish on GitHub") created by [dominictarr](https://github.com/dominictarr "dominictarr on GitHub").

```
npm i -g npm-atomic-publish
npm-atomic-publish
```

## Save time, use shortcuts

- Rather than `npm install`, use `npm i`.
- `npm uninstall` becomes `npm r`
- Save packages directly in one step with `npm i --save <pkg>`
- For `devDependencies`, do `npm i --save-dev <pkg>`

## Bump package version with `npm`!

Just do `npm version <v>`! It'll even commit and tag for you, if you're in the context of a `git` repository.

```shell
npm version 2.1.0
npm publish
```

## Use Browserify to get Common.JS in browser-land!

If you want to access 50+ thousand packages _(and rapidly growing!)_ in your browser, and you're starting to get tired of the AMD module definition, you might want to give [Browserify](https://github.com/substack/node-browserify "browserify on GitHub") a spin.  You'll be able to use the CJS notation in the browser, and you're even able to port modules that used the file system, thanks to some [creative shimmery by substack](https://github.com/substack/node-browserify#compatibility "Browserify compatibility").

And, of course, there's [a Grunt plugin](https://github.com/jmreidy/grunt-browserify "grunt-browserify on GitHub") for that!

## Read all the way to the end? Have something extra!

- [Managing Node.js Dependencies with Shrinkwrap](http://blog.nodejs.org/2012/02/27/managing-node-js-dependencies-with-shrinkwrap/ "Managing Node.js Dependencies with Shrinkwrap")
- Use `--prefix <dir>` to install packages in a given directory, rather than working directory
- Don't uppercase, `npm` is really `npm`, not `NPM`, not `Npm`. Just `npm`. That's it. [Source](https://npmjs.org/doc/faq.html "npm documentation FAQ"):

> Contrary to the belief of many, "npm" is not in fact an abbreviation for "Node Package Manager". It is a recursive bacronymic abbreviation for "npm is not an acronym". (If it was "ninaa", then it would be an acronym, and thus incorrectly named.)
>
> "NPM", however, is an acronym (more precisely, a capitonym) for the National Association of Pastoral Musicians. You can learn more about them at http://npm.org/.
>
> In software, "NPM" is a Non-Parametric Mapping utility written by Chris Rorden. You can analyze pictures of brains with it. Learn more about the (capitalized) NPM program at http://www.cabiatl.com/mricro/npm/.
>
> The first seed that eventually grew into this flower was a bash utility named "pm", which was a shortened descendent of "pkgmakeinst", a bash function that was used to install various different things on different platforms, most often using Yahoo's yinst. If npm was ever an acronym for anything, it was node pm or maybe new pm.
>
>So, in all seriousness, the "npm" project is named after its command-line utility, which was organically selected to be easily typed by a right-handed programmer using a US QWERTY keyboard layout, ending with the right-ring-finger in a postition to type the - key for flags and other command-line arguments. That command-line utility is always lower-case, though it starts most sentences it is a part of.

A package is:

1. a folder containing a program described by a `package.json` file
2. a gzipped tarball containing **[1]**
3. a url that resolves to **[2]**
4. a `<name>@<version>` that is published on the registry with **[3]**
5. a `<name>@<tag>` that points to **[4]**
6. a `<name>` that has a "latest" tag satisfying **[5]**
7. a git url that, when cloned, results in **[1]**.

So there you have it!

> What are your favorite `npm` tricks?

## Remove development dependencies

Using the command below, you can remove `devDependencies` after `npm install`. This is useful in some set ups where you deploy to a CI environment, which installs everything so that it can run your Grunt build, or something, and then deploys to the live servers.

```shell
npm prune --production
```

Woo, that's ten!
