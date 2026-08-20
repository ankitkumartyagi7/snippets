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

> Note: This implementation does not clone `Date`, `Map`, `Set`, functions, circular references, or class instances.

### isEqual

Recursively compares objects and arrays.

```js
function isEqual(a, b) {
  if (Object.is(a, b)) {
    return true;
  }

  if (
    a === null ||
    b === null ||
    typeof a !== "object" ||
    typeof b !== "object"
  ) {
    return false;
  }

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) {
    return false;
  }

  for (const key of keysA) {
    if (
      !Object.prototype.hasOwnProperty.call(b, key) ||
      !isEqual(a[key], b[key])
    ) {
      return false;
    }
  }

  return true;
}
```

### deepCopyWithAppend

Deep-copies an object and appends the original array length to every copied array.

```js
function deepCopyWithAppend(value) {
  if (value === null || typeof value !== "object") {
    return value;
  }

  if (Array.isArray(value)) {
    const copy = value.map(deepCopyWithAppend);
    copy.push(value.length);
    return copy;
  }

  const copy = {};

  for (const key in value) {
    if (Object.prototype.hasOwnProperty.call(value, key)) {
      copy[key] = deepCopyWithAppend(value[key]);
    }
  }

  return copy;
}
```

Example:

```js
const original = {
  a:,[1][2]
  b: {
    c:[10][20][30]
  }
};

console.log(deepCopyWithAppend(original));

// {
//   a:,[1][2]
//   b: {
//     c:[3][10][20][30]
//   }
// }
```

### flattenObject

Converts nested objects into dot-separated keys.

```js
function flattenObject(object, parent = "", result = {}) {
  for (const key in object) {
    if (!Object.prototype.hasOwnProperty.call(object, key)) {
      continue;
    }

    const newKey = parent ? `${parent}.${key}` : key;
    const value = object[key];

    if (
      value !== null &&
      typeof value === "object" &&
      !Array.isArray(value)
    ) {
      flattenObject(value, newKey, result);
    } else {
      result[newKey] = value;
    }
  }

  return result;
}
```

Example:

```js
const object = {
  user: {
    name: "John",
    address: {
      city: "Delhi"
    }
  }
};

console.log(flattenObject(object));

// {
//   "user.name": "John",
//   "user.address.city": "Delhi"
// }
```

---

## Throttling and Rate Limiting

### Flag-Based Throttle

Allows one function execution during each waiting period.

```js
function throttleWithFlag(fn, delay) {
  let waiting = false;

  return async function (...args) {
    if (waiting) {
      return;
    }

    waiting = true;

    try {
      await fn.apply(this, args);
    } finally {
      setTimeout(() => {
        waiting = false;
      }, delay);
    }
  };
}
```

### Timestamp-Based Throttle

Uses timestamps to determine whether execution is allowed.

```js
function throttle(fn, delay) {
  let lastCall = 0;

  return function (...args) {
    const now = Date.now();

    if (now - lastCall >= delay) {
      lastCall = now;
      return fn.apply(this, args);
    }
  };
}
```

### Throttle with Call Limit

Allows a fixed number of calls within a time window.

```js
function throttleWithLimit(fn, maxCalls, timeframe) {
  const timestamps = [];

  return function (...args) {
    const now = Date.now();

    while (
      timestamps.length &&
      now - timestamps >= timeframe
    ) {
      timestamps.shift();
    }

    if (timestamps.length >= maxCalls) {
      console.warn("Rate limit exceeded");
      return;
    }

    timestamps.push(now);
    return fn.apply(this, args);
  };
}
```

Example:

```js
const limitedFn = throttleWithLimit(
  () => {
    console.log("API called", Date.now());
  },
  10,
  60_000
);

setInterval(limitedFn, 2_000);
```

### Sliding-Window Rate Limiter

Rejects calls after the configured limit is reached.

```js
function rateLimiter(fn, limit, windowSize) {
  let timestamps = [];

  return async function (...args) {
    const now = Date.now();

    timestamps = timestamps.filter(
      timestamp => now - timestamp < windowSize
    );

    if (timestamps.length >= limit) {
      throw new Error("Rate limit exceeded");
    }

    timestamps.push(now);
    return fn.apply(this, args);
  };
}
```

