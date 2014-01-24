# Spritesheets, Grunt, and You

If you are using Grunt, you really have **no excuse** not to be using CSS spritesheets. If you aren't using Grunt _yet_, then you should know that a well thought-out workflow with Grunt will allow you to _seamlessly integrate icons together_ into a spritesheet during your builds.

In case you've been living undersea for the last decade, I wonder how you're still breathing. Sprites are [a very old concept][1], originating in game engines, which were more performant when they just loaded a _single graphic_ containing all the frames needed to render an animation. Today, spritesheets are a well-known improvement over using a single image for each icon in modern websites, but they are still slower to gain ground than they could be.

Grunt makes this super-easy for us, and in this article I want to demonstrate exactly how I use it to generate spritesheets.

  [1]: http://en.wikipedia.org/wiki/Sprite_(computer_graphics) "History of Sprites in Wikipedia"

![spritesheet.gif][1]

The first thing we'll want to install is `grunt-spritesmith`.

```shell
npm install --save-dev grunt-spritesmith
```

Configuring it is kind of tricky to get right, but I've put together a function to make it easier for us to set it up.

```js
function sprite (type, short) {
  return {
    src: 'src/img/sprite/' + type + '/**/*.{png,jpg,gif}',
    destImg: 'bin/public/img/sprite/' + type + '.png',
    destCSS: 'bin/public/css/sprite/' + type + '.css',
    imgPath: '/img/sprite/' + type + '.png',
    cssOpts: {
      cssClass: function (item) {
        var prefix = short ? short + '-' : '';
        return '.' + prefix + item.name + ':before';
      }
    },
    engine: 'gm',
      imgOpts: {
        format: 'png'
      }
  };
}
```

Say we want to set up a **"tools"** spritesheet, with a bunch of tool icons we have. We can simply configure Grunt like below.

```js
grunt.initConfig({
  tools: sprite('tools', 'tl')
});
```

Lastly, we'll need a few extra tweaks to our CSS so that it works in a `:before` pseudo-element.

```css
.icon:before {
  vertical-align: bottom;
  display: inline-block;
  content: '';
}
```

That's it, we're now able to render sprited icons in our HTML, effortlessly.

```html
<span class="icon tl-screw"></span>
```

The tremendous upside is that we can drag and drop new icons into our source folder, and immediately build a spritesheet again. Something that used to take _pestering the designer_, or considerable effort on **Paint** on our part, is now just a matter of typing `grunt sprite` into the terminal.

![icons-directory.png][2]

Fully working [source code here](https://github.com/bevacqua/grunt-spriting-example "grunt-spriting-example on GitHub").

  [1]: http://i.imgur.com/1ud2mRR.gif "An spritesheet, used in Megaman"
  [2]: http://i.imgur.com/5pRLV9c.png "Just drag and drop!"