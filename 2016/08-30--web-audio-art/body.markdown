## A little theory

First, let's look at one of the many audio formats out there: PCM. PCM is an acronym for 'Pulse Code Modulation'. The PCM file format allows uncompressed audio data storage. You've probably played PCM files before - common file extensions that are in PCM are .WAV, .PCM, .AIFF, and more.

So what does a PCM file look like? Let's examine a WAVE file to see. A WAVE file has three distinct sections to it - the RIFF chunk, format chunk, and data chunk.

![diagram of file icon with RFF, FMT and DATA labeled thirds][1]

The RIFF chunk briefly describes details such as what type of file it is. The FMT or format chunk contains important information including the type of compression, how many channels of audio are present, the sample rate, average bytes per second, etc etc.

We are most concerned with the last one - the data chunk. This contains the PCM data itself. Each data point is an individual sample of the audio content. The value of each sample ranges from -1 to +1. This is what we'll be working with when analysing audio sources. The diagram below shows how the audio is recorded digitally by sampling the sound waves at tiny intervals:

![diagram of pcm samples][6]

Note: each audio channel in the WAVE file (eg. 'left' and 'right' channels) are interleaved within the data. This means, that if our audio file has 2 channels, each sample will output both channels' values next to each other in the PCM data chunk. This is easy to deal with, as our format chunk tells us how many channels are in the WAVE file. 

## Extracting PCM data in the browser

The Web Audio API offers a method we can use to extract the PCM data from any PCM file. The method is `decodeAudioData`. It takes an array buffer of the PCM file contents, and a callback function for when the data has been successfully decoded.

