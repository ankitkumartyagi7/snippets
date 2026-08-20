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

Implements retry logic with exponential backoff for failed operations.

```js
async function retryWithBackoff(fn, {
  retries = 3,
  backoff = 1_000,
  maxBackoff = 10_000
} = {}) {
  let attempt = 0;

  while (attempt <= retries) {
    try {
      return await fn();
    } catch (error) {
      attempt++;

      if (attempt > retries) {
        throw error;
      }

      const delay = Math.min(backoff * 2 ** (attempt - 1), maxBackoff);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

Usage:

```js
const result = await retryWithBackoff(
  () => fetch('https://api.example.com/data'),
  { retries: 3, backoff: 1_000 }
);
```

---

## Debouncing

### Basic Debounce

Delays function execution until after a specified wait time has passed since the last call.

```js
function debounce(fn, delay) {
  let timeoutId;

  return function (...args) {
    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

### Debounce with Cancel

Adds a cancel method to clear the pending timeout.

```js
function debounceWithCancel(fn, delay) {
  let timeoutId;

  const debounced = function (...args) {
    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };

  debounced.cancel = function () {
    clearTimeout(timeoutId);
  };

  return debounced;
}
```

### Debounce with Cancel and Flush

Adds both cancel and immediate execution (flush) capabilities.

```js
function debounceWithCancelAndFlush(fn, delay) {
  let timeoutId;

  const debounced = function (...args) {
    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };

  debounced.cancel = function () {
    clearTimeout(timeoutId);
  };

  debounced.flush = function (...args) {
    clearTimeout(timeoutId);
    return fn.apply(this, args);
  };

  return debounced;
}
```

---

## Currying

### Infinite Curry

Allows unlimited chained function calls.

```js
function infiniteCurry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }

    return function (...moreArgs) {
      return curried(...args, ...moreArgs);
    };
  };
}
```

Usage:

```js
function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = infiniteCurry(add);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
```

### Variadic Curry

Handles functions with variable arguments.

```js
function variadicCurry(fn) {
  return function curried(...args) {
    return function (...moreArgs) {
      const allArgs = [...args, ...moreArgs];

      if (allArgs.length >= fn.length) {
        return fn(...allArgs);
      }

      return curried(...allArgs);
    };
  };
}
```

### Seed-Based Infinite Curry

Accumulates values with a seed until explicitly called.

```js
function seedBasedInfiniteCurry(seed = 0) {
  const curry = function (value) {
    if (value === undefined) {
      return seed;
    }

    return seedBasedInfiniteCurry(seed + value);
  };

  curry.valueOf = () => seed;
  curry.toString = () => seed;

  return curry;
}
```

Usage:

```js
const add = seedBasedInfiniteCurry();

console.log(add(1)(2)(3)()); // 6
console.log(+add(1)(2)(3)); // 6
```

---

## Other JavaScript Utilities

### Class Names Utility

Conditionally joins class names based on truthy values.

```js
function classNames(...args) {
  return args
    .flatMap(arg => {
      if (typeof arg === 'string' || typeof arg === 'number') {
        return arg;
      }

      if (Array.isArray(arg)) {
        return classNames(...arg);
      }

      if (typeof arg === 'object' && arg !== null) {
        return Object.keys(arg).filter(key => arg[key]);
      }

      return [];
    })
    .filter(Boolean)
    .join(' ');
}
```

Usage:

```js
classNames('btn', 'btn-primary', { disabled: true, active: false });
// "btn btn-primary disabled"
```

### Custom setTimeout

A promise-based setTimeout wrapper.

```js
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

Usage:

```js
await sleep(1000);
console.log('Executed after 1 second');
```

### Pub/Sub

Publish-Subscribe pattern implementation.

```js
class PubSub {
  constructor() {
    this.events = {};
  }

  subscribe(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }

    this.events[event].push(callback);

    return () => this.unsubscribe(event, callback);
  }

  unsubscribe(event, callback) {
    if (!this.events[event]) {
      return;
    }

    this.events[event] = this.events[event].filter(
      cb => cb !== callback
    );
  }

  publish(event, data) {
    if (!this.events[event]) {
      return;
    }

    this.events[event].forEach(callback => callback(data));
  }
}
```

Usage:

```js
const pubsub = new PubSub();

const unsubscribe = pubsub.subscribe('message', (data) => {
  console.log('Received:', data);
});

pubsub.publish('message', 'Hello!');
unsubscribe();
```

### Async Function Composition

Composes multiple async functions.

```js
function composeAsync(...fns) {
  return async function (input) {
    let result = input;

    for (const fn of fns) {
      result = await fn(result);
    }

    return result;
  };
}
```

Usage:

```js
const addOne = async x => x + 1;
const double = async x => x * 2;

const pipeline = composeAsync(addOne, double);

console.log(await pipeline(5)); // 12
```

### Pipeline

Left-to-right function composition.

```js
function pipe(...fns) {
  return function (input) {
    return fns.reduce((acc, fn) => fn(acc), input);
  };
}
```

Usage:

```js
const addOne = x => x + 1;
const double = x => x * 2;

const pipeline = pipe(addOne, double);

console.log(pipeline(5)); // 12
```

---

## React Examples

### Uncontrolled Input Wrapper

A reusable wrapper for uncontrolled inputs.

```jsx
import { useRef, useEffect } from 'react';

function UncontrolledInput({ defaultValue, onChange, ...props }) {
  const inputRef = useRef(null);

  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.value = defaultValue;
    }
  }, [defaultValue]);

  return (
    <input
      ref={inputRef}
      onChange={e => onChange?.(e.target.value)}
      {...props}
    />
  );
}
```

### useDebounce

Custom hook for debouncing values.

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timeoutId);
  }, [value, delay]);

  return debouncedValue;
}
```

