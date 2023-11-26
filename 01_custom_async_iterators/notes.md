# Creating Custom Asynchronous Iterators

What about when you need to process asynchronous data via web APIs, like stock prices? Synchronous iterators cannot represent such data sources, so that’s where you’ll need to use asynchronous iteration.

Following are the JavaScript features we’ll discuss in this chapter along with some links to up-to-date sources for browser support: 

- Async functions.
- `for await...of` loops.
- `async` and `await` keywords.

## Creating a Custom Iterator

Collection objects (including Array, Set, and Map) come with built-in iterators that allow us to navigate their values. But sometimes these objects don’t serve our purpose. What if we want to customize the iteration behavior of an object to return values backward or randomly? Or iterate over a plain object or class, both being not iterable by default? In that case, we’ll need to define our own `Symbol.iterator`.

Iterable is an object that allows its values to be looped over in a `for...of` construct. It does so by providing a method whose key is `Symbol.iterator`. That method should be able to produce any number of iterators. To be classified as an iterable, an object must come with a `Symbol.iterator` property and specify the return value for each iteration.

Iterator, on the other hand, is an object that’s used to obtain the values to be iterated.

In the following example, we have a plain object that’s iterable because we’ve defined an iterable protocol that allows us to access the items of the object one at a time:

```js
const collection = {  
  a: 10,  
  b: 20,  
  c: 30,  
  [Symbol.iterator]() {  // we define a method whose key is [Symbol.iterator]
    let i = 0;  // initial index
    const values = Object.keys(this); // an array of ['a', 'b', 'c']
    
    // returns an iterator: an object that has a next() method
    return {  
      // the next() method returns an object with two properties
        //  value: current value, and adds one to the index
        //  done: a boolean to indicate if the iteration has ended
      next: () => {  
        return {
          value: this[values[i++]], 
          done: i > values.length  
        }  
      }  
    };  
  }  
};      

const iterator = collection[Symbol.iterator]();      
console.log(iterator.next());   // ? {value: 10, done: false}    
console.log(iterator.next());   // ? {value: 20, done: false}    
console.log(iterator.next());   // ? {value: 30, done: false}    
console.log(iterator.next());   // ? {value: undefined, done: true}
```

Using `next()` isn’t the only way to iterate over iterable objects. The `for...of` statement lets us create a loop and easily repeat the same function on iterable objects. `for...of` works better if we want to quickly get the values of all items in the object. `next()`, on the other hand, is more verbose but allows us to see what’s happening in each iteration:

```js
const collection = {
  a: 10,
  b: 20,
  c: 30,
  [Symbol.iterator]() {
    let i = 0;
    const values = Object.keys(this);

    return {
      next: () => {
        return {
          value: this[values[i++]],
          done: i > values.length
        }
      }
    };
  }
};

// the for construct already knows how to use the method.
for ( const value of collection) {
  console.log(value);
}

 // logs: 
 // ⇒ 10 
 // ⇒ 20 
 // ⇒ 30 
```

The iterator object is designed to maintain an internal pointer (the `value` property) to a position in the items; and each time through the loop, it gives the succeeding value.

Calling `[Symbol.iterator]()` on an array will return the result of the `values()` method because that’s the default iterator of arrays, and also sets.

`entries()` is the default iterator of maps. 

An object may have several iterators, such as `keys()`, `values()`, and `entries()`, but only one of them serves as the default iterator. 

Built-in iterators makes possible to iterate over collection objects easily, and custom iterators allow us to define or customize the iteration behavior of objects. But if we want to work with asynchronous sources, we’ll need to create custom asynchronous iterators.

## Creating a Custom Asynchronous Iterator

An asynchronous iterator is very similar to a regular non-async iterator except that its` next()` method returns a promise rather than a plain object. Thus, instead of immediately returning the result, the promise will provide the value (or failure reason) at some point in the future.

An object is classified as asynchronous iterable when it has a `Symbol.asyncIterator` method that returns an asynchronous iterator:

