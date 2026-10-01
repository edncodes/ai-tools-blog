---
title: "JavaScript promises explained: the concept that confused me for months"
description: "I could write async/await code that worked without understanding why. Here's what finally made promises click — including the methods nobody explains well."
date: 2026-10-01
tags: ["javascript", "async", "beginners"]
category: "Web Development"
readTime: "10 min read"
---

I could write `async/await` code that worked. I just didn't understand why it worked. I'd copy a pattern, it would run, and I'd move on without really knowing what was happening underneath. That worked fine until something broke in a way I didn't expect, and I had no idea how to even start debugging it.

What actually fixed this wasn't a better explanation of syntax. It was understanding what a promise actually is.

## Start with a receipt, not a definition

Order food at a restaurant and the waiter doesn't stand at your table waiting for the kitchen. They hand you a receipt and walk off. That receipt isn't the food — it's a promise that food is coming. Eventually one of three things happens: it arrives, the kitchen runs out and cancels your order, or you're still sitting there waiting.

A JavaScript promise is that receipt. It's a placeholder for a value that doesn't exist yet, but will — or won't — at some point in the future.

## Why this exists at all

Before promises, asynchronous code in JavaScript ran on callbacks: functions you pass into other functions to run later. That works for one async step. It gets ugly fast once you need several in a row, because each one nests inside the last. People called this callback hell, and it earned the name — six levels of indentation and error handling scattered across every level.

Promises gave async code one consistent shape. Do this, then do that, and handle any failure in one place instead of ten.

## The three states

A promise is always in exactly one of these:

- **Pending** — still waiting
- **Fulfilled** — it succeeded, and now holds a value
- **Rejected** — it failed, and now holds a reason why

It moves pending → fulfilled, or pending → rejected. Never backward, never both. Once it settles, that's final.

## A basic example

```javascript
const orderFood = new Promise((resolve, reject) => {
  const kitchenHasIngredients = true;

  setTimeout(() => {
    if (kitchenHasIngredients) {
      resolve("Your food is ready!");
    } else {
      reject("Sorry, we're out of that dish.");
    }
  }, 2000);
});

orderFood
  .then((result) => console.log(result))
  .catch((error) => console.log(error));
```

`resolve` and `reject` are the two doors out of "pending." `.then()` runs if it walks through the resolve door. `.catch()` runs if it walks through the other one.

## The misconception that trips up almost everyone

A promise does not run code in parallel, in the background, on a separate thread. JavaScript is single-threaded — there is no second track running your code at the same time.

What a promise actually does is defer handling the result until later, while the rest of your script keeps going. The actual waiting happens somewhere outside your JavaScript — in the browser's network layer, in the OS, on a server somewhere. The promise is just how JavaScript gets notified when that outside thing finishes.

This matters because it explains why promises don't make your code faster. They don't. They just stop it from blocking while something slow happens elsewhere.

## A real example, not a toy one

`setTimeout` is fine for learning, but nobody writes that in real code. Here's what you'll actually type — fetching data from an API:

```javascript
fetch("https://api.example.com/users/1")
  .then((response) => response.json())
  .then((user) => {
    console.log(user.name);
  })
  .catch((error) => {
    console.log("Could not load user:", error);
  });
```

Two `.then()` calls here, not one. The first turns the raw response into usable JSON — that's also a promise, which is why it needs its own `.then()`. The second actually does something with the data. This two-step shape trips people up the first time they see it because it looks redundant. It isn't — `response.json()` is itself asynchronous.

## Chaining more than two steps

```javascript
fetchUser(userId)
  .then((user) => fetchOrders(user.id))
  .then((orders) => fetchOrderDetails(orders[0].id))
  .then((details) => console.log(details))
  .catch((error) => console.log("Something failed:", error));
```

Each step waits for the one before it. One `.catch()` at the end catches a failure from any step in the chain — you don't need to repeat error handling at every link.

## The methods nobody explains well

Once you're comfortable with a single promise, you'll eventually need to run several at once. This is the part most beginner guides skip, and it's where I got stuck for a while because I didn't know these existed.

**`Promise.all()`** — runs promises in parallel, waits for all of them, and fails immediately if any single one rejects.

```javascript
const [user, posts, comments] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id),
]);
```

Use this when you need everything to succeed or none of it matters — like loading a dashboard that needs all three pieces to render correctly.

**`Promise.allSettled()`** — also runs them in parallel, but waits for all of them regardless of failures, and gives you a result for each one marked either `fulfilled` or `rejected`.

```javascript
const results = await Promise.allSettled([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id),
]);

results.forEach((result) => {
  if (result.status === "fulfilled") {
    console.log(result.value);
  } else {
    console.log("One failed:", result.reason);
  }
});
```

Use this when partial success is fine — like loading three widgets on a page where one failing shouldn't take down the other two.

**`Promise.race()`** — settles as soon as the first promise settles, fulfilled or rejected, and ignores the rest.

A common use: a timeout. Race your actual request against a promise that rejects after a few seconds, and whichever finishes first wins.

**`Promise.any()`** — settles as soon as the first one fulfills, and only rejects if every single one fails. The opposite priority from `race()`.

I didn't touch any of these for months because every tutorial I found stopped at `.then()` and basic chaining. If you only remember one thing from this section: reach for `Promise.all()` when you need everything, and `Promise.allSettled()` when some failures are acceptable.

## How this connects to async/await

`async/await` isn't a different thing from promises. It's syntax sitting directly on top of them, built to make promise code read like it runs top to bottom:

```javascript
async function getOrderDetails(userId) {
  try {
    const user = await fetchUser(userId);
    const orders = await fetchOrders(user.id);
    const details = await fetchOrderDetails(orders[0].id);
    console.log(details);
  } catch (error) {
    console.log("Something failed:", error);
  }
}
```

Same chain as before. `await` pauses execution inside that function until the promise resolves. `try/catch` takes the place of `.catch()`. Underneath, it's still a promise — `await` is just a nicer way to read one.

## Mistakes I actually made

**Forgetting to return a promise inside a chain.** If you don't `return` the next call inside a `.then()`, the chain doesn't wait for it. The bug that shows up isn't an error — it's your code running in the wrong order, which is much harder to notice.

**Not handling rejections at all.** An unhandled rejected promise fails silently in some places and crashes the process in others. Pair every `.then()` with a `.catch()`, or wrap every `await` in `try/catch`. No exceptions to this one.

**Mixing `.then()` and `await` in the same function.** Both work, but switching between them mid-function makes code genuinely harder to follow. Pick one style per function and stay with it.

**Using `Promise.all()` when I actually needed `allSettled()`.** I had a page that loaded three independent sections, and I wrote it with `Promise.all()`. One section's API would occasionally fail, and because of how `all()` works, the entire page would show an error instead of just the one broken section. Switching to `allSettled()` fixed it in one line.

## Test yourself

Before moving to the next thing — can you explain, in your own words, the difference between `Promise.all()` and `Promise.allSettled()`, and when you'd reach for one over the other? If you can answer that without looking back up, this has actually stuck.
