# Introduction

## Demystifying Asynchronous Execution

The concept of asynchrony determines whether a task can start executing before another task is finished. In a synchronous execution, the program pauses until the current task is completed before moving to the next task. But in an asynchronous execution, the program continues executing even when the previous operation hasn’t finished yet.

Asynchrony and multithreading are two completely different concepts. JavaScript is often considered a single-threaded language, mostly because web browsers run one thread per global environment.

But JavaScript, as a programming language, isn’t single-threaded. And there are some JavaScript environments that are multi-threaded. With the introduction of Web Workers, you can even have multiple threads on web browsers (they don’t run on the same global environment though).

But even on a single thread, JavaScript is capable of executing asynchronous code. Threads aren’t the only way to perform tasks in parallel.

Threading describes the number of _workers_, but asynchrony is about _tasks_.

## Working with events

The JavaScript language was created to add interactivity to web pages, so it needed a way to detect user actions and react to them. JavaScript’s solution for this need was events: whenever you interact with a web page, such as when clicking a button, an event takes place, allowing JavaScript code to react to the action.

Although events have enabled JavaScript programs to detect interaction with objects and react to them, its lack of flexibility has been a significant problem for some developers. For example, events can happen before the program starts listening to them.

## Working with callback functions

The simplest asynchronous execution in JavaScript is the `setTimeOut()` function. This function defines a callback function to be executed in the future independently of the main program flow, so it doesn’t block the execution of the program. 

Another common asynchronous execution in JavaScript is Ajax. It specifies a piece of code to run as soon as the code receives data from a server. The main advantage of using callbacks is that the program can continue doing useful work while other tasks are running

Nesting callbacks is a common practice in JavaScript. But nesting too many callbacks can make the code hard to understand. By moving functions to the top level, we’ll have a shallower code that is separated into small logical sections. This small change results in a more manageable code. Still, the callback model is difficult to work with when more complex functionality is needed.

With promises, you can easily chain multiple asynchronous tasks dynamically.

Callbacks are still useful when your code may receive a notification more than once. For instance, the `setInterval()` method defines a callback function to be executed repeatedly, with a fixed time delay between each call. You can’t call a promise again once it’s executed, but you can call a callback function multiple times.

## Introducing Promises

The `fetch()` method allows us to retrieve files across the network. This method returns a promise object that acts as a placeholder for the future result of the operation. To react to the result, we use the `then()` method.

It’s worth noting that a promise cannot succeed or fail more than once. It also cannot switch from failure to success or vice versa.

## Creating Settled Promises

The static `Promise.resolve()` and `Promise.reject()` methods allow you to quickly create settled promises and see how the code works when you give it a different value.

Don’t forget to take advantage of these methods when debugging your code.

## Handling Rejection

There are two primary ways to handle a rejected promise. In the previous example, we used the pattern `then(fulfill, reject)`, but we can also use the `catch()` method. 

When chaining promises and an error occurs, the interpreter skips all `then()` methods that follow and executes the first `catch()` method it can find.

## Managing Multiple Concurrent Promises

Methods to achieve real concurrency:

- `Promise.race()` – lets you know as soon as one of the given promises either fulfills or rejects 
- `Promise.allSettled()` – lets you know when all of the given promises either fulfill or reject 
- `Promise.all()` – lets you know as soon as one of the given promises rejects or when all of them fulfill 
- `Promise.any()` – lets you know as soon as one of the given promises fulfills or when none of them fulfills

