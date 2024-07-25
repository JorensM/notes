# How to sync React state across tabs with workers

Hello, in this article I will show you how you can sync state in React across multiple tabs. We will be using the SharedWorker API to achieve this. If you want to skip the tutorial and just use it, I have an npm package [react-tab-state](https://www.npmjs.com/package/@printy/react-tab-state)

The tutorial is written in TypeScript (aside from the worker), but you can just as well use JavaScript for this.

So without further ado, let's get started!

## Project structure

I will assume that you already know how to set up a React project. You can use CRA or Vite or even NextJS, just make sure that the environment supports shared workers(it probably does)!

For this project we will have 3 main files -

 * `index.tsx`, where our page code will reside
 * `useTabState.ts`, where the code for our tab- synced-state hook will reside
 * `worker.js`, where the code for our shared worker will reside

## worker.js

First of all let's write the code of our worker.

First let's add some variables

```
let ports = []; // All of our ports
let latest_state = {} // The currently synced states.
```

So first we have `ports` which stores all of open ports. There is a single port for each tab. A port is basically a connection between a tab and the shared worker, which allows you to send and receive messages. We need this array so we can send a single message to all ports at once

Next, `latest_state` stores the most recent synced state values in an object. Each key in the object will be a unique id for that particular state. This is so we can have multiple states. We will need this variable in order to sync the state of a newly opened tab

Now let's add a function to post a message to all ports

```
// Post message to all connected ports
const postMessageAll = (msg, excluded_port = null) => {
    ports.forEach( port => {
        // Don't post message to the excluded port, if one has been specified
        if(port == excluded_port){
              return;
        }
        port.postMessage(msg);
    });
}
```

What this does is simply post a provided mesage to all ports in the `ports` array. You can also optionally provide an `excluded_port`, to which the message won't be sent. This is so that when a tab wants to update state in the rest of the tabs, you don't send a message back to the initiator tab.

Let's write an `onconnect` handler. Here we will define message handlers and some initialization logic

```
onconnect = (e) => {
    const port = e.ports[0];
    ports.push(port);
}
```

In the code above, we have created an `onconnect` handler and also made sure that the newly connected port gets been added to our `ports` array.

In the `onconnect` handler, let's add handlers for receiving messages

```
port.onmessage = (e) => {
    // Sent by a tab to update state in other tabs
    if (e.data.type == 'set_state') {
               
    }
    // Sent by a tab to request the value of current state. Used when initializing the state.
    if (e.data.type == 'get_state') {
               
    }
}
```

What the code above does is create a message handler for the connected port. So each time a tab sends a message to the worker, this handler will be invoked. Then in the handler we check the type of message and act accordingly.

Now let's add a handler for each message

```
if (e.data.type == 'set_state') {
    latest_state[e.data.id] = e.data.state;

    postMessageAll({
        type: 'set_state',
        id: e.data.id,
        state: latest_state[e.data.id]
    }, port);
 }
```

This handler will get called when a tab sends a `set_state` message, a.k.a when state is updated in one of the tabs. When this happens, we update our `latest_state` with the state that the tab has sent us for the key with a matching ID, as well as send the new state to all the other tabs. We exclude the tab that initially sent the message because it already has the new state.

Next let's add our `get_state` handler. It will be called when a tab with our page first opens, to retrieve the up-to-date state value.

```
if (e.data.type == 'get') {
    port.postMessageAll({
        type: 'set_state',
        id: e.data.id,
        state: latest_state[e.data.id]
    });
}
```

What we do here is simply send a `set_state` message to the tab that requested it, with our up-to-date state.

And that's it for the worker! Pretty simple, right? We're about halfway done.

Now let's write our `useTabState` hook.

## useTabState.ts

Our useTabState will be used the same way as a regular `useState` - you call the hook and it will return an array with the state and a `setState()` function

Let's start writing our function:

```
export default function useTabState<T>(initial_state: T, id: string): [T, (new_state: T) => void]{
  const [localState, setLocalState] = useState<T>(null);
}
```

So far we have made a hook that accepts 2 arguments - `initial_state` for the initial state and `id` for a unique ID to distinguish between multiple tab states.

Then we create a state called `localState` and set its initial value to `null`

You may have noticed that we're using generics here. If you're using TypeScript but don't know how to use generics or don't want to use them, you can remove the `<T>` and change any occurence of `T` to `any`. If you're using plain JavaScript then you can remove the `<T>` and omit the types completely.

Next up let's create a function below the `localState` that updates the state across all tabs

```
const setTabState = (new_state: T) => {
    setLocalState(new_state);
    worker.port.postMessage({
        type: 'set_state',
        id: id,
        state: new_state
    });
}
```

What this function does is set its localstate as well as sends a `set_state` message to the worker, resulting in state being synced across all tabs. This is the function that our hook will return.

We're almost done! Now all we have to do is create a listener for when the shared worker sends a message to the tab.

Create a `useEffect` and add the following code to it:

```
useEffect(() => {
    worker.port.addEventListener('message', e => {
        if (!e.data?.type) {
            return;
        }

        switch (e.data.type){
            case 'set_state': {
                if (e.data.id == id) {
                    if (e.data.state) {
                        setLocalState(e.data.state);
                    } else {
                        setLocalState(initial_state);
                    }
                    
                }
            }
        }
        
    })

    worker.port.start(); // This is important, otherwise the messages won't be received.

    worker.port.postMessage({
        type: 'get_state',
        id: id
    });
}, []);
```

As you can see in the code above, we add a message event listener and update our local state with the up-to-date state when a `set_state` message has been received. Also we send a `get_state` message upon mount to get the up-to-date state. If the up-to-date state is null, then we use the `initial_state` value.

Finally, let's return our `localState` as well as the `setTabState` function:

```
return [localState, setTabState];
```

And we're done! Now all that is left is to test our hook!

## index.tsx

```
import useTabState from './useTabState';

export default function TabStateExample() {
    const [counter, setCounter] = useTabState<number>(0, 'counter');

    return (
        <div>
            <button
                onClick={() => setCounter(counter + 1)}
            >
                {counter}
            </button>
        </div>
    );
}
```

In the code above, we simply create a button that shows a number, and increments this number each time it is clicked. Try testing this page with multiple tabs!

## Conclusion

In this article we explored a simple way how one can add a tab-synced state behavior to their app using the SharedWorker API. I hope you learned something new and that you will find a use for this pattern. If you don't want to go through the hassle of learning and implementing this yourself (though it's quite simple), you can use my NPM package [react-tab-state](https://www.npmjs.com/package/@printy/react-tab-state)

If you have any questions or feedback, feel free to leave it in the comments or email me at [jorensmerenjanu@gmail.com](mailto:jorensmerenjanu@gmail.com)

Good luck in your dev journey and your projects!

Tags:
  Article, Blog