Usage:

```jsx
const searchValue = useDebounce(inputValue, 300);
```

### useThrottle

Custom hook for throttling values.

```jsx
import { useState, useEffect, useRef } from 'react';

function useThrottle(value, interval) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastUpdated = useRef(0);

  useEffect(() => {
    const now = Date.now();

    if (now - lastUpdated.current >= interval) {
      lastUpdated.current = now;
      setThrottledValue(value);
    } else {
      const timeoutId = setTimeout(() => {
        lastUpdated.current = Date.now();
        setThrottledValue(value);
      }, interval - (now - lastUpdated.current));

      return () => clearTimeout(timeoutId);
    }
  }, [value, interval]);

  return throttledValue;
}
```

### useFetch

Custom hook for data fetching.

```jsx
import { useState, useEffect } from 'react';

function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    const fetchData = async () => {
      try {
        const response = await fetch(url, {
          ...options,
          signal: controller.signal
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
```

### useLocalStorage

Custom hook for persisting state in localStorage.

```jsx
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}
```

### Custom useState

A simplified useState implementation for understanding.

```jsx
let state = [];
let currentStateIndex = 0;

function customUseState(initialValue) {
  const currentIndex = currentStateIndex;

  if (state[currentIndex] === undefined) {
    state[currentIndex] = initialValue;
  }

  const setState = (newValue) => {
    state[currentIndex] = newValue;
    render();
  };

  currentStateIndex++;

  return [state[currentIndex], setState];
}

function render() {
  currentStateIndex = 0;
  // Re-render component
}
```

---

## React Router

### Role-Based Protected Routes

Protects routes based on user roles.

```jsx
import { Navigate, Outlet } from 'react-router-dom';

function ProtectedRoute({ allowedRoles, user }) {
  if (!user) {
    return <Navigate to="/login" replace />;
  }

  if (!allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return <Outlet />;
}
```

Usage:

```jsx
<Routes>
  <Route element={<ProtectedRoute allowedRoles={['admin']} user={user} />}>
    <Route path="/admin" element={<AdminDashboard />} />
  </Route>
</Routes>
```

---

## Asynchronous Programming

### Task Scheduler

Limits concurrent task execution.

```js
class TaskScheduler {
  constructor(maxConcurrent) {
    this.maxConcurrent = maxConcurrent;
    this.running = 0;
    this.queue = [];
  }

  async add(task) {
    if (this.running >= this.maxConcurrent) {
      await new Promise(resolve => this.queue.push(resolve));
    }

    this.running++;

    try {
      return await task();
    } finally {
      this.running--;

      if (this.queue.length) {
        const next = this.queue.shift();
        next();
      }
    }
  }
}
```

Usage:

```js
const scheduler = new TaskScheduler(3);

scheduler.add(() => fetch('https://api.example.com/1'));
scheduler.add(() => fetch('https://api.example.com/2'));
```

### URL Crawler

Crawls URLs with concurrency control.

```js
async function crawlUrls(urls, maxConcurrent = 5) {
  const scheduler = new TaskScheduler(maxConcurrent);
  const results = [];

  const tasks = urls.map(url =>
    scheduler.add(async () => {
      const response = await fetch(url);
      const data = await response.json();
      results.push({ url, data });
    })
  );

  await Promise.all(tasks);

  return results;
}
```

### mapAsyncLimit

Maps over array with concurrency limit.

```js
async function mapAsyncLimit(array, limit, asyncFn) {
  const results = [];
  const scheduler = new TaskScheduler(limit);

  const promises = array.map((item, index) =>
    scheduler.add(async () => {
      results[index] = await asyncFn(item, index);
    })
  );

  await Promise.all(promises);

  return results;
}
```

Usage:

```js
const results = await mapAsyncLimit(
 ,[1][2][3][4][5]
  2,
  async (num) => {
    await sleep(1000);
    return num * 2;
  }
);
```

### Sequential vs Parallel Execution

```js
// Sequential execution
async function runSequentially(tasks) {
  const results = [];

  for (const task of tasks) {
    results.push(await task());
  }

  return results;
}

// Parallel execution
async function runParallel(tasks) {
  return Promise.all(tasks.map(task => task()));
}

// Parallel with limit
async function runParallelWithLimit(tasks, limit) {
  return mapAsyncLimit(tasks, limit, task => task());
}
```

---

## Important Notes

1. **Polyfills**: These implementations are for learning purposes. In production, use native methods or well-tested libraries.

2. **Rate Limiting**: Choose the right strategy based on your use case:
   - **Throttle**: For UI events (scroll, resize)
   - **Debounce**: For search inputs, form validation
   - **Rate Limiter**: For API calls

3. **Memoization**: JSON-based is simpler but doesn't handle circular references. Trie-based is more flexible but uses more memory.

4. **Async Patterns**: Always handle errors in async operations and clean up resources (abort controllers, timeouts).

5. **React Hooks**: Follow the rules of hooks - only call at the top level and in React functions.

6. **Performance**: Consider time and space complexity when choosing implementations for interviews.
