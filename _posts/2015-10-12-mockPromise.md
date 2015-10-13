---
layout: post
title: mockPromise( )
---
I was looking a bit deeper into Javascript's native Promises and ended up creating a simple helper function that generates a timed mock promise. I thought it might be useful for people who want to play around with promises-- just paste the code block into your code. Keep in mind `console.error` doesn't work in [repl.it](https://repl.it)

There are four **optional** arguments:

* `resolveProbability` - [0 - 1] The probability the promise will resolve. 0 rejects all, 1 resolves all.
* `minTime` / `maxTime` - Minimum/Maximum timeout in milliseconds the promise will resolve or reject.
* `mockData` - Custom data argument in case you want to generate a promise with data similar to what you expect.

`mockPromise` will always resolve within 1000ms when no arguments are passed in.

### Code
```javascript
var mockPromise = function(resolveProbability, minTime, maxTime, mockData) {
  if (resolveProbability === undefined || resolveProbability === null) {
    resolveProbability = 1;
  }
  
  // Creating mock data in case none is passed in
  var data = mockData || 'Resolved Value';
  
  // Picks a random time between minTime and maxTime
  var timeOut = Math.floor(Math.random()*((maxTime||1000)-(minTime||1)+1))+(minTime||1);

  // Creates a new promise. If the resolve probability is higher than
  // Math.random(), the promise will resolve. If not, it will be rejected.
  return new Promise(function(resolve, reject) {
    if (Math.random() < resolveProbability) {
      setTimeout(function() {
        var toBeResolved = { data: data, timeOut: timeOut + 'ms' };
        resolve(toBeResolved);
      }, timeOut);

    } else {
      setTimeout(function() {
        reject('Promise failed to resolve. ' + timeOut + 'ms');
      }, timeOut);
    }
  });
};
```

#### ES6
```javascript
const mockPromise = (resolveProbability = 1, minTime = 1, maxTime = 1000, mockData = 'Resolved Value') => {
  const timeOut = Math.floor(Math.random() * (maxTime - minTime + 1)) + minTime;

  return new Promise((resolve, reject) => {
    if (Math.random() < resolveProbability) {
      setTimeout(() => {
        const toBeResolved = { data: mockData, timeOut: timeOut + 'ms' };
        resolve(toBeResolved);
      }, timeOut);

    } else {
      setTimeout(() => {
        reject('Promise failed to resolve. ' + timeOut + 'ms');
      }, timeOut);
    }
  });
};
```

### Example

#### Single promise
```javascript
mockPromise(.3) // Notice this call will only generate mock promises that resolve 30% of the time
.then(function(result) {
  console.log(result);
})
.catch(function(error) {
  console.error(error);
});

/*
Usually returns something like this -> Promise failed to resolve. 586ms
Once in a while returns something like this -> { data: { body: 'Resolved Value' }, time: '644ms' }
*/
```

#### Promise.all
```javascript
var promiseArray = [];
var myJSON = {hello: 'world'};

for(var i = 0; i < 10; i++) {
  promiseArray.push(mockPromise(1, 1000, 5000, myJSON));
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
