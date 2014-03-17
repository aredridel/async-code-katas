# Asynchronous Javascript Code Katas

## Part 1: Simple values, sequential flow, no errors

Write a synchronous get function and set function. Get a value, set a value, get again.

```
function makeThing() {
	var val = 0;
	return {
		get: function () {
			return val;
		},
		set: function (v) {
			val = v;
		}
	}
}

var thing = makeThing();
var v = thing.get();
thing.set(v + 1);
console.log(thing.get(v));
```

### Going asynchronous

Transform that into asynchronous, continuation-passing style. Instead of a function returning a value back to its caller, the program continues by passing a value forward to the remaining code: calling a continuation function with a value.

```
function makeThing(cb) {
	var val = 0;
	cb({
		get: function (cb) {
			cb(val);
		},
		set: function (v, cb) {
			val = v;
			cb();
		}
	});
}

makeThing(function (thing) {
	thing.get(function(v) {
		thing.set(v + 1, function () {
			thing.get(function (v) {
				console.log(v);
			});
		});
	});
});
```
Notice that things are very isolated. The actual process happening is invisible, but the flow is shown as lexical scopes.

### Break things apart

Unnest and name the callbacks to escape nesting hell.

```
function makeThing(cb) {
	var val = 0;
	cb({
		get: function (cb) {
			cb(val);
		},
		set: function (v, cb) {
			val = v;
			cb();
		}
	});
}

var thing;

makeThing(handleThing);

function handleThing(aThing) {
	thing = aThing; // Export into the scope of the greater task
	thing.get(updateValue);
}

function updateValue(v) {
	thing.set(v + 1, verifySet);
}
	
function verifySet() {
	thing.get(logResult);
}

function logResult(v) {
	console.log(v);
}
```

Notice that in this form, our shared state `thing` is extra visible because it's the only variable declared. All local variables are parameters to functions.

At this point, the flow events have been named. It's easy to see what the program does, but not in what order.

### Refactor to use promises.

```
var Promisable = require('promisable');

function makeThing() {
	var val = 0;
	return Promisable.resolve({
		get: function () {
			return Promisable.resolve(val);
		},
		set: function (v, cb) {
			val = v;
			return Promisable.resolve();
		}
	});
}

var thing;

makeThing().then(function handleThing(aThing) {
	thing = aThing;
	return thing.get(resolve)
}).then(function updateValue(v) {
	return thing.set(v + 1);
}).then(function verifySet() {
	return thing.get(v);
}).then(function logResult(value) {
	console.log(value);
});
```

Now our shared state `thing` is visible. Flow is top-to-bottom, and I've left the names of the functions in place  as commentary on what they do.

This doesn't get to the style where you treat a promise as data that just isn't ready yet, and instead stays focused on flow control.