# 03: Fetching Multiple Resources

Suppose you want to take an action after multiple async requests have completed, regardless of their success or failure. For example, you need to obtain data from four separate web APIs and process the result, but there might be a network error for a resource that you can live without. The `Promise.all()` method isn’t suitable for this task because a single network error will cause the method to reject immediately. Fortunately, ECMAScript provides a newer tool that’s designed to report the outcome of all requests: `Promise.allSettled()`. With this method, we can track the state of multiple promises without letting any promise spoil the result of others. We’ll start this chapter by examining a common async task: executing multiple promises and handling the result. Once you’ve learned about potential pitfalls, we’ll look at the `Promise.allSettled()` method and see how it compares to `Promise.all()`.

## Executing Multiple Promises

When creating complex JavaScript applications, you’ll inevitably encounter circumstances where you need to execute multiple promises. Say you have an async function that retrieves a blog post, like this:

```js
async function getPost(id = 1) {
  try {
    return await Utility.loadPost(id)
  } catch(error) {
    // handles error
  }
}
```

What if you need to retrieve multiple posts? If we try this:

```js
const postIds = ['1', '2', '3', '4'];
postIds.forEach(async id => {
  const post = await getPost(id);
  // process the post   
})
```

The `await` keyword will pause the loop until it gets a response from `getPost()`. In other words, this code will load the posts sequentially rather than asynchronously (making multiple requests at the same time).

One way to fix this issue is to use the `Promise.all()` method:

```js
const postIds = ['1', '2', '3', '4'];

const promises = postIds.map(async (id) => {
  return await getPost(id);
});

const arr = Promise.all(promises);
```

But If one of the promises in the iterable rejects, `Promise.all()` immediately rejects, causing every other post not to load.

With the `Promise.allSettled()` method, we can get the result of all promises passed to the method.

## Using `Promise.allSettled()` to Fetch Multiple Resources

The `Promise.allSettled()` method returns a promise that resolves when all the given promises have either successfully fulfilled or rejected (“settled,” in other words). This behavior is very useful to track multiple asynchronous tasks that are not dependent on one another to complete.

This method returns an array of special objects:

```js
const promises = [
  fetch('https://picsum.photos/200', {mode: "no-cors"}), 
  fetch('https://does-not-exist', {mode: "no-cors"}),
  fetch('https://picsum.photos/100/200', {mode: "no-cors"})
];

Promise.allSettled(promises)
  .then((results) => results.forEach((result) => console.log(result)));

// logs:
// => { status: "fulfilled", value: Response }
// => { status: "rejected", reason: TypeError }
// => { status: "fulfilled", value: Response }
```

The promise will reject if and only if we pass a value that’s not iterable, such as a plain object.

If you’re in an old JavaScript environment that doesn’t support `Promise.allSettled()` or if you’d like to directly return the promises, there’s a simple workaround for you:

```js
const promises = [
  fetch('https://picsum.photos/200', {mode: "no-cors"}),
  fetch('https://does-not-exist', {mode: "no-cors"}),
  fetch('https://picsum.photos/100/200', {mode: "no-cors"})
].map(p => p.catch(e => e)); 

Promise.all(promises).
  then((results) => {
    results.forEach((result) => console.log(result))
  });
```

We’ve applied the `map()` method to an iterable of promises. Within the method, we use `catch()` to return promises that resolve with an error value. This way, we can simulate the behavior of `Promise.allSettled()` while being able to
directly access the result of promises.

You may find yourself in a situation where you need to read a local file, retrieve a JSON document from a web API, and load an XML document from another API. Once you obtain data from all three async requests, you want to process them. `Promise.all()` and `Promise.allSettled()` are ideal for such scenarios.

You will want to use these methods only when you need to process the result of multiple async requests together. If it’s possible to process the result of each async request individually, then handle each promise with its own `then()` handler. This way, you can execute your code as soon as each promise is resolved.

While `Promise.all()` is very strict in its execution policy, `Promise.allSettled()` is forgiving. That doesn’t mean `Promise.allSettled()` is superior to `Promise.all()`: they complement each other:

- `Promise.all()` is more appropriate when you have essential async tasks that are dependent on each other.
- `Promise.allSettled()` is more suitable for async tasks that might fail but are not essential for your program to function.

As of ES2021, the ECMAScript standard includes one more method for the promise object: `Promise.any()`. This method is the opposite of `Promise.all()`.

