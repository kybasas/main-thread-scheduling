<br>
<br>
<div align="center">
<img width="288px" src="media/logo-centered.png">
</div>

<h3 align="center">
<b>main-thread-scheduling</b>
</h3>
<p align="center">
Consistently responsive apps while staying on the main thread
</p>

<p align="center">
<a href="https://www.travis-ci.com/astoilkov/main-thread-scheduling">
<img src="https://www.travis-ci.com/astoilkov/main-thread-scheduling.svg?branch=master" alt="Build Status" />
</a>
<a href="https://codeclimate.com/github/astoilkov/main-thread-scheduling/test_coverage">
<img src="https://img.shields.io/codeclimate/coverage/astoilkov/main-thread-scheduling" alt="Test Coverage" />
</a>
<a href="https://bundlephobia.com/result?p=use-local-storage-state">
<img src="https://badgen.net/bundlephobia/min/main-thread-scheduling" alt="Test Coverage" />
</a>
<!-- [![Minified Size](https://img.shields.io/npm/dm/main-thread-scheduling)](https://www.npmjs.com/package/use-local-storage-state) -->
<p>

<br>

## Install

```shell
npm install main-thread-scheduling
```

## Overview

The library ensures that:
- the UI doesn't freeze
- the user's computer fan doesn't spin
- it can be easily integrated in an existing code base

This is accomplished through multiple strategies:
- Stops task execution when user interacts with the UI. Uses `navigator.scheduling.isInputPending()` and fallbacks to [IdleDeadline](https://developer.mozilla.org/en-US/docs/Web/API/IdleDeadline).
- Global queue. Multiple tasks are executed one by one so increasing the number of tasks doesn't degrade performance linearly.
- Sorts tasks by importance. Sorts by [priority](#priorities) and gives priority to tasks requested later.
- Urgent UI changes are given highest priority possible. Tasks with `user-visible` priority are optimized to deliver smooth UX.
- Considerate about your existing code. Tasks with `background` priority are executed last so there isn't some unexpected work that slows down the main thread after the background task is finished.

## Use Cases

- You want to turn a synchronous function into a non-blocking asynchronous function. **Avoids UI freezes.**
- You want to render important elements first and less urgent ones second. **Improves perceived performance.**
- You want to run a long background task that doesn't spin the fans after a while. **Avoids bad reputation.**
- You want to run multiple backgrounds tasks that don't degrade your app performance with time. **Prevents death by a thousand cuts.**

## Why

Why rely on some open-source library to ensure a good performance for my app?
- Not a weekend project. I have been working on this code for months. If you want to dive deeper, you can read the [in-depth](./docs/in-depth.md) doc.
- This is the future. Browsers are probably going to support scheduling tasks on the main thread in the future. Here is the [spec](https://github.com/WICG/scheduling-apis).
- Simple. 90% of the time you only need `yieldOrContinue(priority)` function. The API has two more functions for more advanced cases.
- Aiming for high-quality with [my open-source principles](https://astoilkov.com/my-open-source-principles).

## Example

You can see the library in action in [this CodeSandbox](https://codesandbox.io/s/main-thread-scheduling-example-qqef6?file=/src/App.js:1188-1361). Try to remove the call to `yieldToContinue()` and then type in the input to see the difference.

## API

Note: If you want to understand how this library works under the hook and some of the details – read the [in-depth](./docs/in-depth.md) doc.

### `yieldOrContinue(priority: 'background' | 'user-visible')`

The complexity of the entire library is hidden behind this method. You can have great app performance by calling a single method.

```ts
async function findInFiles(query: string) {  
    for (const file of files) {
        await yieldOrContinue('user-visible')
        
        for (const line of file.lines) {
            fuzzySearchLine(line, query)
        }
    }
}
```

### More complex scenarios

The library has two more functions available:
- `yieldControl(priority: 'background' | 'user-visible')`
- `isTimeToYield(priority: 'background' | 'user-visible')`

These two functions are used together to handle more advanced use cases.

A simple use case where you will need those two functions is when you want to render your view before yielding back control to the browser to continue its work:
```ts
async function doHeavyWork() {
    for (const value of values) {
        if (isTimeToYield('user-visible')) {
            render()
            await yieldControl('user-visible')
        }
        
        computeHeavyWorkOnValue(value)
    }
}
```

### Priorities

Currently there are only two priorities available: `background` and `user-visible`:
- `background` – use this for background tasks. Every background task is run for 5ms.
- `user-visible` – use this for things that need to display to the user as fast as possible. Every `user-visible` task is run for 83ms – this gives you a nice cycle of doing heavy work and letting the browser render pending changes.

If you have a use case for a third priority, you can write in [this issue](https://github.com/astoilkov/main-thread-scheduling/issues/1).

## Alternatives

The problem this library solves isn't new. However, I haven't found a library that can solve this problem in a simple manner. [Open an issue](https://github.com/astoilkov/main-thread-scheduling/issues/new) if there is such a library so I can add it here.

Web Workers are a possible alternative. However, in reality, it's rare to see people using them. That's because they require significant investment of time due to the complexity that can't be avoided when working with CPU threads regardless of the programming language.

React has an implementation for scheduling tasks – [react/scheduler](https://github.com/facebook/react/tree/3c7d52c3d6d316d09d5c2479c6851acecccc6325/packages/scheduler). They plan to make it more generic but there doesn't seem to be a public roadmap for that.
