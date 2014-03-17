# Asynchronous Javascript Code Katas

## Part 1: Simple values, sequential flow

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
v = v + 1;
thing.set(v);
console.log(thing.get(v));
```

Transform that into asynchronous, continuation-passing style.

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
		v = v + 1;
		thing.set(v, function () {
			thing.get(function (v) {
				console.log(v);
			});
		});
	});
});
```

Unnest and name the callbacks

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
	thing.get(doFirstAccess);
}

function doFirstAccess(v) {
	v = v + 1;
	thing.set(v, verifySet);
}
	
function verifySet() {
	thing.get(logResult);
}

function logResult(v) {
	console.log(v);
}
```

Refactor to use promises

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

makeThing().then(function (aThing) {
	thing = aThing;
	return thing.get(resolve)
}).then(function (v) {
	return thing.set(v + 1);
}).then(function () {
	return thing.get(v);
}).then(function (value) {
	console.log(value);
});
```