```js
const collection = {
  a: 10,
  b: 20,
  c: 30,
  [Symbol.asyncIterator]() {
    const keys = Object.keys(this);
    let i = 0;
    return {
      next: () => {
        return new Promise((resolve, reject) => {   
          setTimeout(() => {
            resolve({
              value: this[keys[i++]],
              done: i > keys.length
            });
          }, 1000);
        });
      }
    };
  }
};

const iterator = collection[Symbol.asyncIterator]();

iterator.next().then(result => {
  console.log(result);  // => {value: 10, done: false}
});

iterator.next().then(result => {
  console.log(result);  // => {value: 20, done: false} 
});

iterator.next().then(result => {
  console.log(result);  // => {value: 30, done: false} 
});

iterator.next().then(result => {
  console.log(result);  // => {value: undefined, done: true} 
});
```

For the sake of simplicity, we’ve used the `setTimeout()` method to resolve the promise after one second. But in a real-world example we’re more likely to make a call to an API and wait for a response.

## Retrieving URLs separately

Suppose we want a function that retrieves several URLs and processes the result of each URL separately before moving on to the next. In other words, we want to retrieve and parse the URLs asynchronously, but not the results. That’s one scenario where an asynchronous iterator is useful:

```js
const srcArr = [
  'https://eloux.com/async_js/examples/1.json',
  'https://eloux.com/async_js/examples/2.json',
  'https://eloux.com/async_js/examples/3.json',
];

// we add the method to the object as we'd normally do
srcArr[Symbol.asyncIterator] = function() { 
  let i = 0;

  // the iterator returned contains an async next() method
  return {
    async next() {
      if (i === srcArr.length) return { done: true };

      const url = srcArr[i++];
      const response = await fetch(url);
      
      // promise rejection
      if (!response.ok) throw new Error('Unable to retrieve URL: ' + url);

      return {
        value: await response.json(),
        done: false
      };
    }
  };
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

It’s also important to ensure the response was successful (status in the range 200–299) before proceeding further.

Async iterators are invaluable tools when working with web APIs. Often, the data can only be retrieved in the form of stream or pagination. Iterators make it possible to gracefully obtain the amount of data we need and process them.

## Iterating over Async Iterables with `for…await…of`

Sometimes we want a more straightforward way of accessing the items of an async iterable. We want to quickly get the result of all promises and terminate the loop automatically once the done property has a value of `true`.

The `for..of` loop does allow you to loop over iterable objects, but it doesn’t work with asynchronous iterables (returns `undefined`). ES2018 introduced `for...await...of` as a variant of `for...of` that can iterate over both sync and async iterables.

Let’s look at this rewritten version of the previous example. Notice how for...await...of saves lines of code by executing the same statement for the value of each property:

```js
const srcArr = [
  'https://eloux.com/async_js/examples/1.json',
  'https://eloux.com/async_js/examples/2.json',
  'https://eloux.com/async_js/examples/3.json',
];

srcArr[Symbol.asyncIterator] = function() {
  let i = 0;

  return {
    async next() {
      if (i === srcArr.length) {
        return { done: true};
      }

      const url = srcArr[i++];
      const response = await fetch(url);

      if (!response.ok) {
        throw new Error('Unable to retrieve URL: ' + url);
      }

      return {
        value: await response.json(),
        done: false
      };
    }
  };
};

// we need to enclose the call within an async function
(async function() {

  for await (const url of srcArr) {
    console.log(url.firstName);
  }
})();

// logs:
// â‡’ John
// â‡’ Peter
// â‡’ Anna
```

When we run this code, the JavaScript engine executes the `Symbol.asyncIterator` method of the object to obtain an asynchronous iterator. With each iteration of the loop, the iterator executes the `next()` method and returns a promise (this happens behind the scenes). As soon as the promise is fulfilled, the value of the value property is assigned to `url`.

It’s a common practice to enclose `for...await...of` in a `try...catch` statement. This way when a promise rejects, we can gracefully handle the rejection.

The `for...await...of` statement provides a convenient, concise way of accessing the items of an async iterable. By wrapping it in a `try...catch` statement, we have the ability to handle promise rejections the way we want.

> An interesting aspect of iterators is that they are infinite. For instance, you may have a Fibonacci iterator that delivers an infinite sequence.

## Detecting Whether an Object is Iterable

It’s important to ensure that the object is iterable; otherwise, the code may throw a `TypeError`.

Detecting whether an object is iterable isn’t complicated: check for the existence of [Symbol.iterator] on the object and ensure it’s a function:

```js
function isIterable(object) {
  return typeof object[Symbol.iterator] === 'function'
}

function isAsyncIterable(object) {
  return typeof object[Symbol.asyncIterator] === 'function'
}
```



