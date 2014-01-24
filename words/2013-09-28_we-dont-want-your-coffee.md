# We don't want your Coffee

_An open letter to the [CoffeeScript](http://coffeescript.org/) community._

This rant probably also holds true for [TypeScript](http://www.typescriptlang.org/ "TypeScript: Typed superset of JavaScript"), and similar. Heck, even [asm.js](http://kripken.github.io/mloc_emscripten_talk "Big Web App? Compile it!"). I know `asm` is awesome _in theory_, but have you actually tried going through [a piece of code](https://github.com/srijs/rusha/blob/master/rusha.js#L186 "rusha's hashing algorithm with asm.js on GitHub") written in that? It's _garbage_, and you **aren't really supposed to do that to your everyday code base** either.

Now, don't get me wrong, I'm completely fine with you sipping your fancy coffee _yourself_. My problem lies with you spilling coffee on everything you do, or trying to force-feed coffee to everyone else. Your coffee is becoming an infectious disease, and you need to stop it from spreading. Languages like CoffeeScript represent a problem to the community as a whole not because the language is inherently bad, but because you assume everyone else understands that non-sense.

> What? It's not non-sense, it's beautiful! It's JavaScript.

Newflash. It isn't JavaScript, it just compiles down to it. Stop trying to subjugate people on StackOverflow by answering their JavaScript questions with CoffeeScript code. Stop trying to get JavaScript help posting Coffee code we don't understand.

![coffee-script.jpg][1]

The problem isn't in the language itself. While I may not like CoffeScript personally, I celebrate the diversity, like pretty much everyone else. The issue is with people like yourself, who post things on the web as if everyone knew how CoffeeScript works. Sure, it _compiles down_ to JavaScript. We get that. But we're not going to learn your language, and it looks funny. 

I don't have to tell you things are bad, everybody knows things are bad. It's like everything everywhere is going crazy, so we don't go out anymore. We sit in the house and slowly the world we're living in is getting smaller, and all we say is _"please, at least leave us alone in our living rooms"_.

So, I want you to get up now.

[![mad-as-hell][2]](http://www.youtube.com/watch?v=WINDtlPXmmE "Mad As Hell - Network")

Did you know that some companies are [in fact turning their back on Coffee](http://meta.discourse.org/t/is-it-better-for-discourse-to-use-javascript-or-coffeescript/3153 "Is it better for Discourse to use JavaScript or CoffeeScript?"), just because people don't really know the language? The problem **Discourse** encountered was simply that they weren't getting as many contributions as they expected simply because their JavaScript was obfuscated behind Coffee, reason enough for them to flip the switch and move away from CoffeeScript. The problem is clear: people who know Coffee should know JavaScript if they want to write decent code. The opposite doesn't really hold true, the rest of the universe has no reason to learn Coffee, and those of us who don't buy into its perceived value are left out in the cold.

I'm not even getting into _the [which one's better?](http://wekeroad.com/2012/03/21/coffeescript-or-straight-up-js-i-suck-either-way "CoffeeScript or Straight Up Javascript? It's Decision Time") debate_. I don't really like the syntax but then again, I don't care that some people do, either. Besides, others have typed [pretty long articles](http://ryanflorence.com/2011/case-against-coffeescript/ "Case Against CoffeeScript") on the subject. After all, the first non-procedural language I came upon was **Visual Basic**, which has the same_ish_ "human-readable" syntax style that we find in CoffeeScript, with gems such as `one isnt two` reminding me that once upon a time I, too, relinquished code quality in favor of blindly telling my compiler `On Error Resume Next`, an obscure construct in VB where it acted as if every single line was wrapped in a `try`/`catch` block. Oh, the joy of blindly and naively debugging code back then.

> The problem with the child-like syntax is that it hides too much, making the code nearly impossible to read for un-caffeinated fellows such as myself. Not that we want to. This isn't the case of all languages that compile to web-ready languages _(HTML, JS, CSS)_.

Jade, for example, produces clean HTML that simply uses CSS selectors, in a syntax something very similar to what you might know as the [Zen Coding](http://coding.smashingmagazine.com/2009/11/21/zen-coding-a-new-way-to-write-html-code/). When you compare Jade with the exceedingly verbose nature of HTML, or XML, the benefits start becoming glaringly obvious.

Here's some Jade

```jade
ul#crocodile-items
  li(ng-repeat='foo in bar')
    div {{name}}
    div.details
      div
        span Description
        span {{desc}}
      div
        span Price
        span {{price}}
```

The HTML took me quite a bit longer to type by hand

```html
<ul id='crocodile-items'>
  <li ng-repeat='foo in bar'>
    <div>{{name}}</div>
    <div class='details'>
      <div>
        <span>Description</span>
        <span>{{desc}}</span>
      </div>
      <div>
        <span>Price</span>
        <span>{{price}}</span>
      </div>
    </div>
  </li>
</ul>
```

The pros to Jade start piling up when we take into account things like inheritance, mixins, JavaScript, and, well, the fact that you won't be forgetting to close tags anymore.

In the case of Coffee, though, it's as if it was the other way around. What the hell is happening in the JavaScript that gets compiled? I've no idea. To me, it's a black box. I don't like black boxes. They're black, and cube shaped. And that's about all I know about black boxes, **they don't tell me anything about what's going on under the covers**. Writing JavaScript code directly means we have full control over what we are doing, and that has precious value. One last thing. How are you going to incorporate ES6 when it comes around? To my knowledge, it'll break CoffeeScript as you know it. That kind of sucks. It means you'll either be stuck in a language that can't exploit all of the latest features in JavaScript, or you'll have to change tons of code in order to be back at square one. Me? I'd rather not be boxed into feature lockdown by the same language that's pretending to help me write better code.

My suggestion to you isn't even to stop using CoffeeScript, but to be more thoughtful of those who don't know anything about your pretty language. Always compile your code down to JavaScript before posting a question to StackOverflow, or replying to a blog post. You might even learn things _about JavaScript itself_, too.

  [1]: http://i.imgur.com/KLkLxBC.jpg "The dreaded CoffeeScript"
  [2]: http://img.youtube.com/vi/WINDtlPXmmE/hqdefault.jpg