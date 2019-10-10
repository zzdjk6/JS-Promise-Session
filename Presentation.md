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

- Background
- Use `Promise`
- `Async` / `Await`

<!-- slide -->

## Background: Synchronous

Code runs line by line (blocking)

```javascript
console.log("A");
console.log("B");
console.log("C");
```

<!-- slide -->

## Background: Asynchronous

Some code runs "in parallel" and we keep going with next lines without waiting for the result

```javascript
console.log("A");
setTimeout(() => {
  console.log("B");
}, 0);
console.log("C");
```

<!-- slide -->

## Background: Callback

Callback was the solution for using asynchronous code

```js
function successCallback(result) {...}

function failureCallback(error) {...}

doSomethingAsync(params, successCallback, failureCallback);
```

- [Example with XHR](https://jsbin.com/hoceluv/2/edit?js,console)

<!-- slide data-notes="
* Every step needs waiting
* Every step can fail
* It will be extremly hard to understand and debug
" -->

## The problem: callback hell

```javascript
chooseToppings(toppings => {
  placeOrder(toppings, order => {
      collectOrder(order, pizza => {
          eatPizza(pizza);
        }, failureCallback);
    }, failureCallback);
}, failureCallback);
```

<!-- slide vertical=true -->
![callback-hell](/assets/callback-hell.png)

- What if you have more steps or more logic (e.g, if/else branches) in each callback?

<!-- slide vertical=true -->

## Promise to rescure

```js
chooseToppings()
  .then(toppings => placeOrder(toppings))
  .then(order => collectOrder(order))
  .then(pizza => eatPizza(pizza))
  .catch(failureCallback);
```

- More readable
  - No "Pyramid"
  - Only one failureCallback is needed
  - Chained code for easier reviewing executing order

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

- In our daily development, promises are most likely returned from a built-in method call (such as `fetch`) or 3rd party lib

```js
fetch("http://example.com/movies.json")
  .then(response => {
    const data = response.json();
    return data;
  })
  .then(data => {
    console.log(data)
  })
  .catch(console.error);
```

<!-- slide vertical=true -->

## `.then()`

```js
// p is a Promise
p.then(onFulfilled[, onRejected]);

p.then(value => {
  // fulfillment
}, reason => {
  // rejection
});
```

- we pass `onFulfilled` function which takes one argument to `.then()`
- `onRejected` is rarely used explicitly, we usually use `.catch()` for readability (explain it later)

<!-- slide vertical=true -->

## Promise chain

```js
// p is a Promise
p.then(result => {
    const newResult = func1(result);
    return newResult;
})
 .then(result => {
    const newResult = func2(result);
    return newResult;
})
```

- The `newResult` can be a `value` or `Promise`
- If you want to nest promise (do something asynchronously) inside `.then()`, make sure to return the `Promise`, otherwise the next `.then()` will not wait for its execution

<!-- slide vertical=true -->

## Promise chain (continue)

What if you forget to return nested `Promise`?

```js
const wait = seconds =>
  new Promise(resolve => setTimeout(resolve, seconds * 1000, seconds));

Promise.resolve()
    .then(() => console.log('start'))
    .then(() => {
        wait(2).then(() => console.log('inside'));
    })
    .then(() => console.log('finish'));
```
- Read more on [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)

<!-- slide vertical=true -->

## Javascript tips

These statements are equal

```js
p.then(result => {
    const newResult = func(result);
    return newResult;
});

p.then(result => func(result));

p.then(func);
```

<!-- slide vertical=true -->

## `.catch`

```js
Promise.reject('I am an error')
  .catch(console.error)
  .then(() => console.log('I am fine'));
```

- Tip: after `.catch()`, the `Promise` becomes a `fullfiled` one, so you can call `.then()` afterwards

<!-- slide vertical=true -->

## `.finally`

```js
Promise.reject('Something bad')
  .then(() => console.log('I am good'))
  .catch(console.error) // try comment this
  .finally(() => console.log('I am done'));
```

- When the promise is `settled`, i.e either `fulfilled` or `rejected`, the specified callback function is executed
- This helps to avoid duplicating code in both the promise's `then()` and `catch()` handlers.
- Tip: `return` in `.finally()` callback has no effects, but `throw` does

<!-- slide -->

## Create Promise

- Usually the constructor of `Promise` is used to wrap a callback-style function

```js
const wait = (resolveTime, rejectTime) => {
  return new Promise((resolve, reject) => {
    setTimeout(
      resolve,
      resolveTime * 1000,
      `resolve in ${resolveTime} seconds`
    );
    setTimeout(reject, rejectTime * 1000, `timeout in ${rejectTime} seconds`);
  });
};

console.log("A");
wait(2, 3) // try (3, 2)
  .then(result => console.log(result))
  .catch(error => console.error(error));
console.log("B");
```

<!-- slide vertical=true -->

- There are shorthand functions to create immediately resolved promise:
  - `Promise.resolve()`
  - `Promise.reject()`
- Complex usage: two-way bridging between native mobile code and JavaScript:
  - https://medium.com/@zzdjk6/wkwebview-cors-solution-da20ca1194e8

<!-- slide -->

## Combine Promises

```js
// Run promises in series
const wait = seconds =>
  new Promise(resolve => setTimeout(resolve, seconds * 1000, seconds));

const log = data =>
  console.log(`${parseInt(new Date().getTime() / 1000)}`, data);

const createFunc = seconds => () => wait(seconds).then(log);

const func1 = createFunc(1);
const func2 = createFunc(2);
const func3 = createFunc(3);

log("start");

func1()
  .then(func2)
  .then(func3)
  .catch(console.error)
  .then(() => log("finish"));
```

<!-- slide vertical=true -->

```js
// Run promises in parallel
Promise.all([func1(), func2(), func3()])
  .then(results => console.log(results))
  .catch(console.error)
  .then(() => log("finish"));
```
```js
const log = data => {
  console.log(`${parseInt(new Date().getTime() / 1000)}`, data);
  return data;
};
const funcE = wait(2).then(() => Promise.reject('error'));
```

- `Promise.all` creates a new `Promise`
- it will execute all asynchronous functions in parallel and collect their results
- it will be rejected if any of the promise is rejected
- Bonus: try `Promise.allSettled()`

<!-- slide vertical=true -->

- `Promise.race` creates a new `Promise`
- It executes all promises in parallel but will end up with the first resolved one
- Notice: other promises are not terminated

```js
Promise.race([func1(), func2(), func3()])
  .then(result => log(result))
  .catch(console.error)
  .then(() => log("finish"));
```

<!-- slide vertical=true -->

## Personal suggest

- Promise only supports very basic composition
- if you need more complex data flow, consider using `rxjs` or other promise libraris (`q`, `bluebird`, etc)

<!-- slide -->

## Common mistakes

```js
step1()
  .then(result1 => {
    step2(result1)
      .then(result2 => step3(result2));
  })
  .then(() => step4());
```

- Forgot to return promise from inner chain
  - `step4` will not wait for `step2` or `step3`
- Unnecessary nesting
  - Less readable again
- Forgot to terminate chain with a catch!

<!-- slide vertical=true -->

## Make it correct

```js
step1()
  .then(result1 => step2(result1))
  .then(result2 => step3(result2))
  .then(() => step4())
  .catch(error => console.error(error));
```

Or even shorter

```js
step1()
  .then(step2)
  .then(step3)
  .then(step4)
  .catch(console.error);
```

<!-- slide -->

## Better syntax: Async/Await

```js
// Using promise syntax
const makeRequest = () =>
  getJSON()
    .then(data => {
      console.log(data);
      return data;
    })
    .catch(error => console.error(error))
    .finally(() => {
      console.log("done");
    });
```

```js
// Using async/await syntax
const makeRequest = async () => {
  try {
    const data = await getJSON();
    console.log(data)
    return data;
  } catch (error) {
    console.error(error);
  } finally {
    console.log("done");
  }
}
```

<!-- slide vertical=true -->

## Why it is better?

- [6 Reasons Why JavaScript Async/Await Blows Promises Away](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9)
- [From JavaScript Promises to Async/Await: why bother?](https://blog.pusher.com/promises-async-await/)
- [JavaScript’s Async/Await versus Promises: The Great Debate](https://itnext.io/javascripts-async-await-versus-promise-the-great-debate-6308cb2e10b3)
- etc.,

<!-- slide vertical=true -->

### Avoid nesting with condition logic

```js
const makeRequest = () => {
  return getJSON()
    .then(data => {
      if (data.needsAnotherRequest) {
        return makeAnotherRequest(data)
          .then(moreData => {
            console.log(moreData);
            return moreData;
          });
      } else {
        console.log(data);
        return data;
      }
    });
};
```

```js
const makeRequest = async () => {
  const data = await getJSON();
  if (data.needsAnotherRequest) {
    const moreData = await makeAnotherRequest(data);
    console.log(moreData);
    return moreData;
  } else {
    console.log(data);
    return data;
  }
};
```

<!-- slide vertical=true -->

### Avoid nesting with intermediate values

```js
const makeRequest = () => {
  return step1().then(value1 => {
    // do something
    return step2(value1).then(value2 => {
      // do something
      return step3(value1, value2);
    });
  });
};
```

```js
const makeRequest = async () => {
  const value1 = await step1();
  const value2 = await step2(value1);
  return step3(value1, value2);
};
```

<!-- slide -->

## Browser compatibility

### Problems

- `Promise` is not supported natively in IE
- `Async` / `Await` is introduced in ECMAScript 2017
- Some methods are in future ES version (e.g., `.finally()`, `.allSettled()`)

<!-- slide vertical=true -->

### Solution: transpilers + polyfill 😊
- Transpilers: 
  - Babel
  - TypeScript
- Polyfill:
  - @babel/polyfill
  - core-js
  - es6-promise
  - promise-polyfill
  - etc.,

They alows us to use the latest features of JavaScript without fear!

<!-- slide -->

## Recap

- `Callback` was the solution for asynchronous code
- `Promise` is invented to solve "callback hell"
- `Async` / `Await` make `Promise` great again :)
