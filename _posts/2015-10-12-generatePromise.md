---
layout: post
title: generatePromise( )
---
I was looking a bit deeper into Javascript's native Promises and ended up creating a simple helper function that generates a timed promise. I thought it might be useful for people who want to play around with promises-- just paste the code block into your code. Keep in mind 'console.error' doesn't work in [repl.it](https://repl.it)

There are four optional arguments:

* `resolveProbability` - Accepts values from 0-1. The probability the promise will resolve.
* `minTime` - Minimum time in milliseconds the promise will resolve.
* `maxTime` - Maximum time in milliseconds the promise will resolve.
* `mockData` - Custom data argument in case you want to generate a promise with data similar to what you will expect.

`generatePromise` will always resolve within 1000ms when no parameters are passed in.

### Code
```javascript
var generatePromise = function(resolveProbability, minTime, maxTime, mockData) {
  if (resolveProbability === undefined || resolveProbability === null) {
    resolveProbability = 1;
  }

  var body = mockData || Math.random().toString(2).substring(25);
  var id = Math.random().toString(36).substring(2);
  var randomRange =
  Math.floor(Math.random()*((maxTime||1000)-(minTime||1)+1))+(minTime||1);
  var time = Math.round(randomRange);

  return new Promise(function(resolve, reject) {
    if (Math.random() < resolveProbability) {
      setTimeout(function() {
        var toBeResolved = {
          data: mockData || {body: body},
          time: time + 'ms',
        }
        resolve(toBeResolved);
      }, time);

    } else {
      setTimeout(function() {
        reject('Promise failed to resolve. ' + time + 'ms');
      }, time);
    }
  });
};
```

### Example

#### For single promises
```javascript
generatePromise(.3)
.then(function(result) {
  console.log(result);
})
.catch(function(error) {
  console.error(error);
});

// -> Promise failed to resolve. 586ms
```

#### For Promise.all
```javascript
var promiseArray = [];
var myJSON = {hello: 'world'};

for(var i = 0; i < 10; i++) {
  promiseArray.push(generatePromise(1, 1000, 5000, myJSON));
}

Promise.all(promiseArray)
.then(function(promises) {
  console.log(promises);
})
.catch(function(error) {
  console.error(error);
});

/* 
  ->  [ { data: { hello: 'world' }, time: '4054ms' },
        { data: { hello: 'world' }, time: '2894ms' },
        { data: { hello: 'world' }, time: '2859ms' },
        { data: { hello: 'world' }, time: '2594ms' },
        { data: { hello: 'world' }, time: '4797ms' },
        { data: { hello: 'world' }, time: '1181ms' },
        { data: { hello: 'world' }, time: '1061ms' },
        { data: { hello: 'world' }, time: '3013ms' },
        { data: { hello: 'world' }, time: '3158ms' },
        { data: { hello: 'world' }, time: '1651ms' } ]
*/
```
