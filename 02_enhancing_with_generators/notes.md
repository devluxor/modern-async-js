# 02: Enhancing With Generators

JavaScript provides generator functions as a shortcut to create iterators.

Every generator function is an iterator, but the opposite is not true. You may want to define a custom iterator protocol directly when you need an object with complicated state-maintaining behavior or you want to provide other methods besides `next()`. But in most other cases, you are best suited to define a generator that returns an iterator because state maintenance is mainly done for you.

## Using a Generator to Define a Custom Iterator

When called, a generator function doesn’t execute its body immediately. Instead, it returns a special type of iterator known as a **generator** object.

We can run the generator function’s body by calling the `next()` method of the generator object. 

The `yield` keyword pauses the generator and specifies the value to be returned:

```js
const collection = {
  a: 10,
  b: 20,
  c: 30,
  // this method returns a generator
  // we run the method's body by calling the next()
  // of the generator
  [Symbol.iterator]: function*() {  
    for (let key in this) {
      yield this[key];
    }
  }
};

const iterator = collection[Symbol.iterator]();
  
console.log(iterator.next());  // {value: 10, done: false}
console.log(iterator.next());  // {value: 20, done: false}
console.log(iterator.next());  // {value: 30, done: false}
console.log(iterator.next());  // {value: undefined, done: true}
```

Notice the asterisk `*` following the `function` keyword at line 5. This is our generator function and defines a custom iterator for collection.

With each iteration, the `yield` keyword halts the loop’s execution and returns the value of the expression after it (the succeeding property in this case) to the caller.

It’s possible to call a generator function as many times as needed, and each time it returns a new generator object. But a generator object can be iterated only once. 

## Creating an Asynchoronous Generator

An async generator is similar to a sync generator in that calling `next()` resumes the execution of the generator until reaching the yield keyword. But rather than returning a plain object, `next()` returns a promise:

```js
const srcArr = [
  'https://eloux.com/async_js/examples/1.json',
  'https://eloux.com/async_js/examples/2.json',
  'https://eloux.com/async_js/examples/3.json',
];

srcArr[Symbol.asyncIterator] = async function*() {  
  let i = 0;
  for (const url of this) {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error('Unable to retrieve URL: ' + response.status);
    }
    yield response.json();
  }
};

const iterator = srcArr[Symbol.asyncIterator]();

iterator.next().then(result => {
  console.log(result.value.firstName);  // â‡’ John
});

iterator.next().then(result => {
  console.log(result.value.firstName);  // â‡’ Peter
});

iterator.next().then(result => {
  console.log(result.value.firstName);  // â‡’ Anna
});
```

In production, you’ll also want to use `catch()` to handle errors and rejected cases during the iteration. A well-designed program should be able to recover from common errors without terminating the application. You can chain a `catch()` method the same way as its sister method `then()`:

```js
iterator.next()
  .then(result => {
    console.log(result.value.firstName)
  })
  .catch(error => {
    console.log(error.message)
  })
```

## Iterating over Paginated data

One situation we want to use asynchronous iteration over synchronous is when working with web APIs that provide paginated data. By using an asynchronous iterator, we can seamlessly perform multiple network requests asynchronously and iterate over the results.

For example, GitHub provides an API that allows us to retrieve commits for a repository. The response is in JSON format and contains the data for the last 30 commits of the repository. The API will also provide pagination link headers for the remaining commits.

Say we want to retrieve info for the last 90 commits of a particular GitHub repository:

```js
// create an async generator function
async function* generator(repo) {

  // create an infinite loop
  for (;;) {

    // fetch the repo
    const response = await fetch(repo);

    // parse the body text as JSON
    const data = await response.json();

    // yield the info of each commit
    for (let commit of data) {
      yield commit;
    }

    // extract the URL of the next page from the headers
    const link = response.headers.get('Link');  
    repo = /<(.*?)>; rel="next"/.exec(link)?. [1];
    
    // if there's no "next page", break the loop.
    if (repo === undefined) {
      break;  
    }
  }
}

async function getCommits(repo) {

  // set a counter
  let i = 0;

  for await (const commit of generator(repo)) {

    // process the commit
    console.log(commit);

    // break at 90 commits
    if (++i === 90) {  
      break;
    }
  }
}

getCommits('https://api.github.com/repos/tc39/proposal-temporal/commits');
```

The takeaway from this example is that asynchronous generators allow us to smoothly and continuously make several network requests and iterate over the results.

## Other

Another interesting use case for asynchronous generator would be fetching images from a photo sharing website like Flickr. The Flickr API provides an endpoint for fetching images based on given keywords. With an async generator function, you can fetch and navigate the batches asynchronously. Using an async generator would also open up the possibility to seamlessly aggregate photos from several sources.

>Asynchronous iterators provide an alternative to 'data' events on streams in Node and can be used to represent a stream of user input events in client-side JavaScript.


