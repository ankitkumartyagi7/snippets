# JavaScript Interview Questions and Utilities

A collection of JavaScript interview questions, polyfills, utility functions, asynchronous programming patterns, React hooks, routing examples, and system-design-oriented implementations.

This repository is intended for developers preparing for JavaScript, React, and frontend engineering interviews.

## Table of Contents

- [JavaScript Polyfills](#javascript-polyfills)
  - [customBind](#custombind)
  - [myMap](#mymap)
  - [myReduce](#myreduce)
  - [myFlat](#myflat)
  - [myFilter](#myfilter)
- [Memoization](#memoization)
  - [JSON-Based Memoization](#json-based-memoization)
  - [Trie-Based Memoization](#trie-based-memoization)
- [Object Utilities](#object-utilities)
  - [deepClone](#deepclone)
  - [isEqual](#isequal)
  - [deepCopyWithAppend](#deepcopywithappend)
  - [flattenObject](#flattenobject)
- [Throttling and Rate Limiting](#throttling-and-rate-limiting)
  - [Flag-Based Throttle](#flag-based-throttle)
  - [Timestamp-Based Throttle](#timestamp-based-throttle)
  - [Throttle with Call Limit](#throttle-with-call-limit)
  - [Sliding-Window Rate Limiter](#sliding-window-rate-limiter)
  - [Queue-Based Rate Limiter](#queue-based-rate-limiter)
  - [Token-Bucket Rate Limiter](#token-bucket-rate-limiter)
  - [Advanced Rate Limiter](#advanced-rate-limiter)
- [Retry and Backoff](#retry-and-backoff)
- [Debouncing](#debouncing)
  - [Basic Debounce](#basic-debounce)
  - [Debounce with Cancel](#debounce-with-cancel)
  - [Debounce with Cancel and Flush](#debounce-with-cancel-and-flush)
- [Currying](#currying)
  - [Infinite Curry](#infinite-curry)
  - [Variadic Curry](#variadic-curry)
  - [Seed-Based Infinite Curry](#seed-based-infinite-curry)
- [Other JavaScript Utilities](#other-javascript-utilities)
  - [Class Names Utility](#class-names-utility)
  - [Custom setTimeout](#custom-settimeout)
  - [Pub/Sub](#pubsub)
  - [Async Function Composition](#async-function-composition)
  - [Pipeline](#pipeline)
- [React Examples](#react-examples)
  - [Uncontrolled Input Wrapper](#uncontrolled-input-wrapper)
  - [useDebounce](#usedebounce)
  - [useThrottle](#usethrottle)
  - [useFetch](#usefetch)
  - [useLocalStorage](#uselocalstorage)
  - [Custom useState](#custom-usestate)
- [React Router](#react-router)
  - [Role-Based Protected Routes](#role-based-protected-routes)
- [Asynchronous Programming](#asynchronous-programming)
  - [Task Scheduler](#task-scheduler)
  - [URL Crawler](#url-crawler)
  - [mapAsyncLimit](#mapasynclimit)
  - [Sequential vs Parallel Execution](#sequential-vs-parallel-execution)
- [Important Notes](#important-notes)

---

## JavaScript Polyfills

### customBind

A custom implementation of `Function.prototype.bind`.

```js
Function.prototype.customBind = function (context, ...args) {
  const fn = this;

  return function (...arg) {
    return fn.apply(context, [...args, ...arg]);
  };
};
```

Usage:

```js
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const user = { name: "Alex" };

const boundGreet = greet.customBind(user, "Hello");

console.log(boundGreet("!"));
// Hello, Alex!
```

### myMap

A custom implementation of `Array.prototype.map`.

```js
Array.prototype.myMap = function (callback) {
  const result = [];

  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i, this));
  }

  return result;
};
```

### myReduce

A custom implementation of `Array.prototype.reduce`.

```js
Array.prototype.myReduce = function (callback, initialValue) {
  const array = this;
  let accumulator = initialValue;
  let startIndex = 0;

  if (accumulator === undefined) {
    accumulator = array;
    startIndex = 1;
  }

  for (let i = startIndex; i < array.length; i++) {
    accumulator = callback(accumulator, array[i], i, array);
  }

  return accumulator;
};
```

### myFlat

Recursively flattens nested arrays.

```js
Array.prototype.myFlat = function () {
  const result = [];

  function flatten(array) {
    for (const item of array) {
      if (Array.isArray(item)) {
        flatten(item);
      } else {
        result.push(item);
      }
    }
  }

  flatten(this);
  return result;
};
```

Usage:

```js
console.log([1, [2,, 4]].myFlat());[1]
//[2][3][4][1]
```

### myFilter

A custom implementation of `Array.prototype.filter`.

```js
Array.prototype.myFilter = function (callback) {
  const result = [];

  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      result.push(this[i]);
    }
  }

  return result;
};
```

---

## Memoization

Memoization stores previously computed results to avoid repeating expensive calculations.

### JSON-Based Memoization

Suitable when function arguments can be safely serialized using `JSON.stringify`.

```js
function memoize(fn) {
  const cache = {};

  return function (...args) {
    const key = JSON.stringify(args);

    if (Object.prototype.hasOwnProperty.call(cache, key)) {
      return cache[key];
    }

    const result = fn.apply(this, args);
    cache[key] = result;

    return result;
  };
}
```

### Trie-Based Memoization

Useful when arguments may contain objects, arrays, or other values that should not be serialized.

```js
class MemoNode {
  constructor() {
    this.children = new Map();
    this.hasValue = false;
    this.value = undefined;
  }
}

function memoizeWithTrie(fn) {
  const root = new MemoNode();

  return function (...args) {
    let current = root;

    for (const arg of args) {
      if (!current.children.has(arg)) {
        current.children.set(arg, new MemoNode());
      }

      current = current.children.get(arg);
    }

    if (current.hasValue) {
      return current.value;
    }

    const result = fn.apply(this, args);

    current.hasValue = true;
    current.value = result;

    return result;
  };
}
```

---

## Object Utilities

### deepClone

Creates a recursive copy of arrays and plain objects.

```js
function deepClone(object) {
  if (object === null || typeof object !== "object") {
    return object;
  }

  if (Array.isArray(object)) {
    return object.map(deepClone);
  }

  const cloned = {};

  for (const key in object) {
    if (Object.prototype.hasOwnProperty.call(object, key)) {
      cloned[key] = deepClone(object[key]);
    }
  }

  return cloned;
}
```

> Note: This implementation does not clone `Date`, `Map`, `Set`,
