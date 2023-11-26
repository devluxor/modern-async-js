# 04: Improving Reliability and Performance

ES2021 `Promise.any()` is a recent addition to ECMAScript that helps us achieve both of these goals at the same time. We can protect our app from potential API downtimes by making network requests to multiple APIs asynchronously and using the result of the one that’s accessible. What’s more, we can improve the performance of critical application services by using the API that responds first.

## Using the `Promise.any()` Method

`Promise.any()` returns a pending promise that resolves asynchronously as soon as one of the promises in the given iterable fulfills.

Note that, if more than one promise is fulfilled, only the first one will be used. The other fulfillment values are ignored.

```js
const promises = [
  Promise.reject(new Error('failure #1')),
  Promise.reject(new Error('failure #2')),
  Promise.resolve('YES!')
];

Promise.any(promises).
  then((result) => console.log(result));

// logs:
// => YES!
```

If all promises are rejected, an `AggregateError` object is thrown, wrapping all the rejection reasons of all the input promises in a single error, and we can read them using the `errors` property:

```js
const promises = [
  Promise.reject(new Error('failure #1')),
  Promise.reject(new Error('failure #2')),
  Promise.reject(new Error('failure #3'))
];

Promise.any(promises).then(
  result => console.log(result),
  error => console.error(error.errors)
);
```

We will see in the console:

```text
...
0: Error: failure #1 at <anonymous>:2:18
1: Error: failure #2 at <anonymous>:3:18
2: Error: failure #3 at <anonymous>:4:18
...
```

Passing an empty iterable also throws an `AggregateError` _with the same message_.

```js
Promise.any([]).then(
  result => console.log(resut),
  error => console.log(error)
)

// logs: AggregateError: No Promise in Promise.any was resolved
```

## Avoiding the Single Point of Failure

A single point of failure (SPOF) is a component of a system that with just one malfunction or fault will stop the entire system from working. If you want to have a reliable application, you should be able to identify and avoid potential SPOFs in the system.

A common SPOF in web applications occurs when fetching critical resources, such as data for financial markets, from external APIs. If the API is inaccessible, the app will stop working. The `Promise.any()` method is extremely useful in this regard. It enables us to request data from multiple APIs and use the result of the first successful promise.

```js
const apis = [  'https://eloux.com/todos/1',  'https://jsonplaceholder.typicode.com/todos/1'   ];

async function fetchData(api) {
  const response = await fetch(api)
  if (!response.ok) return Promise.reject(new Error('Request failed'))

  return response.json()
}

function getData() {
  return Promise
    .any(apis.map(p => fetchData(p)))
    .catch(() => Promise.reject(new Error('Unable to access the API')))
}

getData()
.then(response => console.log(response))
.catch(error => console.log(error))
```

We’ve learned how to execute multiple promises at the same time to make our app more reliable. But this approach has one more benefit: performance.

## Improving the Performance of Your App

So, in addition to avoiding the single point of failure issue, we can use `Promise.any()` to improve the performance of your application, because only the first (fastest) promise is used, as soon as it's ready, and the others (slower) promises are ignored.



