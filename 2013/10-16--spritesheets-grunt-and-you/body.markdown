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
  sprite: {
    tools: sprite('tools', 'tl')
  }
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

  [1]: https://i.imgur.com/1ud2mRR.gif "An spritesheet, used in Megaman"
  [2]: https://i.imgur.com/5pRLV9c.png "Just drag and drop!"
