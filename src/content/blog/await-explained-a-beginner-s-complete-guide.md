---
key: outputs/skills/4218ebea-da6f-46c3-ad0c-a268c2d0bd51/blog/javascript-async-await-explained/POST.md
indexed_at: 2026-07-05T11:21:04.903849+00:00
agent_id: 4218ebea-da6f-46c3-ad0c-a268c2d0bd51
author: 16
categories: ["tutorials"]
category: Web Development
coverImage: https://cdn-public.eesel.ai/b449f812-0061-4442-addb-cf11472cec0d/4218ebea-da6f-46c3-ad0c-a268c2d0bd51/66319f6e0ce64ef9a2c1b573767e3924.png
coverImageAlt: Flat illustration of a code editor with JavaScript async/await syntax
coverImageHeight: 1080
coverImageWidth: 1920
date: 2026-07-03T00:00:00.000Z
description: Learn JavaScript async/await from scratch. Understand promises, write clean async code, handle errors, and run requests in parallel - with real examples throughout.
excerpt: Learn JavaScript async/await from scratch. Understand promises, write clean async code, handle errors, and run requests in parallel - with real examples throughout.
faqs: {"heading": "Frequently Asked Questions", "type": "blog", "answerType": "html", "faqs": [{"question": "What is async/await in JavaScript?", "answer": "Async/await is a modern JavaScript syntax that makes asynchronous code look and behave like synchronous code. The <code>async</code> keyword marks a function as asynchronous, and <code>await</code> pauses execution inside that function until a Promise resolves. It was introduced in ES2017 and is now supported in all modern browsers. Learn more on <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function'>MDN Web Docs</a>."}, {"question": "Do I need to learn Promises before async/await?", "answer": "Yes - and this guide covers both. Async/await is built on top of Promises, so understanding what a Promise is (and why it exists) makes async/await click much faster. You don't need to master every Promise method, but knowing the basics of <code>.then()</code>, <code>.catch()</code>, and what 'pending,' 'fulfilled,' and 'rejected' mean will save you a lot of debugging time."}, {"question": "What happens if I forget to use await?", "answer": "You get a Promise object instead of the actual value. For example, <code>const data = fetchData()</code> gives you <code>Promise { &lt;pending&gt; }</code>, not the data you expected. This is one of the most common beginner mistakes. Always <code>await</code> async function calls when you need the resolved value, and use <code>try/catch</code> to handle any errors that come back."}, {"question": "When should I use Promise.all vs sequential await?", "answer": "Use <code>Promise.all()</code> when your operations are independent and can run at the same time - for example, fetching user data and product data simultaneously. Use sequential <code>await</code> when each step depends on the result of the previous one - for example, fetching a user ID first and then using that ID to fetch their orders. Running independent tasks sequentially wastes time; <code>Promise.all()</code> runs them in parallel."}, {"question": "Is async/await supported in all browsers?", "answer": "Yes. Async/await has been supported in all major browsers since 2017, including Chrome 55+, Firefox 52+, Safari 10.1+, and Edge 15+. For Node.js, it is available from version 7.6 onward. You can safely use async/await in any modern project without a transpiler, though older codebases using Babel may still compile it to Promise chains for broader compatibility."}], "supportLink": null}
is_primary_artifact: True
locale: en
readTime: 12 min
reviewer: 4
run_id: 54ca6faa-7d52-4f6c-8732-b1de7267a32a
seo: {"title": "JavaScript async/await explained: a beginner's complete guide (2026)", "description": "Learn JavaScript async/await from scratch. Understand promises, write clean async code, handle errors with try/catch, and run requests in parallel.", "image": "https://cdn-public.eesel.ai/b449f812-0061-4442-addb-cf11472cec0d/4218ebea-da6f-46c3-ad0c-a268c2d0bd51/66319f6e0ce64ef9a2c1b573767e3924.png"}
skill: blog
slug: javascript-async-await-explained
tags: ["javascript", "async-await", "promises", "web-development"]
task_id: 505c6f3b-9219-461f-909e-595e421e91a1
template: default
title: JavaScript async/await explained: a beginner's complete guide
updated: 2026-07-03
---
## TL;DR

JavaScript runs one thing at a time. When you need to fetch data from an API, read a file, or do anything that takes time, you need asynchronous code - code that can wait without freezing everything else. Async/await is the modern, readable way to write it. You mark a function with `async`, use `await` before any operation that returns a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises), and JavaScript handles the waiting. Wrap it in `try/catch` for error handling, and use `Promise.all()` when you want multiple things running at the same time. By the end of this guide, you will understand why async/await exists, how to use it, and the five mistakes most beginners make.

## Why JavaScript needs async code

JavaScript is single-threaded. One script, one thread, one task at a time.

That is fine for most things. But the moment your code needs to wait for something outside the browser - a network request, a database query, a file read - you have a problem. If JavaScript waited synchronously, the whole page would freeze. Nothing would respond. You could not scroll, click, or type.

