"If an effect has dependencies and those dependencies change, the cleanup function for the *previous* effect run will execute *before* the new effect run. This ensures you always clean up old resources before setting up new ones."

Let's break down how to handle this situation, why it's designed this way, and illustrate it with a practical example.

### Why This Behavior is Crucial

Imagine you have a component that displays data from a specific user. This component might:
1.  **Fetch data** for `userId=1`.
2.  **Set up an event listener** for `userId=1` (e.g., to track their online status).

Now, what if the `userId` changes to `2`?
*   If React just ran the new effect for `userId=2` without cleaning up the old one, you'd still have an active event listener for `userId=1` running in the background. This is a **memory leak** and could lead to incorrect behavior (e.g., showing `userId=1` as online even if `userId=2` is the one currently displayed).
*   By running the cleanup *first*, React ensures that the resources associated with `userId=1` are properly released *before* it starts setting up new resources for `userId=2`. This keeps your application efficient, prevents bugs, and ensures that your component is always synchronized with its current props and state.

### How to Handle This Situation: The Cleanup Function

The way you handle this is by consistently providing a **cleanup function** whenever your `useEffect` sets up something that needs to be torn down. The `useEffect` Hook is designed to manage this lifecycle for you automatically.

Let's use an example of a component that simulates subscribing to a chat room. When the `roomId` changes, we need to unsubscribe from the old room and subscribe to the new one.

```jsx
import React, { useState, useEffect } from 'react';

// --- Mock Chat API (Imagine this is a real external library) ---
const ChatAPI = {
  subscribe: (roomId, callback) => {
    console.log(`[ChatAPI] Subscribing to room: ${roomId}`);
    // Simulate receiving messages
    const intervalId = setInterval(() => {
      callback(`Message from ${roomId}: Hello!`);
    }, 2000);
    return intervalId; // Return an ID to clear the interval later
  },
  unsubscribe: (roomId, intervalId) => {
    console.log(`[ChatAPI] Unsubscribing from room: ${roomId}`);
    clearInterval(intervalId);
  }
};
// -----------------------------------------------------------------

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    console.log(`[Effect Setup] Setting up chat for room: ${roomId}`);

    // 1. Set up the subscription (the "side effect")
    const subscriptionId = ChatAPI.subscribe(roomId, (message) => {
      setMessages(prevMessages => [...prevMessages, message]);
    });

    // 2. Return the cleanup function
    return () => {
      console.log(`[Effect Cleanup] Cleaning up chat for room: ${roomId}`);
      ChatAPI.unsubscribe(roomId, subscriptionId);
      setMessages([]); // Clear messages when changing rooms
    };
  }, [roomId]); // Dependency array: Re-run effect when roomId changes

  return (
    <div style={{ border: '1px solid gray', padding: '10px', margin: '10px' }}>
      <h3>Chat Room: {roomId}</h3>
      <div style={{ height: '100px', overflowY: 'scroll', border: '1px solid lightgray' }}>
        {messages.length === 0 ? (
          <p>No messages yet...</p>
        ) : (
          messages.map((msg, index) => <p key={index}>{msg}</p>)
        )}
      </div>
    </div>
  );
}

function App() {
  const [currentRoom, setCurrentRoom] = useState('general');

  return (
    <div>
      <h1>My Chat Application</h1>
      <button onClick={() => setCurrentRoom('general')}>Join General Room</button>
      <button onClick={() => setCurrentRoom('tech')}>Join Tech Room</button>
      <button onClick={() => setCurrentRoom('random')}>Join Random Room</button>

      <ChatRoom roomId={currentRoom} />
    </div>
  );
}

export default App;
```

### How it Works Step-by-Step:

Let's trace the execution when you switch rooms:

1.  **Initial Mount (e.g., `currentRoom` is 'general'):**
    *   `ChatRoom` component renders.
    *   `useEffect` runs for the first time.
    *   `console.log("[Effect Setup] Setting up chat for room: general")`
    *   `ChatAPI.subscribe('general', ...)` is called.
    *   The cleanup function `return () => { ... }` is registered by React.

2.  **User Clicks "Join Tech Room" (`currentRoom` changes to 'tech'):**
    *   `App` component re-renders, passing `roomId='tech'` to `ChatRoom`.
    *   `ChatRoom` component re-renders.
    *   React notices that `roomId` (a dependency) has changed from 'general' to 'tech'.
    *   **Crucially, React first executes the cleanup function from the *previous* effect run:**
        *   `console.log("[Effect Cleanup] Cleaning up chat for room: general")`
        *   `ChatAPI.unsubscribe('general', ...)` is called, stopping the old timer.
        *   `setMessages([])` clears the messages from the previous room.
    *   **Then, React executes the `setupFunction` for the *new* effect run:**
        *   `console.log("[Effect Setup] Setting up chat for room: tech")`
        *   `ChatAPI.subscribe('tech', ...)` is called, starting a new timer for the 'tech' room.
        *   A *new* cleanup function for `roomId='tech'` is registered.

3.  **Component Unmounts (e.g., `ChatRoom` is removed from the DOM):**
    *   React executes the cleanup function for the *last* active effect.
    *   `console.log("[Effect Cleanup] Cleaning up chat for room: tech")`
    *   `ChatAPI.unsubscribe('tech', ...)` is called, stopping the timer for the 'tech' room.

### Key Takeaways for Handling This:

*   **Always provide a cleanup function:** If your `useEffect` does anything that creates a resource (like a timer, an event listener, a subscription, or a network request that can be cancelled), make sure you return a function that cleans up that resource.
*   **The dependency array is paramount:** This array `[roomId]` tells React *when* to re-run your effect and, consequently, *when* to run the cleanup for the old effect. If `roomId` doesn't change, the effect (and its cleanup) won't re-run.
*   **Think of it as a "reset" mechanism:** When dependencies change, `useEffect` effectively "resets" the side effect: it cleans up the old one and sets up a brand new one with the updated values.

By understanding and consistently applying this pattern, you'll write more robust, efficient, and bug-free React applications. It's a fundamental concept for mastering `useEffect`!