### Queue-Based Rate Limiter

Uses an index instead of repeatedly removing old timestamps.

```js
function queueRateLimiter(fn, limit, windowSize) {
  const timestamps = [];
  let start = 0;

  return async function (...args) {
    const now = Date.now();

    while (
      start < timestamps.length &&
      now - timestamps[start] >= windowSize
    ) {
      start++;
    }

    if (timestamps.length - start >= limit) {
      throw new Error("Rate limit exceeded");
    }

    timestamps.push(now);
    return fn.apply(this, args);
  };
}
```

### Token-Bucket Rate Limiter

Refills the available token count at a fixed interval.

```js
function tokenBucketRateLimiter(fn, maxTokens, refillInterval) {
  let tokens = maxTokens;

  setInterval(() => {
    tokens = maxTokens;
  }, refillInterval);

  return function (...args) {
    if (tokens === 0) {
      console.warn("No tokens available");
      return;
    }

    tokens--;
    return fn.apply(this, args);
  };
}
```

### Advanced Rate Limiter

Combines:

- Maximum calls per time window.
- Request queueing.
- Retries.
- Exponential backoff.

```js
function createRateLimiter({
  maxCalls,
  timeframe,
  retries = 3,
  backoff = 1_000
}) {
  let timestamps = [];
  const queue = [];
  let isProcessing = false;

  async function processQueue() {
    if (isProcessing) {
      return;
    }

    isProcessing = true;

    while (queue.length) {
      const now = Date.now();

      timestamps = timestamps.filter(
        timestamp => now - timestamp < timeframe
      );

      if (timestamps.length >= maxCalls) {
        const waitTime = timeframe - (now - timestamps);
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }

      const request = queue.shift();
      const { fn, resolve, reject, attempt } = request;

      try {
        timestamps.push(Date.now());
        const result = await fn();
        resolve(result);
      } catch (error) {
        if (attempt < retries) {
          setTimeout(() => {
            queue.push({
              fn,
              resolve,
              reject,
              attempt: attempt + 1
            });

            processQueue();
          }, backoff * 2 ** attempt);
        } else {
          reject(error);
        }
      }
    }

    isProcessing = false;
  }

  return function (...args) {
    return new Promise((resolve, reject) => {
      queue.push({
        fn: () => fn.apply(this, args),
        resolve,
        reject,
        attempt: 0
      });

      processQueue();
    });
  };
}
```

---

## Retry and Backoff

(Add your retry and backoff implementations here)

---

## Debouncing

### Basic Debounce

(Add your debounce implementations here)

### Debounce with Cancel

(Add your debounce implementations here)

### Debounce with Cancel and Flush

(Add your debounce implementations here)

---

## Currying

### Infinite Curry

(Add your curry implementations here)

### Variadic Curry

(Add your curry implementations here)

### Seed-Based Infinite Curry

(Add your curry implementations here)

---

## Other JavaScript Utilities

### Class Names Utility

(Add your utility implementations here)

### Custom setTimeout

(Add your utility implementations here)

### Pub/Sub

(Add your utility implementations here)

### Async Function Composition

(Add your utility implementations here)

### Pipeline

(Add your utility implementations here)

---

## React Examples

### Uncontrolled Input Wrapper

(Add your React examples here)

### useDebounce

(Add your React examples here)

### useThrottle

(Add your React examples here)

### useFetch

(Add your React examples here)

### useLocalStorage

(Add your React examples here)

### Custom useState

(Add your React examples here)

---

## React Router

### Role-Based Protected Routes

(Add your React Router examples here)

---

## Asynchronous Programming

### Task Scheduler

(Add your async programming examples here)

### URL Crawler

(Add your async programming examples here)

### mapAsyncLimit

(Add your async programming examples here)

### Sequential vs Parallel Execution

(Add your async programming examples here)

---

## Important Notes

(Add your important notes here)
