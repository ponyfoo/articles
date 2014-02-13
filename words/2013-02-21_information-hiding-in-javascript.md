# Information Hiding in JavaScript #

Even though it's tricky at first, if you are used to _classical_ object-oriented languages, it's easy (and _highly encouraged_) to perform **information hiding** in JavaScript. There are more than a few methods by which you can hide information within _privileged scopes_. **Closures, properties (getters and setters), and factories**, are all ways in which you can improve your designs by **hiding away the complexity** behind the _public interfaces_ your code exposes.

Information hiding results in _cleaner, conciser code, that's easier to read_, and therefore, **more maintainable** (and easily _testable_).

# Closures #

JavaScript, _being the stupendous functional language that it is_, closures let us write clean code without cluttering up [the dreaded global object](https://developer.mozilla.org/en-US/docs/DOM/window "The Global Object in client-side JavaScript") with properties and functions that have no business being there. A better approach is to _self-contain_ the knowledge required by a particular functionality. This can be accomplished by wrapping your code in a function:

```js
function(){
	// your code goes here
}
```

That's a closure right there, but this closure is just a function, you want it to self-execute. You can accomplish this by using any of the following operators: `-`, `+`, `!`.

```js
!function(){
	// your code goes here
}();
```
	
I like prepending `!` to my self-executing functions, some prefer _wrapping it in parenthesis_ instead. It's just a matter of personal preference, but try and stick to a convention. The most common pattern, though, is wrapping the function in parenthesis. Like this:

```js
(function(){
	// your code goes here
})();
```

Note that none of this will be exposed to the global namespace _unless you explicitly do so_, the pattern for that is usually to add a `window` argument to your closure (or an argument with the value of your library namespace), and expose your properties there.

```js
!function(window){
	window.myObject = {
		foo: 'bar'
	};
}(window);
```
	
Now that you have a closure, anything you define within your closure will be **private** to that closure's scope. Due to the cascading nature of closures, every closure has access to a **superset** of it's outer closure's scope, meaning it has access to the entire outer closure, and to whatever gets defined within itself.

```js
!function(window){
	var builder = 'Built with amazing closure awesomeness',
		myCar;
			
	function decorate(car){
		car.builder = builder;
		return car;
	}
	
	!function(){
		// well, not that much of a secret
		var secret = 'They were pioneers!';
		
		myCar = decorate({
			make: 'Ford',
			model: 'T',
			getSecret: function(){
				return secret;
			}
		});
	}();
	
	window.car = myCar;
}(window);
```

Since the inner scope of `getSecret` is a superset of the outer scope, it can access `secret`.

There are **two exceptions** to the cascading rule of scoping, and those are `this` and `arguments`. Those two variables are re-defined at every scope. A common work-around is to keep a reference around for future use:

```js
String.prototype.isInArray = function(array){
	var self = this.toString();

	return array.some(function(value){
		return value === self;
	});
}

var objects = ['a', 'b', 'c', 'd', 'e'];
console.log('a'.isInArray(objects));
console.log('f'.isInArray(objects));
```

# What do you mean, properties? #

Bloggers seldom mention JavaScript properties **(getters and setters)**, to the point where they've barely seen the light of day, and have not even remotely been adopted in widespread use.

> Properties are subtle yet powerful tools, in particular when trying to do [AOP](http://en.wikipedia.org/wiki/Aspect-oriented_programming "Aspect Oriented Programming"). Sometimes, it makes more sense to use a property rather than a field, or a function. The beauty of properties is that you can have behavior in your fields. Additionally, some libraries _require_ you to use fields instead of functions, this helps you circumvent that kind of requirements.

Allow me to demonstrate with a mini-object:

```js
!function(window){
	var workers = [],
		factor = 1;
	
	window.timecard = {
		checkin: function(username){
			if (workers.indexOf(username) === -1){
				console.log(username + ' checked in!');
				workers.push(username);
			}
		},
		checkout: function(username){
			var i = workers.indexOf(username);
			if (i !== -1){
				console.log(username + ' checked out!');
				workers.splice(i, 1);
			}
		},
		get workers() {
			return workers.join(' and ');
		},
		get productivity() {
			return workers.length * factor;
		},
		get productivityFactor(){
			return factor;
		},
		set productivityFactor(value){
			console.log('Productivity ' + (value > factor ? 'increased!' : 'decreased.'));
			factor = value;
		}
	};
}(window);

function p(test){ // print
	console.log(test);
}

timecard.checkin('Joe');
timecard.checkin('Steven');
p(timecard.workers === 'Joe and Steven'); // true
p(timecard.productivity); // 2
|timecard.productivity = 3; // productivity is read-only
p(timecard.productivity === 2); // thus, setting it is a no-op
timecard.productivityFactor = 4;
timecard.checkout('Joe');
p(timecard.productivity); // 4. The getter function is evaluated every time.
```

Note that IE versions up to **IE 8** _don't support `get` and `set` operators_. To work around that, you'll have to use the `__defineGetter__` and `__defineSetter__` methods that can be found in `Object.prototype`. These are a bit more flexible because it's impossible to add a getter or setter to an existing object without using one of them.

```js
var o = {
	name: 'Steven',
	surname: 'Sundae'
};

o.__defineGetter__('displayName', function(){
	return this.name + ' ' + this.surname;
});
```

# Functional Factories #

Closely related to [currying](http://en.wikipedia.org/wiki/Currying "Currying"), are the _functional factories_. These factories are very powerful to the JavaScript hacker. They will allow you to reuse very similar pieces of functionality in cases where it would be otherwise very hard to do so.

Suppose you are writing your [Node.JS](http://nodejs.org/ "Node.JS") application, and you have a series of _router methods_, all of which take `req, res` as parameters. All that changes in each of those, is the function you call in your application logic, but there's a few more tasks you have to do, such as _sending a response_, that are common to every request. Lets assume your module exports an object hash keyed with _routes_, whose values are the _middleware_ for each of those routes.

```js
var list = [1,2,3,4];

function respond(res,value){
	var json = JSON.stringify(value);
	
	res.writeHead(200, { 'Content-Type': 'application/json' });
	res.end(json);
}

module.exports = {
	list: function(req,res){
		respond(res,list);
	},
	add: function(req,res){
		list.push(req.body.item);
		respond(res,list);
	},
	remove: function(req,res){
		list.splice(req.body.index, 1);
		respond(res,list);
	}
};
```

Generally speaking, when you see something along the lines of:

```js
function(arg1,arg2,arg3,arg4){
	do(arg1,arg2,arg3,arg4);
}
```

That code can be refactored to be a bit more [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself "DRY principle").

```js
function handle(before){
	var noop = function(){};
	
	return function(req, res){
		(before || noop)(req, res);
		respond(res, list);
	};
}

module.exports = {
	list: handle(),
	add: handle(function(req){
		list.push(req.body.item);
	}),
	remove: handle(function(req){
		list.splice(req.body.index, 1);
	})
};
```

Granted, this style is a tad more _verbose_, especially if **unwarranted**, but if your `handle` function did _more stuff_, such as _request validation_, _logging_, or trigger web socket events, this would be a nice way to abstract those away from the specifics of each of those methods, that little have to do with validation itself.

Using such factories also allows to abstract away complexity, which is always a very valuable thing to do when designing applications.
