![grunt.png][1]

#### Nobody has time for a full `grunt build` every _2s_

Does `grunt build` automates your builds? Awesome! Who automates `grunt build`? `grunt watch` will! If you're like me, you hit save, or change tabs every few seconds. You can't afford to run a full build every time you change a comment or a comma, because that'd be a tremendous waste of your time. Yet, a lot of people do this, because they haven't found a better way to go about it yet. You're reading this, so you're one foot ahead. Kudos. Let's start off by installing `grunt-watch`.

```shell
npm install grunt-contrib-watch --save-dev
```

If you haven't already, also install `load-grunt-tasks`. Then replace all your `grunt.loadNpmTasks` calls with:

```js
require('load-grunt-tasks')(grunt);
```

That will save you some time in the long run. Now, `grunt-contrib-watch` is fairly easy to set up. Here's a sample:

```js
{
  "watch": {
    "rebuild": {
      "tasks": [
        "build:rebuild"
      ],
      "files": [
        "Gruntfile.js",
        "build/**/*.js"
      ]
    },
    "jshint_client": {
      "tasks": [
        "jshint:client"
      ],
      "files": [
        "src/client/js/**/*.js"
      ]
    },
    "jshint_server": {
      "tasks": [
        "jshint:server"
      ],
      "files": [
        "src/srv/**/*.js",
        "app.js"
      ]
    },
    "css": {
      "tasks": [
        "css:debug"
      ],
      "files": [
        "src/client/css/**/*.styl"
      ]
    }
  }
}
```

This will run `jshint` on my client-side code when it changes, on the server-side code when that changes, it will run the `css:debug` task whenever one of my Stylus stylesheets changes, and it will run the entire build process if it _itself_ changes. The list of tasks goes on, but I didn't want to cloud you with lists of tasks and files, you get the idea.

> Watch only for the files that _directly affect a build task_, and run that particular task _(or set of tasks)_ when one or more of those files change. Repeat this for every build task you perform.

This helps you to avoid running the build process by yourself whenever something changes, and at the same time it'll be way faster, because only the necessary tasks will run at each point. Remember to clean up before running tasks, using `grunt-contrib-clean`.

Now all you need to do is set up a `dev` alias or similar, and make it look like this:

```js
grunt.registerTask('dev', ['build:debug', 'watch']);
```

Awesome, you're halfway there.

#### Ain't nobody got time fo' dat! <kbd>ctrl</kbd><kbd>c</kbd>,<kbd>↑</kbd>,<kbd>↵</kbd>

![aint-nobody-got-time.jpg][2]

That's the sequence of keys you often find yourself typing when you're not using `nodemon`, and you should be embarrased. We won't be using [nodemon](https://github.com/remy/nodemon "nodemon on GitHub") directly, because that's too cumbersome. Instead, we want to integrate it with the bunch of `watch` task targets we're using. The problem is, both [grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch "grunt-contrib-watch on GitHub") and [grunt-nodemon](https://github.com/ChrisWren/grunt-nodemon "grunt-nodemon") are _blocking_. Meaning: these tasks are never supposed to end. That represents a problem when attempting to run them serially, like **Grunt** is used to do. We want them to run _side-by-side_, like the best friends in the world they are. Enter [grunt-concurrent](https://github.com/sindresorhus/grunt-concurrent "grunt-concurrent on GitHub"). `grunt-concurrent` solves that problem by spawning new processes for each task it's required to run. Something like this will do the trick, parallelizing `watch` and `nodemon`:

```js
concurrent: {
  dev: {
    options: {
      logConcurrentOutput: true
    },
    tasks: ['watch', 'nodemon:dev']
  }
}
```

That'll do it. but how should we configure the `nodemon` task? Glad you asked.

```js
nodemon: {
  dev: {
    options: {
      file: 'app.js'
    }
  }
}
```

There's a little thing, too. `nodemon` is kind of silly, in that it watches for almost everything. Well, it defaults to `.js`, and `.coffee` files, but that includes stuff in `node_modules`, too. If we use a `.nodemonignore` file, such as the one below, we can ignore stuff that doesn't really affect `node` itself.

