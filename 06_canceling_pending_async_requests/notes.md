# Canceling Pending Async Requests

But sometimes you may want to cancel a pending async request before it has completed. Perhaps you have a network-intensive application and async requests are taking too long to fulfill, or maybe the user clicked a Cancel button.

The `AbortController` API provides a generic interface that allows you to cancel a fetch request. The cornerstone of the API is the `AbortController` interface, which provides an `abort()` method. You can create a cancelable `fetch` request by passing the signal property of `AbortController` as an option to `fetch()`. Later, when you want to abort the fetch, simply call the `abort()` method to terminate the network transmission.

## Canceling Async Tasks After a Period of Time

With the `AbortController` API, we can cancel a request that we have already issued but don’t want to wait for the operation to finish.

> When you only want to automatically cancel a fetch request after a specified amount of time, you can use either `Promise.race()` or `AbortController`. It’s a matter of preference.
>
> `AbortController` is easier to use if you want to cancel an async request once a specific DOM event is fired, such as when the user clicks a cancel button (or some time passes??). `Promise.race()`, on the other hand, works better if you want to concurrently fetch data and perform another  task, such as pulling backup from a database.

When we run this code, the `setTimeout()` method sets a two-second timer to execute `abort()`. If the fetch is complete in the allotted time, the abort will have no effect. If not, an error is thrown:

```js
// we need to create an instance of AbortController
// before initiating the fetch request
const controller = new AbortController();
const signal = controller.signal;

// note the options object passed to fetch
// we are linking the controller with the request
// this link enables us to abort the request
// via the controller object
fetch('https://eloux.com/todos/1', {signal})
  .then(response => return response.json())
  .then(response => console.log(response))

setTimeout(() => controller.abort(), 2000);
```

`abort()` is the only method of controller and will cause the promise object returned by fetch to reject with an exception:

```text
Uncaught (in promise) DOMException: The operation was aborted.
```

At this point, the control will be passed to the catch() method (if it exists). Upon calling `abort()`, the API will notify the signal, which, if you want, you can listen to by attaching an event handler:

```js
// after the aborting succeeds, the aborted property of signal 
// has a value of true.
signal.addEventListener('abort', () => {
  console.log(signal.aborted);
});

// logs: true 
```

We can create a custom `fetchWithTimeout` maker function to add an abort interface to every request automatically:

```js
function fetchWithTimeout(url, settings, timeout) {
  // If the timeout argument doesn't exists
  if (timeout === undefined) {
    return fetch(url, settings); 
  }
  
  // if timeout isn't an integer, throw an error
  if (!Number.isInteger(timeout)) {
    throw new TypeError('The third argument is not an integer')   
  }
  
  const controller = new AbortController(); 
  setTimeout(() => controller.abort(), timeout);
  settings.signal = controller.signal;
  return fetch(url, settings);
}
```

This function works like a `fetch()` method but provides the option of setting a timeout. If we pass an integer (in milliseconds) as the third argument, the request will abort after the time expires. If not, it will retrieve the resource like a regular `fetch()`.

It’s always a good idea to set a time limit for async requests to avoid keeping your users waiting.

## Handling an Aborted Request

When `abort()` successfully cancels a request, the pending promise rejects with a `DOMException` error. But you don’t want to show the default error message if the operation is canceled by the user. After all, it’s not considered an error if the cancelation is intentional:

```js
const src = 'https://eloux.com/todos/1';
const controller = new AbortController();
const signal = controller.signal;

fetch(src, {signal})
  .then(response => return response.json())
  .then(json => console.log(json))
  .catch(error => {
    // To ensure we’re handling the abort error separately, 
    // we can check the name property of the error.
    if (error.name === 'AbortError') {
      console.log('Request successfully canceled');
    } else {
      console.error('Fetch failed!', error);
    }
  });

controller.abort();

// logs: Request successfully cancelled
```

## Removing Multiple Event Listeners

In client-side JavaScript programming, the flow of the code is determined by events. If we want to do something when a particular event occurs, we can register one or more functions to be called using the `addEventListener()` method.

We can later remove an event handler function from an object using the `removeEventListener()` method. If we register dozens of event handlers, we’ll need the exact same number of `removeEventListener()` methods to deregister them, which unnecessarily bloats the code. 

With `AbortController` we can deregister multiple event listeners in only one statement. The `addEventListener()`  method now accepts an abort signal as the third argument. We can create a controller object and pass its signal property to `addEventListener()`, like this:

```js
const container = document.querySelector('.container');
const controller = new AbortController();
const signal = controller.signal;

function sayHello() {
  container.textContent = 'Hello';
}

function sayBye() {
  container.textContent = 'Bye!';
}

function depress() {
  container.style.backgroundColor = 'aqua';
}

function release() {
  container.style.backgroundColor = 'transparent';
}

container.addEventListener('mouseenter', sayHello, {signal});
container.addEventListener('mouseout', sayBye, {signal});
container.addEventListener('mousedown', depress, {signal});
container.addEventListener('mouseup', release, {signal});
```

Now, we can abort all `addEventListener()` methods with a single `AbortSignal`:

```js
controller.abort()
```

For example, suppose we have a long list of elements, and we want to enable the user to sort the list by dragging and dropping the elements. For each element, there’s an event attached to check for the state of dragging. We want to disable drag and drop once the user clicks Save. Using `AbortController` can be a time-saver in this situation: rather than removing each event handler separately, we can call `abort()` to remove them all.

## Making a User-Cancelable Async Request

When including large files on your page, you should take into account the fact that some users will be on limited bandwidth or mobile devices with expensive data plans. Therefore, the ability for a user to load and cancel loading large items is valuable.

