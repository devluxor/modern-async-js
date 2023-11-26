# Accessing Promise Results from Another Module

Often in modular programs, you need the result from another module before you run the main module.

Top-level await is an addition to the language that provides a straightforward way to use the `await` keyword outside of `async` functions so that we can perform async tasks directly at the top level of the module.

Keep in mind you can’t use top-level await in classic scripts—it only works in module scripts and browser dev tools. If you get a `SyntaxError` (`await is only valid async functions...`), that means you’re not using it in a module.

Adding a module to an HTML file is the same as adding a regular script except that you should add a `type` attribute with the value of `module`, like this:

```html
<script type="module" src="module1.js"></script>
```

Also, modules are subject to same-origin policy, meaning that you can’t import them from the file system.

## Using Top-Level `await`

When the await keyword was first introduced, it wasn’t possible to use it outside `async` functions.

As a way to get access to the feature, we often wrapped the await statements in an immediately invoked async function expression:

```js
(async () => {
  const response = await fetch('url')
})()
```

But with top-level `await` we no longer have to do this, because the `await` keyword works outside `async` functions as well.

When working with ES modules, we can make variables and functions available outside the module using the `export` keyword. Then other modules in separate files can use the `import` keyword to access those variables and functions. Any `export` or `import` statement must be expressed at the top level of the code.

Say we have a module that retrieves weather data for Tokyo, Japan, from an external API, and we want to make the result available to other modules:

```js
let result;
const api = `http://api.openweathermap.org/data/2.5/weather?q=Tokyo,Japan&APPID=1b1b3e9e909416e5bbe365a0a8505fbb`;

(async () => {
  const response = await fetch(api);
  result = await response.json();
})();

export {result};
```

In another module we want to import the result and extract the temperature:

```js
import {result} from './module'

console.log(result.main.temp) // => TypeError: Cannot read property 'main' of undefined 
```

This is the case because we are trying to access the export before the async funcion finishes executing. We still have a promise waiting to be settled; until then, `result` has a value of `undefined`: exported variables are `undeﬁned` until the promise is settled. 

We can’t use the export keyword inside functions, and prior to the introduction of top-level await, we couldn’t use the `await` keyword outside `async` functions either.

One workaround is to export the entire async function as the default export value:

```js
let result;
const api = `http://api.openweathermap.org/data/2.5/weather?q=Tokyo,Japan&APPID=1b1b3e9e909416e5bbe365a0a8505fbb`;

export default (async () => {
  const response = await fetch(api);
  result = await response.json();
})();
```

But as the code becomes more complicated, it will become more difficult to manage the modules this way. Other workarounds could work as well, but they come with their own limitations.

Top-level `await` aims to solve this problem by enabling developers to use the `await` keyword outside async functions. You don’t need to do anything special to start using top-level `await` except having a modern browser that supports the feature:

```js
const api = `http://api.openweathermap.org/data/2.5/weather?q=Tokyo,Japan&APPID=1b1b3e9e909416e5bbe365a0a8505fbb`;  
const response = await fetch(api);   
const result = await response.json();      

export {result};
```

With top-level await, ECMAScript modules can await resources, causing other modules who import them to wait before they start evaluating their own code.

## Putting Top-Level `await` to Work

### Dynamically load a language pack according the client's browser language

When designing a program to support multiple languages and regions, you may want to use a runtime value to determine the language to use. Say you have an ES module and want to load a language pack dynamically, based on the preferred language of the user set in the browser. You can take advantage of top-level `await` to import the messages:

```js
const messages = await import(`./packs/messages-${navigator.language}.js`);
```

The `navigator.language` property allows us to access the preferred language of the user, which is usually the language of the browser UI.

The module will be waiting for the language pack to be imported and can only evaluate the rest of the body once the pack has been loaded.

### Load dependencies with a fallback implementation

>Fallback is a contingency option to be taken if the preferred choice is unavailable.

It’s important to protect our app against external server issues. Network requests to a server might fail. In critical applications, you can provide dependency fallbacks to mitigate such failures using top-level `await`:

```js
// D3 JavaScript library
let d3;      
try {  
  d3 = await import('https://cdnjs.cloudflare.com/ajax/libs/d3/6.7.0/d3.min.js');   
} catch {
  // fallback  
  d3 = await import('https://ajax.googleapis.com/ajax/libs/d3js/6.7.0/d3.min.js');   
}
```

Alternatively, we can use `Promise.any()` to execute both requests asynchronously and use the result of the one that responds faster:

```js
const CDNs = [  
  'https://cdnjs.cloudflare.com/ajax/libs/d3/6.7.0/d3.min.js', 
  'https://ajax.googleapis.com/ajax/libs/d3js/6.7.0/d3.min.js'   
];
const d3 = await Promise.any(CDNs.map(s => fetch(s)));
```

### Resource initialization

```js
import {dbConnector} from './utilities.js';    

const connection = await dbConnector.connect();
```

By using top-level await, we can make the module behave like a big async function. We can now represent resources with await and handle errors if the module can’t be used. 

Remember, a module won’t start evaluating its body until the module that’s being imported has finished executing its body. So if the other module has a top-level `await`, it must be completed before the module that’s importing it begins executing.