```
# seriously? ignore git changes
./.git/*

# package control
./node_modules/*
./bower_components/*

# logs
./npm_debug.log

# build artifacts
./bin/*

# deployment artifacts
./deploy/*

# os artifacts
.DS_Store

# ignore client-side js
./src/client/*

# ignore tests
./test/*
```

Oh, and remember to update your `dev` alias, too.

```js
grunt.registerTask('dev', ['build:debug', 'concurrent:dev']);
```

Done! _Next!_

#### Automate all the things! Drop <kbd>ctrl</kbd><kbd>s</kbd> forever

![automate-all-the-things.jpg][3]

Simple, use a text editor that is dilligent enough that it'll do the saving for you. End the <kbd>ctrl</kbd><kbd>s</kbd> non-sense!

- Using **WebStorm**? You're [golden](http://www.jetbrains.com/webstorm/webhelp/saving-and-reverting-changes.html#3)!
- **vim** is your peanut butter? then you could [use this link](http://stackoverflow.com/q/4637575/389745 "How can I make Vim autosave files when it loses focus?").
- **Sublime Text** and its many cursors make you the happiest assembly line worker? [Here you go!](http://superuser.com/q/366132/48116 "Possible for Sublime Text to save on lost focus?")
- Your text editor doesn't auto-save? _**Ditch it!**_

It will feel kind of weird at first, but as you get used to it, you'll fall in love and never look back.

## Refreshing the browser by hand? No way! Forget <kbd>ctrl</kbd><kbd>r</kbd>

Almost! Okay, we've now fully automated everything. Pretty much. We could throw-in `livereload`, since it's even bundled together in `grunt-contrib-watch` now. First and foremost, [install the browser extension](http://feedback.livereload.com/knowledgebase/articles/86242-how-do-i-install-and-use-the-browser-extensions- "How do I install and use the browser extensions?") for **livereload**.

Then, it's just a matter of adding a target to the `watch` task.

```js
watch: {
  livereload: {
    options: {
      livereload: true
    },
    files: [
      'public/**/*.{css,js}',
      'views/**/*.html'
    ]
  }
}
```

No more <kbd>F5</kbd>? Sign me up!

# Unbox it

![unbox-256.png][4]

I iterate over my build processes a lot, and now I finally put it together in a project that's ready to clone and work on. I didn't want to spend a bunch of time copying and pasting every time, and I figured it'd be useful to you too. Without further ado, I present [unbox](https://github.com/bevacqua/unbox "unbox on GitHub") to you. Clone using the command below:

```shell
git clone https://github.com/bevacqua/unbox my-repo
cd my-repo
cat unbox.sh | sh
```

Doing that will:

- `git clone` the latest version of `unbox`
- Remove the `.git` folder to avoid confusion
- `npm install`
- `bower install`
- **Profit!**

> It doesn't just provide a build process, but an opinionated way to lay out the architecture, build process, and folder structure of any new application you want to develop with Node.

Let me know if you find this kind of module to be _useful_, I sure do!

# Updated `grunt-ec2`!

By the way, I've updated [grunt-ec2](https://github.com/bevacqua/grunt-ec2 "grunt-ec2 on GitHub"), introduced [in this post](http://blog.ponyfoo.com/2013/09/19/deploying-node-apps-to-aws-using-grunt) if you haven't read that yet, and it now has more features!

- Port forwarding
- `nginx`! This one made me a really happy pony
- Proper configuration, setting `NODE_ENV` to the name tag we're using
- Hard reboots of the EC2 instance, `pm2`, or `nginx`
- Fine grained control over your deployed application without having to `ssh` into the EC2 instance by yourself

You can look at [the complete change log](https://github.com/bevacqua/grunt-ec2/blob/master/CHANGELOG.markdown "grunt-ec2 change log on GitHub") on GitHub.

Oh, I didn't see you there! You see, I'm _writing a book_ on this kind of things. If **build processes, application architecture, and JavaScript** are things that warm your noodles, then _stay tuned for updates_ about my book!


  [1]: https://i.imgur.com/EyXjS8r.png "Grunt! JavaScript Task Runner"
  [2]: https://i.imgur.com/LkMiobQ.jpg "Ain't nobody got time fo dat!"
  [3]: https://i.imgur.com/AAQ9riH.jpg "Automate all the things!"
  [4]: https://i.imgur.com/5EwdJvU.png "unbox-256.png"