In this example, a WAVE file will be used from [NASA's Golden Record: Greetings to the Universe (listen)](https://soundcloud.com/nasa/sets/golden-record-greetings-to-the). Let's choose the [English greeting (listen)](https://soundcloud.com/nasa/golden-record-english-greeting?in=nasa/sets/golden-record-greetings-to-the).

The first step is to get the contents of the WAVE file in array buffer format. The [Fetch API](https://jakearchibald.com/2015/thats-so-fetch/) will do the job well for this. If Fetch is not available in your browser of choice, the [XMLHttpRequest API](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) will also work. We return the WAVE file in an `arrayBuffer`, and create a new `Int8Array` 'view' so that we can read the individual elements in a console log to ensure everything worked as expected. 

Putting it all together, we get the code below:

```javascript

fetch('English.wav')
  .then((response) => response.arrayBuffer())
  .then((fileBuffer) => {
    console.log(new Int8Array(fileBuffer)); // [82, 73, 70, 70, 104, 16, 1, 0, 87, ...]
  });
```

Once we have the file contents in the correct form, let's create a new `AudioContext` and decode the audio data. The method to call from the AudioContext is `decodeAudioData`. For arguments, it takes our `arrayBuffer` returned after the fetch from before, and a callback with the decoded data passed in. From there, the decoded data is wrapped in some convenience methods, including getting channel data. For simplicity, we'll grab just one channel of audio data (the first one). 

Let's look at what the code looks like now:

```javascript

fetch('English.wav')
  .then((response) => response.arrayBuffer())
  .then((fileBuffer) => {
    // create new Web Audio API context
    let audioCtx = new AudioContext();
    
    audioCtx.decodeAudioData(fileBuffer, (audioData) => {
      // get the first set of channel data only
      let channelData = audioData.getChannelData(0);
      console.log(channelData); // [0.02567136287689209, 0.026834938675165176, ...]
    });
  });
```

That console log of `channelData` will show the PCM values we're looking for. Now let's do something cool with this data!


## Transforming PCM data

At this point, the PCM data is just a set of raw numbers. Numbers are really versatile to work with, and can be mapped to other contexts.

Keeping to our art in the browser theme, what if we transformed the audio data into something visual. What does sound "look" like? This can be approached in a myriad of ways, but let's keep this simple. What if each PCM sample was mapped to a color? And that color was applied to a pixel? By using a canvas element, we can explore this. First step - create a canvas element and grab its 2D context:

```javascript
// within decodeAudioData callback
let c = document.createElement('canvas');
let ctx = c.getContext('2d');
```

What has this given us? With a canvas element, we essentially have a blank playground to start 'painting' with the PCM audio data. One literal way of representing the PCM values is to paint one pixel on the canvas for each value. If they're all lined up, some patterns might even emerge!

Intuitively, we can size the canvas based on how many pixels it's going to have. The amount of pixels is equal to the length of the PCM data (the single channel we extracted). Based on this, we can also calculate how wide and tall the final canvas element should be.

Squares are a great shape to work with, as they're pleasing to look at, and the math to produce them is relatively minimal. Let's go with drawing a square shape out of the PCM data pixels. 

The next step is to use some math to figure out the height and width of the square 📐📏. Using the length of the `channelData` array, calculating the square root will give us how long each side of the square should be.

After creating the canvas element, setting the width and height property looks like the code example below:

```javascript
// within decodeAudioData callback
let c = document.createElement('canvas');
let ctx = c.getContext('2d');

// math!
let size = Math.sqrt(channelData.length);
c.height = size;
c.width = size;
```

Once the canvas element is the right physical size, we need an image data object to manipulate before 'painting' it onto the square canvas. Canvas elements have a method called `createImageData` which returns an RGBA (red/green/blue/alpha) image data array. Because each pixel has four values (three colors and alpha), the image data array is four times as long as the `channelData` array. We just need to keep this in mind when setting each pixel up. Each pixel will require four adjacent values to be changed.

The last line in the code below sets up an image data array and assigns it to the variable `imgData`:

```javascript
// within decodeAudioData callback
let c = document.createElement('canvas');
let ctx = c.getContext('2d');

// math!
let size = Math.sqrt(channelData.length);
c.height = size;
c.width = size;

// canvas data
let imgData = ctx.createImageData(size, size);
```

Ok! Now for the fun part. Each PCM sample value is mapped to one pixel, as mentioned earlier. Using HSL, or Hue / Saturation / Luminosity in this kind of example is much more straightforward to get visually pleasing results for coloring the pixel. "But canvas image data arrays use RGBA," I hear you protest. That's ok! These values will need to be converted to RGBA later on to resolve this nit, but coding the loop over the PCM samples can happen first. 

The loop examines every PCM data value in `channelData`, and uses the value to influence its associated pixel's hue. The first PCM data value produces the first pixel in the canvas image data array, and so on until all PCM data values have produced a pixel. Therefore the canvas image data array is filled out from left to right, row by row until the entire loop is done.

The final loop is below:

```javascript
channelData.forEach((v, i) => {
  // clamp the data value to a scale of 0-255
  let val = Math.ceil((v + 1) * 255 / 2);
  
  // get color of pixel in RGBA (we'll write the function hslToRgba next)
  // play with the 0.74 value to tweak what kind of colors are returned -  fun!
  let rgba = hslToRgba(val / 0.74, 255, 150, 255);
  
  // starting point in the canvas image data
  // there are four elements per pixel in canvas data arrays (R, G, B, and A)
  let start = i * 4;
  
  // set one canvas pixel to RGBA values
  // loop over the 4 adjacent values the pixel has in the canvas image data
  rgba.forEach((v, i) => {
    imgData.data[start + i] = v;
  });
});
```

Now for the `hslToRgba` function. A very common implementation is below. Complicated math ahead 📐📏 ! We don't need to worry too much about what all of this means. Color theory is a fascinating topic, but not one we have time for today.


```javascript
function hslToRgba(h, s, l, a) {
  let r, g, b; 

  h = h / 255;
  s = s / 255;
  l = l / 255;
	
  function convert(p, q, t) {
    if (t < 0) t += 1;
    if (t > 1) t -= 1;
    if (t < 1 / 6) return p + (q - p) * 6 * t;
    if (t < 1 / 2) return q;
    if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6;
    return p;
  }
	   
  let q = l < 0.5 ? l * (1 + s) : l + s - l * s;
  let p = 2 * l - q;
  r = convert(p, q, h + 1 / 3);
  g = convert(p, q, h);
  b = convert(p, q, h - 1 / 3);
	
  return [r * 255, g * 255, b * 255, a];
}
```

The most difficult parts are done. Finally, let's populate the canvas element's pixels, and place the final result on the page! We can take the finished `imgData` array, and pass it into the canvas element's `putImageData` method. The last two arguments specify the x and y coordinates of where to start placing the image data. Our canvas element is exactly the right size, so 0 and 0 make sense.

```javascript
// put the pixels on the canvas
ctx.putImageData(imgData, 0, 0);

// place the canvas into the HTML document
document.body.appendChild(c);
```

## Results

When NASA's Golden Record: English Greeting is run through the code we wrote, we get the following:

![NASA's Golden Record English Greeting displayed in pixels][2]

Pretty cool, huh! If you study the visual closely, you might be able to make out each individual word, and the slight artifact at the end of the sound file producing the soft pink banding on the bottom. Neato.

Tweaking the first argument in the `hslToRgba` method call will change up which colors will output onto the canvas. I encourage you to play with it and have some fun!

🌊  Here's what the sound of a single crashing ocean wave looks like:

![a crashing ocean sound displayed in pixels][3]

🐓 A rooster crowing looks like this:

![rooster crowing displayed in pixels][7]

## Taking it further

What if we fed the pixel data back into the Web Audio API, to test if it sounds the same as the original? Let's try it! The following code will turn pixels back into sound data again. We have to create a new audio buffer to put the sound data into, in a similar way to the canvas image data array. 

Calling `createBuffer` on the audio context is the way to do this:

```javascript
// create a new single channel audio buffer to put sound into
let soundBuffer = audioCtx.createBuffer(1, channelData.length, audioCtx.sampleRate);
// get channel data for mutating
let channelBuffer = soundBuffer.getChannelData(0);
```

Taking the new `soundBuffer`, we can loop through and transform the red value of each pixel back into a value between -1 and +1. Each data or sample value within the `soundBuffer` object will be assigned in this way, until it's full of PCM values. 

Take a look below to see how the loop works:

```javascript
for (let i = 0, l = channelData.length; i < l; i++) {
  // take the red value of each pixel only for simplicity
  // audio needs to be in -1.0 - 1.0
  // 0 = -1, 255 = 1
  channelBuffer[i] = ((imgData.data[i*4] / 255)  * 2 - 1);
}
```

Then, it's a matter of connecting this new sound data to a new audio context source, and then start playing it 📢 !

```javascript
let source = audioCtx.createBufferSource();

// set the source to be the buffer we created earlier
source.buffer = soundBuffer;

// connect the source to the destination so we can hear the sound
source.connect(audioCtx.destination);

// start the source playing!
source.start();
```

You should hear the WAVE file playing back to you. What happens if those HSL values are tweaked in the pixel loop we wrote earlier? Give that a try yourself. Notice how the sound distorts in different ways? Cool!

You can view this entire example (and fork it) at my [codepen](http://codepen.io/noopkat/pen/QKLbxL?editors=0010#0) - I can't wait to see what you make! 🎉

This is one simple example of the ways you can manipulate something from one format into another in order to create browser art. There are ways to really go to town on this, generating complex visual rules for each PCM data sample. Consider using extra tools, such as [three.js](http://threejs.org) and [D3.js](https://d3js.org). The sky is the limit! For a more abstract application of audio art, check out my ["I love the subway"][5] piece, which attempts to create a song, complete with cats playing guitars! 😸🎸

[![cats on the subway][4]][5]

> What are some cool hacks you've made using the Web Audio API?

  [1]: https://i.imgur.com/8snMhDu.png
  [2]: https://i.imgur.com/OWIf9QZ.png
  [3]: https://i.imgur.com/3m29l7y.png
  [4]: https://i.imgur.com/zQqZBkc.png
  [5]: http://noopkat.github.io/iltsw/index5.html "I love the subway by Suz Hinton"
  [6]: https://i.imgur.com/Bn1dKfl.png
  [7]: https://i.imgur.com/EIrOMvj.png
