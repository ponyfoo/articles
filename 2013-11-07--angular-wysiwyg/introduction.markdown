`ponyedit` allows us to interact with a `contentEditable` element by following the **Obey and Report** pattern. It emits events whenever its state changes, and it takes commands that alter this state. This enables us to completely decouple the user interface from the component's functionality. In this article, we'll dig a little deeper into the pattern, analyzing the decisions made in ponyedit, how it came together, its resulting API, and the [sample _bare bones_ UI implementation][1].

[1]: https://github.com/bevacqua/ponyedit/blob/master/web/assets/js/example.js
