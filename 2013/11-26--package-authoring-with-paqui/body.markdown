In essence, Paqui takes care of three things:

- Scaffolding
- Building
- Distributing

Let's go through them in turn.

# Paqui Flow

The first step is actually [creating a repository](https://github.com/new "Create a New Repository on GitHub") on GitHub. Paqui will take care of everything else, so you should leave **Initialize** _unchecked_.

![github.png][1]

Paqui requires a one-time only installation step, let's go ahead and type that into our terminal now.

```shell
npm install -g paqui
```

Ready! Okay, let's see how this works. The first command we'll look at is `paqui init`.

```shell
paqui init paqui-dummy --remote bevacqua/paqui-dummy
```

I use the `--remote` option to have Paqui _add the remote and push to it_ for me, otherwise I'd need to do that by hand. Note that you can use both the full url or the shorthand `{user}/{repo}` syntax.

This command will create a `paqui-dummy` directory and set up our component there. If we create the directory ourselves, and `cd` into it, we could just `paqui init .`. Before doing anything, Paqui will ask us a few questions, but it'll provide default answers to the best of its knowledge.

![init.png][2]  
> _I used the `--lean` option to avoid verbose `git` command output_

The configuration provided to Paqui when we initialize our component can be modified at any time in `.paquirc`, which is the only file Paqui will burden us with.

![rc.png][3]

At this point we can start hacking away at our package. If we didn't change any of the defaults, the entry point will be `src/main.js`.

## Building

By default, we are going to write our code under Common.JS (using `module.exports` and `require`). If you scroll up a bit you'll see one of the questions Paqui asked was `transform`, where I answered with the default: `universal,banner`. This is where the extensible nature of Paqui becomes apparent.

> The only thing that's governing the way we write our module is the **transformation extensions we apply to our sources** during our builds.

As you can imagine, `universal` means that we are able to write Common.JS modules in our source code, and the `universal` transform will take that code, put it through `browserify`, and turn it into a client-side package that supports [the UMD definition](https://github.com/umdjs/umd "Universal Module Definition").

Paqui is going to generate a couple of files during builds, in this case they'll be called `paqui-dummy.js` and `paqui-dummy.min.js`. These will be the files that package consumers will pay attention to.

![build.png][4]

## Versioning

Now we'll want to publish our package to package management systems. First, we have to deal with versioning, an important part of package management. With Paqui, the **only** _version number that matters_ is the one in `.paquirc`. That version number will be bumped with `paqui bump`.

Paqui also _does a little more_ when we call bump the first time around. It will generate files (such as `package.json`, and `bower.json`) filled with information about the component.

![bump.png][5]

Missing information, such as the `main` script reference some package managers require, is added during the _build_ step. However, you don't need to worry about that, since `paqui deploy` will always run `bump` _before_ it builds. It is always worth checking out what Paqui has generated, though. At least until you get the gist of how it works.

## Publishing

The last step Paqui takes care of is publishing to the various package managers we opt to use. With Paqui, this can be done simply executing `paqui publish` in our command line.

Paqui conveniently packages `bump`, `build`, and `publish` in another command: `paqui deploy`. The flow I suggest would be:

- Create a repo on GitHub
- `paqui init -r username/project`
- `paqui bump`, then verify generated files such as `package.json`, or `bower.json` _*_
- Work on your client-side package
- `paqui build`, validate build results, located in the `bin` directory _*_
- `paqui deploy`

_* (optional steps)_

So we have scaffolding, version bumping, a build step, and publishing. How can we configure these things?

## Configuring `paqui init`

The following table shows data requested by `init`, as well as its defaults.

Property|Description|Defaults
---:|----|----
_name_|Package name|Project directory name
_description_|Pretty self explanatory|Awesome package
_author_|Package author|Info from `git config --global --get`
_version_|Package version|`0.0.1`
_transform_|Build steps|`['universal', 'banner']`
_transport_|Build results|`['file']`
_pm_|Package Managers|`['npm', 'bower']`
_license_|License under which we release the package|**MIT**
_remote_|The git remote we will be pushing to|`origin`
_main_|The path to the entry point for our source code|`'src/main.js'`

The results from these questions asked by Paqui, will be dumped into a JSON file called `.paquirc`, where Paqui keeps everything needed to help you build and release your project.

The relevant command-line options are:

Option|Description
---:|---
`--remote <url>`|Provide a link to the remote, then init will automatically push to it after creating the local repository
`--no-git`|Paqui won't create a git repository using `git init`, nor attempt to push anything
`--existing`|Creates a `.paquirc` for an existing project

> Using `paqui init --existing` is useful to integrate existing projects into Paqui.

The four options which might be unclear at this point are: `transform`, `transport`, `pm`, and `license`. In the case of the last one, I decided not to make licesing extendable. At this point you can pick one of: [Apache](http://choosealicense.com/licenses/apache/ "Apache v2 License on ChooseALicense.com"), [GPL](http://choosealicense.com/licenses/gpl-v3/ "GPL v3 License on ChooseALicense.com"), or [MIT](http://choosealicense.com/licenses/mit/ "MIT License on ChooseALicense.com"). If you'd like to pick another license from Paqui, you're welcome to [open a pull request](https://github.com/bevacqua/paqui/pulls/ "Pull Requests on Paqui") with more templates.

## Extending Paqui

The three remaning options are _extendable_. The `transform` extensions govern _how your project is compiled_. These can be chained, and each transformer's output is piped onto the next one. By default, Paqui projects are compiled using the `universal` and `banner` extensions. The former wraps our code in a UMD definition, using Browserify. Then, `banner` prepends a comment with author information, such as the version number, license type, or package name.

To illustrate, let's examine the function exported by `banner.js`, the file which defines its namesake extension. This function takes a `paqui` argument, which is a succint API helper provided by Paqui. The function should return an object, and in this case, that object should have a `transform` property, as _a function with three arguments_:

- `pkg`: a read-only copy of the contents of `.paquirc`
- `model`: an empty object which can be useful to communicate among different extensions
- `done`: callback to be executed when the extension's work is over, passing an optional error and the resulting code

The current implementation is below.

```js
function (paqui) {
    return {
        transform: function (pkg, model, done) {
            var main = path.join(paqui.wd, pkg.main);
            var raw = '';
            var b = browserify(main);
            var stream = b.bundle({
                standalone: pkg.name
            });

            stream.on('data', function (data) {
                raw += data;
            });

            stream.on('end', function () {
                done(null, raw);
            });
        }
    };
};
```

As you can see, all this does is take the `main` path specified in `.paquirc`, join it with the working directory provided by the Paqui API, and turn it into an UMD module, before passing it to `done(null, raw)`. Using this micro-framework we'll be able to put together any kind of build step we want in just a few lines of code. The difference is that there's almost no configuration after that: we just provide the names to the plugins we want to use.

These plugins are also available to extend what's done after builds, enabling us to do simple things such as writing to a file, or maybe we want to get creative and send an email, or print a report to the standard output. Similarly, we're able to add more package management systems on top of the ones provided by default, which are `npm`, `bower`, and `component`.

For more information on the **Paqui API** and to learn how to _extend its functionality_, [visit its GitHub repository](https://github.com/bevacqua/paqui).

  [1]: https://i.imgur.com/XAlzQ8V.png "Creating the repository on GitHub"
  [2]: https://i.imgur.com/i0grZjO.png "Initializing a component with Paqui"
  [3]: https://i.imgur.com/Tx6ehpE.png "The .paquirc configuration file"
  [4]: https://i.imgur.com/4dx3YHd.png "Building a component with Paqui"
  [5]: https://i.imgur.com/onnWEaI.png "Version bumping!"
