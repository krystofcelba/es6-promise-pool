# Promise Pool [![Build Status](https://travis-ci.org/timdp/es6-promise-pool.svg?branch=master)](https://travis-ci.org/timdp/es6-promise-pool)

Runs `Promise`s in a pool that limits their maximum concurrency.

## Motivation

An ECMAScript 6 `Promise` is a great way of handling asynchronous operations.
The `Promise.all` function provides an easy interface to let a bunch of promises
settle concurrently.

However, it's an all-or-nothing approach: all your promises get created
simultaneously. If you have a ton of operations that you want to run with _some_
concurrency, `Promise.all` is no good.

Instead, you probably want to limit the maximum number of simultaneous
operations. That's where this module comes in. It provides an easy way of
waiting for any number of promises to settle, while imposing an upper bound on
the number of simultaneously executing promises.

The promises can be created in a just-in-time fashion. You essentially pass a
function that produces a new promise every time it is called. On modern
platforms, you can also use ES6 generator functions for this.

## Compatibility

This module can be used both under **Node.js** (version 0.10 and up) and on the
**Web**. If your platform does not have a `Promise` implementation, it will be
polyfilled by [ES6-Promise](https://github.com/jakearchibald/es6-promise).

## Installation

```bash
npm install --save es6-promise-pool
```

```html
<script src="es6-promise.js"></script>
<script>ES6Promise.polyfill();</script>
<script src="es6-promise-pool.js"></script>
```

## Usage

```js
// On the Web, leave out these two lines and use the script tags above instead.
var Promise = require('es6-promise').Promise;
var promisePool = require('es6-promise-pool');

var PromisePool = promisePool.PromisePool;

var promiseProducer = function() {
  // Your code goes here.
  // If there is work left to be done, return the next work item as a promise.
  // Otherwise, return null to indicate that all promises have been created.
  // Scroll down for an example.
};

// The number of promises to process simultaneously.
var concurrency = 3;

// Create a pool.
var pool = new PromisePool(promiseProducer, concurrency);

// Start the pool.
var poolPromise = pool.start();

// Wait for the pool to settle.
poolPromise.then(function() {
  console.log('All promises fulfilled');
}, function(error) {
  console.log('Some promise rejected: ' + error.message);
});
```

## Producers

The `PromisePool` constructor takes a `Promise`-producing function as its first
argument. Let's first assume that we have this helper function that returns a
promise for the given `value` after `time` milliseconds:

```js
var delayValue = function(value, time) {
  return new Promise(function(resolve, reject) {
    console.log('Resolving ' + value + ' in ' + time + ' ms');
    setTimeout(function() {
      console.log('Resolving: ' + value);
      resolve(value);
    }, time);
  });
};
```

### Function

Now, let's use the helper function above to create five such promises, which
are each fulfilled after a second. Because of the `concurrency` of `3`, the
first three promises will be fulfilled after one second. Then, the remaining two
will be processed and fulfilled after another second.

```js
var count = 0;
var promiseProducer = function() {
  if (count < 5) {
    count++;
    return delayValue(count, 1000);
  } else {
    return null;
  }
};

var pool = new PromisePool(promiseProducer, 3);

pool.start()
.then(function() {
  console.log('Complete');
});
```

### Generator

We can achieve the same result with ECMAScript 6 generator functions.

```js
var promiseProducer = function*() {
  for (var count = 1; count <= 5; count++) {
    yield delayValue(count, 1000);
  }
};

var pool = new PromisePool(promiseProducer, 3);

pool.start()
.then(function() {
  console.log('Complete');
});
```

## Events

We can also ask the promise pool to notify us when an individual promise is
fulfilled or rejected. The pool fires `fulfilled` and `rejected` events exactly
for this purpose.

```js
var pool = new PromisePool(promiseProducer, concurrency);

pool.addEventListener('fulfilled', function(event) {
  // The event contains:
  // - target:    the PromisePool itself;
  // - data:
  //   - promise: the Promise that got fulfilled;
  //   - result:  the result of that Promise.
  console.log('Fulfilled: ' + event.data.result);
});

pool.addEventListener('rejected', function(event) {
  // The event contains:
  // - target:    the PromisePool itself;
  // - data:
  //   - promise: the Promise that got rejected;
  //   - error:   the Error for the rejection.
  console.log('Rejected: ' + event.data.error.message);
});

pool.start()
.then(function() {
  console.log('Complete');
});
```

## Alternatives

- [Async.js](https://github.com/caolan/async)
- [Promise Pool](https://github.com/vilic/promise-pool)
- [qlimit](https://www.npmjs.com/package/qlimit)

## Author

[Tim De Pauw](https://tmdpw.eu/)

## License

Copyright &copy; 2015 Tim De Pauw

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