### Use Case: loading a heavy image (22 Mb), with a load and a cancel buttons

First, define an HTML `<image>` element on the page. The `src` attribute of this element will be filled once the image is loaded. We also need an element to inform the user about the outcome, so define a `<span>` element with a class of result. Next, create the buttons. We’re going to disable the abort button until the load button is clicked, so give it a disabled attribute:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Making a User Cancelable Async Request</title>
    <meta name= "viewport" content= "width=device-width, initial-scale=1">
    <script src= "abort_ex08.js" defer></script>
  </head>
  <body>
    <img class="image" src="blob:https://eloux.com/120f08c4-b8a1-4c03-9016-7fd91a102051">
    <span class="result"></span>
    <button class="loadBtn">Load Photo</button>
    <button class="abortBtn" disabled="">Cancel Loading</button>
  </body>
</html>
```

Now, in the JavaScript file, we need to set up two functions: one to call when the Load Photo button is clicked, and other to call when the Cancel Loading button is clicked:

```js
// create a reference to each HTML element
const loadBtn = document.querySelector('.loadBtn');
const abortBtn = document.querySelector('.abortBtn');
const image = document.querySelector('.image');
const result = document.querySelector('.result');

const controller = new AbortController();

// load the image
loadBtn.addEventListener('click', async () => {  
  loadBtn.disabled = true;
  abortBtn.disabled = false;
  
  result.textContent = 'Loading...';

  try {
    const response = await fetch(
      `https://upload.wikimedia.org/wikipedia/commons/a/a3/Kayakistas_en_Glaciar_Grey.jpg`, 
      {signal: controller.signal}
    );

    // To be able to display the image we’ve retrieved, we need to convert it into an object URL
    const blob = await response.blob();  
    image.src = URL.createObjectURL(blob);  
    
    // remove the "Loading.." text
    result.textContent = '';
  } catch (err) {
    if (err.name === 'AbortError') {
      result.textContent = 'Request successfully canceled';
    } else {
      result.textContent = 'An error occurred!'
      console.error(err);
    }
  }

  loadBtn.disabled = false;
  abortBtn.disabled = true;
});

// abort the request
abortBtn.addEventListener('click', () => controller.abort());
```

> Blob stands for binary large object, which is a data type containing a collection of binary data. In JavaScript, Blob serves as an essential data interchange method for several APIs. They’re often used when working with data that isn’t in a JavaScript- native format, such as images, audio, or other multimedia objects.

## Aboorting Multiple Fetch Requests with One Signal

Just as we can abort multiple `addEventListener()` methods, we can abort multiple fetch requests with a single `AbortSignal`.

Let’s revise the previous example to fetch an array of images rather than a single image.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Making a User Cancelable Async Request</title>
    <meta name= "viewport" content= "width=device-width, initial-scale=1">
    <script src= "abort_ex08.js" defer></script>
  </head>
  <body>
    <div class="gallery"></div>
    <span class="result"></span>
    <button class="loadBtn">Load Photos</button>
    <button class="abortBtn" disabled="disabled">Cancel Loading</button>
  </body>
</html>
```

```js
const loadBtn = document.querySelector('.loadBtn');
const abortBtn = document.querySelector('.abortBtn');
const gallery = document.querySelector('.gallery');
const result = document.querySelector('.result');

const controller = new AbortController();

const urls = [
  `https://upload.wikimedia.org/wikipedia/commons/thumb/7/70/Por_do_Sol_em_Baixa_Grande.jpg/320px-Por_do_Sol_em_Baixa_Grande.jpg`,
  `https://upload.wikimedia.org/wikipedia/commons/thumb/5/5e/Zebrasoma_flavescens_Luc_Viatour.jpg/20px-Zebrasoma_flavescens_Luc_Viatour.jpg`,
  `https://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/Domestic_goat_kid_in_capeweed.jpg/320px-Domestic_goat_kid_in_capeweed.jpg`
];

loadBtn.addEventListener('click', async () => {
  loadBtn.disabled = true;
  abortBtn.disabled = false;
  
  result.textContent = 'Loading...';
  
  // we pass references to the same controller
  const tasks = urls.map(url => fetch(url, {signal: controller.signal}));  

  try {
    const response = await Promise.all(tasks);

    response.forEach(async (r) => {
      const img = document.createElement('img');
      const blob = await r.blob();
      img.src = URL.createObjectURL(blob);
      gallery.append(img);
    });

    result.textContent = '';
  } catch (err) {
    if (err.name === 'AbortError') {
      result.textContent = 'Request successfully canceled';
    } else {
      result.textContent = 'An error occurred!'
      console.error(err);
    }
  }
  
  loadBtn.disabled = false;
  abortBtn.disabled = true;
});

// every task linked to the controller (all) will be aborted
abortBtn.addEventListener('click', () => controller.abort());
```

Another use case for `abort()` could be live search: when the user types a character in the input, it triggers a search request; when that promise resolves, you want to show the search results. But if the user presses multiple keys, the first search might resolve before the last. Aborting the “stale” request ensures that the search results reflect the most recent query.

## Wrapping Up

An interesting aspect of the `AbortController` API is that it’s provided by the DOM standard and designed to be generic.

Make use of the `AbortController` API in your programs to cancel async requests that are no longer needed or taking too long to complete. You can do that by calling `abort()` directly, setting a timer to call `abort()`, or providing a cancel button for users to abort requests whenever they want. You can even use an `AbortController` to deregister an event listener, or multiple event listeners, which is an ability that JavaScript previously lacked.

