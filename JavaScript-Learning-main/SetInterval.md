# SetInterval()

- `setInterval()` used as `setTimeout()` [[SetTimeout]] function, but it could calls the function several times.
- While `setTimeout()` executes a function once after a delay, `setInterval()` executes a function **repeatedly** at a fixed interval.

## Purpose and Basic Concept

- `setInterval()` is used to execute a function or a piece of code repeatedly, with a fixed time delay between each execution. Like `setTimeout()`, it is non-blocking, meaning it schedules the function for repeated execution and immediately returns control to the calling code.

---

## Syntax

The basic syntax for `setInterval()` is:

```javascript
setInterval(function, delay, arg1, arg2, ...);
```

Or, if you're passing a string (generally discouraged):

```javascript
setInterval(codeString, delay);
```

###  Parameters 
- **`function` (or `codeString`)**:
    - **Required**. The function you want to execute repeatedly. Can be a named function, an anonymous function, or an arrow function.
    - **Discouraged**: Passing a string of code is generally bad practice for the same reasons as with `setTimeout()` (security, performance).
- **`delay` (optional)**:
    - **Optional**. The time, in **milliseconds**, that `setInterval()` should wait between each execution of the specified function.
    - If omitted, it defaults to `0`.
    - The actual interval might be slightly longer than specified due to the event loop and the time it takes for the function itself to execute.
- **`arg1, arg2, ...` (optional)**:
    - **Optional**. Additional arguments that will be passed directly to the `function` each time it is executed.

- It returns the interval ID(a positive integer).
- This ID is used to stop the repeated execution with `clearInterval()` function.

```JavaScript
let counter = 0;
let intervalId = setInterval(() => {
    counter++;
    console.log(`Count: ${counter}`);
}, 1000); // This will log "Count: X" every second

console.log(intervalId); // e.g., 1
```

- The previous code here will start counting from 1 until we use the following command to stop the interval `clearInterval(intervalId)`.

- To get the functionality of working check [[SetTimeout]] functionality.

## Common Use Cases

- **Live Clocks/Timers**: Displaying real-time updates.
- **Image Carousels/Slideshows**: Automatically advancing slides.
- **Polling**: Regularly checking a server for new data (though WebSockets or Server-Sent Events are often better for real-time updates).
- **Game Loops**: Basic game logic updates (for simple browser games).

`setInterval()` is a powerful tool, but it requires careful management with `clearInterval()` to prevent unintended behavior and resource consumption. For many use cases, chained `setTimeout()` or `requestAnimationFrame` might be more appropriate.
