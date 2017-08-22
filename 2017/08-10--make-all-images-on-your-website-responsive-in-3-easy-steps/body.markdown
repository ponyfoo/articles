## Down-Scaling Images

Using CSS or JavaScript to down-scale images only makes them *dimensionally* responsive. The following image illustrates this better:

::: .mde-inline.mde-50
![Down-scaling illustration on mobile](http://res.cloudinary.com/christekh/image/upload/c_crop,w_auto/v1501763693/Screen_Shot_2017-08-03_at_1.17.28_PM_aukf2g.png)
:::

::: .mde-inline.mde-50
![Down-scaling illustration on desktop](http://res.cloudinary.com/christekh/image/upload/c_crop,w_auto/v1501763696/Screen_Shot_2017-08-03_at_1.20.05_PM_ogvphp.png)
:::

Both on web and mobile devices, the size is still **8.9MB**. Bearing in mind that your mobile phones have less resources than your PC, we still have more work to do.

Using only a down-scaling approach is not ideal because it does not account for the size of the image; just its dimensions.

We have already seen that up-scaling wouldn't work either, hence we need something that handles both dimensions and size.

## Size-Fitting

Our best option is to generate images based on the screen size and render them. This process can be extremely complex, but there’s a shortcut that can help you automate much of the process. You can make all images responsive in three easy steps with [Cloudinary](http://cloudinary.com):

### 1. Include Cloudinary in Your Project

Add the Cloudinary SDK to your project simply by including it in your index.html using script tags:

```html
<script
  src="https://cdnjs.cloudflare.com/ajax/libs/cloudinary-core/2.3.0/cloudinary-core-shrinkwrap.min.js">
</script>
```

### 2. Add Images with `data-src`

You don't want the images rendered immediately, until JavaScript runs. Hence, include the image using the `data-src` attribute instead of `src`: 

```html
<img
  data-src="
    http://res.cloudinary.com/christekh/image/upload/w_auto,c_scale/v1501761946/pexels-photo-457044_etqwsd.jpg
  "
  alt=""
  class="cld-responsive" />
```

Using this approach, Cloudinary analyzes your browser screen first, resizes the image saved in Cloudinary storage as provided in `data-src`, and then renders the image in the appropriate size and dimension using JavaScript.

Two things to note from the tag:

- `w_auto,c_scale` transformation parameters tell Cloudinary to dynamically generate an image URL scaled to the correct width value, based on the detected width actually available for the image in the containing element.
- The class `cld-responsive` tells Cloudinary which images to apply this feature too.

### 3. JavaScript Call

Finally, initialize a Cloudinary instance in your JavaScript files and call the `responsive` method on this instance:

```js
const cl = cloudinary.Cloudinary.new({ cloud_name: 'YOUR_CLOUD_NAME' })
cl.responsive()
```

Remember to [create a free Cloudinary](https://cloudinary.com/users/register/free) account so you can receive a cloud name for configuring this instance.

This piece of code will walk through the DOM, find all image tags with the class `cld-responsive` to apply **size** and **dimension** fitting images on them.

## Final Words

Always keep in mind that when you use CSS like the following code below to make images responsive, it does not guarantee a good user experience:

```css
img {
  width: 100%;
  height: auto;
}
```
The sizes of these images remain the same. Large images on mobile devices eat up resources (like allocated memory and running processes) causing slow downloads or unexpected behavior on the user's device. Responsive images ensure that users save lots of data bandwidth & have great experiences when using your image-rich website or app.

Lastly, it’s good to keep in mind that the suggested approach relies on JavaScript. Therefore, the preloading capabilities provided by `srcset` and `sizes` are sacrificed and your browser must have JavaScript enabled for this feature to work.
