# 05: Setting a Time Limit for Async Tasks

## Comparing `Promise.race()` to `Promise.any()`

`Promise.any()` uses the value of the first promise that fulfills. `Promise.race()` behaves exactly the same as far as promise fulfillment. However, when it comes to rejection, `Promise.race()` is completely different: it settles as soon as one of the given promises rejects.

While `Promise.any()` rejects if all the given promises reject, `Promise.race()`takes an array of Promises and returns another Promise. This Promise is fulfilled or rejected when the first of the Promises in the input array is fulfilled or rejected (or the first of non-Promise values, as they count as immediately fulfilled):

```js
const promiseA = new Promise((resolve, reject) => {
  setTimeout(reject, 1000, 'A');
});

const promiseB = new Promise((resolve) => {
  setTimeout(resolve, 2000, 'B');
});

Promise.race([promiseA, promiseB]) // => 'A' Rejects after one second
  .then((response) =>  console.log(response))
  .catch((error) => console.error(error));

Promise.any([promiseA, promiseB]) // => 'B' Fulfills after two seconds
  .then((response) => console.log(response))
  .catch((error) => console.error(error))
```

Another difference between the two methods is that passing an empty array (or any other empty iterable) to `Promise.race()` results in a promise that remains in pending state.

## Enforcing a Time Limit for Async Tasks

The `Promise.race()` method can be useful when fetching an external resource that may take a while to complete. With this method, we can race an async task against a promise that’s going to be rejected after a number of milliseconds. Depending on the promise that settles first, we either obtain the result or report an error message. In other words, we represent each outcome with a promise: the successful fetching from the API, or its failure:

```js
function fetchData() {
  const timeOut = 2000
  const data = fetch('https://jsonplaceholder.typicode.com/todos/1')
  // data contains a promise, not the data
  const failure = new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error(`Failed to fetch data after ${timeOut} milliseconds`))
    }, timeOut)
  })
  // as soon as the promise is created, the body of its definition is evaluated,
  // and the timeOut function is set: after 2 seconds, the promise will be rejected.
  
  // this means that, if data is not ready before the time limit, failure will settle
  // with failure, and Promise.race() will return a promise with the rejection value
  return Promise.race([data, failure])
}
```

### Improving the program with cached data

We can further improve this code by using cached data if fresh data isn’t available on time by adding a cache:

```js
function fetchData() {
  const timeOut = 2000
  const data = fetch('https://jsonplaceholder.typicode.com/todos/1')
  const cachedData = loadFromCache().then(cache => { // this simulates the loading process from a cache
    return new Promise(resolve => setTimeout(() => resolve(cache), timeOut))
  })

  return Promise.race([data, cachedData])
}

function loadFromCache() {
  const cachedData = {
    userId: 1,
    id: 1,
    title: `cached data`,
    completed: false
  };

  return new Promise(resolve => resolve(cachedData))
}

fetchData()
  .then((response) => console.log(response))
  .catch((error) => console.error(error))
```

In a real-world app probably we will load the cached data from a database.

## Other Use Cases

Of course, using cached data would work only for certain types of information that doesn’t change frequently. If you’re retrieving data like stock prices or exchange rates, then using `Promise.any()` is a better choice (see [Avoiding the Single Point of Failure](../04_improving_reliability_and_performance/notes.md)) since it allows you to request data from multiple APIs and use the result of the first that's accessible.

An interesting use case for `Promise.race()` is to batch async requests: if you have to make numerous async requests and don’t want the pending requests to get out of hand, you can use `Promise.race()` “to keep a fixed number of parallel promises running and add one to replace whenever one completes.” Using `Promise.race()` in this way lets you run multiple jobs in a batched way while preventing too much work from happening at one time.

You can also apply `Promise.race()` to a computationally expensive background task. It’s easy to imagine cases where some task might be attempted in the background, such as rendering a complex canvas while the user is occupied with something else. Using `Promise.race()` there, again gives you some knowable time to work with—and the opportunity to introduce some logic of what to do should the task fail.

