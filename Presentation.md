---
presentation:
  width: 1024
  height: 800
  theme: solarized.css
  enableSpeakerNotes: true
---

<!-- slide -->

## JavaScript Promise

### Thor Chen
11th Oct 2019


<!-- slide -->

## Agenda

TODO:

<!-- slide -->

## Background: Synchronous

Code runs line by line (blocking)

```javascript
console.log('A');
console.log('B');
console.log('C');
```

<!-- slide -->

## Background: Asynchronous

Some code runs "in parallel" and we keep going with next lines without waiting for the result


```javascript
console.log('A');
setTimeout(() => {
    console.log('B');
}, 0);
console.log('C');
```

<!-- slide -->

## Background: Callback

Callback was the solution for using asynchronous code

```js
function successCallback(result) {...}

function failureCallback(error) {...}

doSomethingAsync(params, successCallback, failureCallback);
```

* Example with XHR: https://jsfiddle.net/zzdjk6/kn9t46c8/3/

<!-- slide data-notes="
* Every step needs waiting
* Every step can fail
" -->

## The problem: callback hell

### Think about ordering a pizza

```javascript
chooseToppings(function(toppings) {
    placeOrder(toppings, function(order) {
        collectOrder(order, function(pizza) {
            eatPizza(pizza);
        }, failureCallback);
    }, failureCallback);
}, failureCallback);
```

* What if you have more steps or more logic (e.g, if/else branches) in each callback?
    * It will be extremly hard to understand and debug

<!-- slide vertical=true -->

## Promise to rescure

```js
chooseToppings()
    .then(toppings => placeOrder(toppings))
    .then(order => collectOrder(order))
    .then(pizza => eatPizza(pizza))
    .catch(failureCallback);
```

* More readable
  * No "Pyramid"
  * Only one failureCallback is needed
  * Chained code for easier reviewing executing order

<!-- slide data-notes="
* When a promise is created, it is neither in a success or failure state. It is said to be `pending`.
* When a promise returns, it is said to be `resolved` or `settled`.
    * A successfully resolved promise is said to be `fulfilled`. It returns a `value`, which can be accessed by chaining a `.then()` block onto the end of the promise chain. The executor function inside the `.then()` block will contain the promise's return value.
    * An unsuccessful resolved promise is said to be `rejected`. It returns a `reason` - an error message stating why the promise was rejected. This reason can be accessed by chaining a `.catch()` block onto the end of the promise chain.
" -->

## Defination & Explaination

A Promise is an object representing the eventual completion or failure of an asynchronous operation [[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)]

<img src="/assets/promise-life-cycle.png" alt="" style="width: 100%" />

<!-- slide -->

## Use of promise

* In our daily development, promises are most likely returned from a built-in method call (such as `fetch`) or 3rd party lib

```js
fetch('http://example.com/movies.json')
    .then(response => response.json())
    .then(data => console.log(data));
```

<!-- slide -->

## Create Promise

* Usually the constructor of `Promise` is used to wrap a callback-style function

```js
const wait = (resolveTime, rejectTime) => {
    return new Promise((resolve, reject) => {
    	setTimeout(resolve, resolveTime * 1000, `resolve in ${resolveTime} seconds`);
        setTimeout(reject, rejectTime * 1000, `timeout in ${rejectTime} seconds`);
    });
}

console.log('A');
wait(2, 3)
    .then(result => console.log(result))
    .catch(error => console.error(error));
console.log('B');
```

<!-- slide vertical=true -->

* There are shorthand functions to create immediately resolved promise:
  * `Promise.resolve()`
  * `Promise.reject()`
* Complex usage: bridging between native mobile code and JavaScript:
  * https://medium.com/@zzdjk6/wkwebview-cors-solution-da20ca1194e8

<!-- slide -->

## Compose Promise

```js
// Chain promise
const wait = seconds => new Promise(resolve => setTimeout(resolve, seconds * 1000, seconds));

const log = data => console.log(`${parseInt((new Date).getTime()/1000)}`, data);

const createFunc = seconds => () => wait(seconds).then(log);

const func1 = createFunc(1);
const func2 = createFunc(2);
const func3 = createFunc(3);

log('start');

func1()
  .then(func2)
  .then(func3)
  .catch(console.error)
  .then(() => log('finish'));
```