<mark>Asynchronous code solves this by saying: "Start this task, keep doing other things, and come back when it is done."</mark>

The question is how you write that in a clean, readable way. JavaScript has had three answers over the years - and they get progressively better.

![The evolution from callbacks to Promises to async/await - each step cleaner than the last](https://cdn-public.eesel.ai/b449f812-0061-4442-addb-cf11472cec0d/4218ebea-da6f-46c3-ad0c-a268c2d0bd51/06ebf2777c294894949a408aed341462.png)

## The old way: callbacks

Before Promises existed, JavaScript used callbacks. A callback is just a function you pass to another function, to be called when the work is done.

Here is a simple example:

```javascript
function fetchUser(userId, callback) {
  setTimeout(() => {
    callback({ id: userId, name: "Alice" });
  }, 1000);
}

fetchUser(1, (user) => {
  console.log(user.name); // "Alice"
});
```

This works. But it falls apart quickly when operations depend on each other. If you need to fetch a user, then fetch their orders, then fetch the details of each order, you end up with deeply nested callbacks:

```javascript
fetchUser(1, (user) => {
  fetchOrders(user.id, (orders) => {
    fetchOrderDetails(orders[0].id, (details) => {
      fetchProductInfo(details.productId, (product) => {
        console.log(product.name);
        // This is "callback hell" - also called the pyramid of doom
      });
    });
  });
});
```

Every level of nesting adds complexity. Error handling becomes a nightmare. Code that used to be readable is now a wall of indented closures.

Promises were invented to fix this.

## Promises: the foundation you need to understand

A [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) is an object that represents the eventual result of an asynchronous operation. When you call a modern async function - like [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) - it returns a Promise immediately. You get that Promise right away, even though the actual data has not arrived yet.

A Promise can be in one of three states:

- **Pending** - the operation is still in progress
- **Fulfilled** - the operation succeeded, and the Promise has a value
- **Rejected** - the operation failed, and the Promise has an error

![Promise state diagram: pending transitions to either fulfilled or rejected](https://cdn-public.eesel.ai/b449f812-0061-4442-addb-cf11472cec0d/4218ebea-da6f-46c3-ad0c-a268c2d0bd51/dc0af3cbf107457eb1962c0bdc80c85d.png)

You handle those outcomes with `.then()` and `.catch()`:

```javascript
fetch("https://api.example.com/users/1")
  .then((response) => response.json())
  .then((user) => {
    console.log(user.name);
  })
  .catch((error) => {
    console.error("Something went wrong:", error);
  });
```

This is already much better than callbacks. The code reads left-to-right and top-to-bottom. Errors are handled in one place at the end.

But there is still a problem: chaining `.then()` after `.then()` after `.then()` gets unwieldy when you have complex logic. And mixing `if` statements into Promise chains is genuinely awkward.

That is where async/await comes in.

## async/await: the readable way to write async code

[Async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) was introduced in [ES2017](https://262.ecma-international.org/8.0/) and has been supported in all major browsers since mid-2017. It is not a replacement for Promises - it is built on top of them. It just gives you a much cleaner syntax.

### The async keyword

Add `async` before a function declaration, and it does two things:

1. It always returns a Promise, even if you return a plain value
2. It lets you use `await` inside it

```javascript
async function greet() {
  return "Hello!";
}

greet().then(console.log); // "Hello!"
```

That function returns a resolved Promise - not the string directly. Under the hood, `async` wraps your return value in `Promise.resolve()`.

### The await keyword

`await` pauses execution of the async function until the Promise it is waiting on settles. Then it unwraps the resolved value and gives it to you directly.

```javascript
async function getUser() {
  const response = await fetch("https://api.example.com/users/1");
  const user = await response.json();
  console.log(user.name); // actual user object, not a Promise
}
```

Compare that to the `.then()` chain from earlier. Same result. Far more readable.

You can use `if`, `for`, `while`, and any other standard JavaScript logic around `await` calls without any special handling. This is the real win.

### A practical real-world example

Here is what a complete async function looks like - fetching data from an API, parsing it, and doing something with the result:

```javascript
async function loadUserProfile(userId) {
  const response = await fetch(`https://api.example.com/users/${userId}`);

  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }

  const user = await response.json();
  return user;
}

loadUserProfile(1).then((user) => console.log(user.name));
```

Clean. Top-to-bottom. Reads almost like synchronous code.

## Error handling with try/catch

Every async function can fail. Network requests time out. APIs return errors. Data comes back in unexpected shapes.

With `.then()` chains, you add a `.catch()` at the end. With async/await, you use `try/catch` - the same pattern JavaScript uses everywhere else for error handling.

```javascript
async function loadUserProfile(userId) {
  try {
    const response = await fetch(`https://api.example.com/users/${userId}`);

    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }

    const user = await response.json();
    return user;
  } catch (error) {
    console.error("Failed to load user:", error.message);
    return null;
  }
}
```

The `try` block runs your async operations. If any `await` throws - or if you throw manually - the `catch` block catches it. One `try/catch` can wrap as many `await` calls as you need.

One thing to know: if you forget `try/catch`, the error propagates as a rejected Promise. The caller can catch it with `.catch()`:

```javascript
loadUserProfile(1)
  .then((user) => console.log(user))
  .catch((error) => console.error("Caught:", error));
