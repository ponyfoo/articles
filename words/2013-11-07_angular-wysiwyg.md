# Angular WYSIWYG

Building on the blocks laid out [in my previous article](/2013/10/25/event-emitter-obey-and-report "Event Emitter: Obey and Report"), I open-sourced a [WYSIWYG](http://en.wikipedia.org/wiki/WYSIWYG "What you see is what you get - Wikipedia") editing library _which doesn't provide an UI_. You can [find the source code here](https://github.com/bevacqua/ponyedit "bevacqua/ponyedit on GitHub").

`ponyedit` allows us to interact with a `contentEditable` element by following the **Obey and Report** pattern. It emits events whenever its state changes, and it takes commands that alter this state. This enables us to completely decouple the user interface from the component's functionality. In this article, we'll dig a little deeper into the pattern, analyzing the decisions made in ponyedit, how it came together, its resulting API, and the [sample _bare bones_ UI implementation](https://github.com/bevacqua/ponyedit/blob/master/web/assets/js/example.js "Sample WYSIWYG using ponyedit").

The sample UI, bundled in the module's repository, looks like this:

![ui.png][1]

### Implemented in Angular

The [Angular implementation example](http://ponyedit.herokuapp.com/angular "Ponyedit using Angular") for `ponyedit` uses a directive and a controller. However we could organize the application however we want it, as the library doesn't set forth any constraints. The directive will simply instance the object on a DOM element. The controller will display the object state, and allow us to send commands back to the object. Please note this is just the way I hacked it together in under 5 minutes, definitely not the best way to go about it. You might want everything in a single directive. In the particular use case we had, we needed to display state and commands in a panel that was controlled by a directive, while the editor was within another directive, so `ponyedit` was born.

Here's the sample directive.

```js
app.directive('ponyeditable', [
    '$rootScope',
    function ($rootScope) {
        return {
            restrict: 'A',
            link: function (scope, element, attrs) {
                var dom = element[0];
                ponyedit.init(dom);
                $rootScope.$broadcast('pony', dom);
            }
        };
    }
]);
```

This directive is loaded with `article.editable(ponyeditable)` in the [example Jade file](https://github.com/bevacqua/ponyedit/blob/master/web/views/angular.jade "Sample Jade file").

> As you can see, that just initializes the editor, and then broadcasts so our controller can do its thing. Clearly this doesn't scale up very well at all (if you have multiple instances of this directive), but I wanted to keep things simple for the demo.

The controller could definitely be simpler, but I didn't want to include the entire ponyeditor in the scope, to get a more decoupled result. Its code is below:

```js
app.controller('exampleCtrl', [
    '$scope',
    function ($scope) {
        $scope.$on('pony', function (e, element) {
            var pony = ponyedit(element);

            $scope.state = {};
            $scope.setBold = pony.setBold.bind(pony);
            $scope.setItalic = pony.setItalic.bind(pony);
            $scope.decreaseSize = pony.decreaseSize.bind(pony);
            $scope.increaseSize = pony.increaseSize.bind(pony);

            pony.on('report.*', function (value, property) {
                $scope.state[property] = value;

                if (!$scope.$$phase) {
                    $scope.$apply();
                }
            });
        });
    }
]);
```

The controller acts as a proxy, just passing commands through to the ponyeditor, and passing state reports through to the `$scope`. This keeps the controller thin. The view has some bindings to display the state, and wire up the commands.

```jade
div.ui(ng-controller='exampleCtrl', ng-disabled='!state.active')
    span.ui-state.ui-bold(ng-click='setBold()', ng-class='{true:"ui-enabled"}[state.bold]') Bold
    span.ui-state.ui-italic(ng-click='setItalic()', ng-class='{true:"ui-enabled"}[state.italic]') Italic
    span.ui-state.ui-size-up(ng-click='increaseSize()') +
    span.ui-state.ui-size-down(ng-click='decreaseSize()') -
```

That's all I had to do in order to use `ponyedit` with Angular from the ground up, giving it a UI that _just works_.

### Get the Source!

All the library really does is sort out a few inconsistencies in `contentEditable` elements, and provide a nice API around it, making it almost feel easy to deal with it. The code is [fairly easy to follow](https://github.com/bevacqua/ponyedit/blob/master/src/ponyedit.js "ponyedit.js on GitHub"), and I [thoroughly documented its public API](https://github.com/bevacqua/ponyedit#api "Ponyedit API documentation on GitHub"), so that's definitely worth a look if you're interested.

  [1]: http://i.imgur.com/NYNlIWg.png "A sample UI implementation for ponyedit"