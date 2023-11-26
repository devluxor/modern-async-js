# Quick Reference Guide

## Promise Parallelism Methods

- `Promise.all()`: One of the Promise concurrency methods (it will tell all the promises in the argument array to run concurrently). It can be useful for aggregating the results of multiple promises. This method takes an array of `Promise` objects as argument and returns another Promise. That Promise will be fulfilled with an array of the fulfillment values of each of the input Promises in the array argument; it will be rejected if any of the input Promises is rejected. The argument array can take any kind of value: for non-Promise values, it will be treated as it is the value of an already fulfilled Promise.  
- `Promise.allSettled()`: this method takes an array of Promises and returns another Promise. This Promise won't be fulfilled until all the Promises in the input array have settled. The value of this Promise is an array of special objects, one per input Promise, that have three properties: `status`, `value`, and `reason`, all self-explanatory.  
- `Promise.race()`: This method takes an array of Promises and returns another Promise. This Promise is fulfilled or rejected when the first of the Promises in the input array is fulfilled or rejected (or the first of non-Promise values, as they count as immediately fulfilled).
- `Promise.any()`: This method takes an iterable (like an array) of promises as input and returns a single Promise. This returned promise fulfills when any of the input's promises fulfills, with this first fulfillment value, ignoring the rest. It rejects when all the input's promises reject (including when an empty iterable is passed) with an `AggregateError` containing an array of rejection reasons.

## `blob()`

- `Response.prototype.blob()` The `blob()` method of the `Response` interface takes a `Response` stream and reads it to completion. It returns a promise that resolves with a Blob.

### Example of use case

We can fetch a JPG image, and, when the fetch is successful, we read a Blob out of the response using `blob()`, put it into an object `URL` using `URL.createObjectURL()`, and then set that URL as the source of an `<img>` element to display the image:

```js
fetch(myRequest)
  .then((response) => response.blob())
  .then((myBlob) => {
    const objectURL = URL.createObjectURL(myBlob);
    myImage.src = objectURL;
  });
```

## `AbortController`

The `AbortController` interface represents a controller object that allows you to abort one or more Web requests as and when desired.

You can create a new `AbortController` object using the `AbortController()` constructor. Communicating with a DOM request is done using an `AbortSignal` object.

- `AbortController.signal`: Returns an `AbortSignal` object instance, which can be used to communicate with, or to abort, a DOM request.

- `AbortController.abort()`: Aborts a DOM request before it has completed. This is able to abort fetch requests, consumption of any response bodies, and streams.