<!-- slide vertical=true -->

```js
// Run promise in parallel
const wait = seconds => new Promise(resolve => setTimeout(resolve, seconds * 1000, seconds));

const log = data => console.log(`${parseInt((new Date).getTime()/1000)}`, data);

const createFunc = seconds => () => wait(seconds).then(data => {
  log(data);
  return data;
});

const func1 = createFunc(1);
const func2 = createFunc(2);
const func3 = createFunc(3);

log('start');
Promise
  .all([func1(), func2(), func3()])
  .then(results => console.log(results))
  .catch(console.error)
  .then(() => log('finish'));
```

* `Promise.all` will execute all asynchronous functions in parallel and collect their results
* `Promise.all` will be rejected if any of the promise is rejected


<!-- slide vertical=true -->

* `Promise.race` to execute all promises in parallel but will end up with the first resolved one

```js
Promise
  .race([func1(), func2(), func3()])
  .then(result => log(result))
  .catch(console.error)
  .then(() => log('finish'));
```

<!-- slide vertical=true -->

* Promise only supports very basic composition, if you need more complex data flow, consider using `rxjs`
* Promise does not support `cancel`


<!-- slide -->

## Common mistakes

```js
// TODO: use step1, step2, ...
doFirstThing()
    .then(result1 => {
        doSecondThing(result1)
            .then(result2 => doThirdThing(result2));
    })
    .then(() => doFourthThing());
```

* Forgot to return promise from inner chain
    * `doFourthThing` will not wait for `doSecondThing` or `doThirdThing`
* Unnecessary nesting
    * Less readable again
* Forgot to terminate chain with a catch!

<!-- slide vertical=true -->

## Make it correct

```js
doFirstThing()
    .then(result1 => doSecondThing(result1))
    .then(result2 => doThirdThing(result2))
    .then(() => doFourthThing())
    .catch(error => console.error(error));
```

<!-- slide -->

## Better syntax: Async/Await

```js
// Using promise
const makeRequest = () =>
    getJSON()
        .then(data => {
            console.log(data)
            return "done"
        })
        .catch(error => console.error(error));
```

```js
// Using async/await
const makeRequest = async () => {
    try {
        const data = await getJSON();
        console.log(data)
        return "done"
    } catch (error) {
        console.error(error);
    } finally {
        ...
    }
}
```

<!-- slide vertical=true -->

## Why it is better?

* 6 Reasons Why JavaScript Async/Await Blows Promises Away (Tutorial)
    * Link: https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9
    * The "better debugging support" point looks very interesting but I haven't tried yet (TODO: Try it)

<!-- slide vertical=true -->

### Avoid nesting with condition logic

```js
const makeRequest = () => {
  return getJSON()
    .then(data => {
      if (data.needsAnotherRequest) {
        return makeAnotherRequest(data)
          .then(moreData => {
            console.log(moreData)
            return moreData
          })
      } else {
        console.log(data)
        return data
      }
    })
}
```

```js
const makeRequest = async () => {
  const data = await getJSON()
  if (data.needsAnotherRequest) {
    const moreData = await makeAnotherRequest(data);
    console.log(moreData)
    return moreData
  } else {
    console.log(data)
    return data    
  }
}
```

<!-- slide vertical=true -->

### Avoid nesting with intermediate values

```js
const makeRequest = () => {
  return promise1()
    .then(value1 => {
      // do something
      return promise2(value1)
        .then(value2 => {
          // do something          
          return promise3(value1, value2)
        })
    })
}
```

```js
const makeRequest = async () => {
  const value1 = await promise1()
  const value2 = await promise2(value1)
  return promise3(value1, value2)
}
```

<!-- slide -->

## Browser compatibility

* Promise is not supported natively in IE
  * We have polyfill
* Async/Await is introduced in ECMAScript 2017
    * We have transpilers 😊
    * Babel / TypeScript alows us to use the latest features of JavaScript without fear!

<!-- slide -->

## Recap

* TODO:
* `Promise` is invented to solve "callback hell"