```

But you are better off handling it inside the async function itself. That way the function always returns something predictable - either a value, or `null`, or a specific fallback.

## Running multiple requests at once with Promise.all

Here is a trap many beginners fall into. They need to fetch several things, so they write this:

```javascript
async function loadDashboard() {
  const user = await fetchUser();     // waits ~500ms
  const posts = await fetchPosts();   // waits ~500ms
  const stats = await fetchStats();   // waits ~500ms
  // Total: ~1500ms
}
```

Each `await` blocks until the previous one finishes. But `fetchUser`, `fetchPosts`, and `fetchStats` do not depend on each other. You are waiting 1.5 seconds when you only needed 500ms.

`Promise.all()` fixes this by running Promises in parallel:

```javascript
async function loadDashboard() {
  const [user, posts, stats] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchStats()
  ]);
  // Total: ~500ms (all three ran at once)
}
```

![Serial vs parallel execution: running tasks with Promise.all is significantly faster](https://cdn-public.eesel.ai/b449f812-0061-4442-addb-cf11472cec0d/4218ebea-da6f-46c3-ad0c-a268c2d0bd51/f3a1c91af0cb4d74b6916bb2957efe14.png)

`Promise.all()` takes an array of Promises and returns a single Promise that fulfills when all of them fulfill. The result is an array of values, in the same order as the input.

One important behavior: if any Promise in the array rejects, `Promise.all()` rejects immediately. The other Promises keep running in the background, but you get the error right away.

If you need all results even when some fail, use `Promise.allSettled()` instead:

```javascript
const results = await Promise.allSettled([
  fetchUser(),
  fetchPosts(),
  fetchStats()
]);

results.forEach((result) => {
  if (result.status === "fulfilled") {
    console.log(result.value);
  } else {
    console.error(result.reason);
  }
});
```

`Promise.allSettled()` waits for every Promise to finish, regardless of success or failure. Each item in the results array has a `status` of either `"fulfilled"` or `"rejected"`.

Use `Promise.all()` when everything needs to succeed. Use `Promise.allSettled()` when you want to process all results even if some fail.

## The 5 mistakes every beginner makes

These are the patterns I see trip people up most often.

### 1. Forgetting await

```javascript
// Wrong
async function getUser() {
  const user = fetchUser(); // returns a Promise, not the data
  console.log(user);        // logs Promise { <pending> }
}

// Right
async function getUser() {
  const user = await fetchUser(); // waits and unwraps the value
  console.log(user);              // logs the actual user object
}
```

If you see `Promise { <pending> }` in your console, this is almost always why.

### 2. Using await outside an async function

`await` only works inside `async` functions. In older environments, using it at the top level of a script throws a syntax error.

```javascript
// Wrong (top-level await requires ES modules or Node.js v14.8+)
const data = await fetchData();

// Works in any environment
(async () => {
  const data = await fetchData();
  console.log(data);
})();
```

Modern Node.js projects with ES modules and browsers support top-level `await`. But if you are writing code for a broader environment, wrap your top-level async logic in an async IIFE (immediately invoked function expression).

### 3. Running independent tasks in series

The example from the previous section. When tasks do not depend on each other, run them with `Promise.all()` instead of sequential `await`.

### 4. Not handling errors

```javascript
// Dangerous - unhandled Promise rejection
async function loadData() {
  const response = await fetch("/api/data"); // could fail
  return response.json();
}
```

If `fetch()` fails, this throws an unhandled rejection. In modern Node.js, unhandled rejections crash the process. In the browser, they log a warning and can cause silent failures.

Always wrap async operations in `try/catch`, or at minimum chain `.catch()` on every async function call.

### 5. Awaiting things that are not Promises

```javascript
// Pointless
const value = await 42; // still just 42

// Also pointless
async function add(a, b) {
  return await a + b; // no reason to await here
}
```

`await` on a non-Promise value is valid but does nothing useful. Only `await` when you are actually waiting for a Promise to resolve. Adding unnecessary `async` and `await` does not break anything, but it adds noise and misleads readers.

## Try DevAI Tutorials

DevAI Tutorials covers JavaScript, HTML, CSS, and AI tools in plain language - no jargon, no unnecessary theory. If you are learning to code or building your first projects, the site has step-by-step guides that take you from zero to something you actually built.

Check it out at [ai-tools-blog-blush.vercel.app](https://ai-tools-blog-blush.vercel.app).

> "Every post is practical. Every tutorial ends with something you actually built or used." - [DevAI Tutorials - About](https://ai-tools-blog-blush.vercel.app/about)