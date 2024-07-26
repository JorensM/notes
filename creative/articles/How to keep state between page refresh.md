# How to keep state between page refreshes in React

Hello dear reader!

In this article I would like to show you how to keep state between page refreshes in React.

This sort of thing is called **state persistance**, and it allows you to keep your state intact even through page refreshes or closing of the browser.

State persistance has many use cases, for example you might want to keep the filled fields in a form stay even after the user has left the page and then came back later.

Making state persistant is actually quite easy. We will be utilizing local storage for this.

This project will be using TypeScript, but you can use JavaScript too, in which case just omit the types.

So without further ado, let's get started!

## Project Structure

We will be creating a hook called `usePersistState()` that will be used just like a regular state hook, with the added bonus of being persistant.

This article assumes that you already know how to set up a project, so I will skip that part. Feel free to use any tool such as CRA, Vite, NextJS, etc. for setting up the project.

Our project structure will look like this

 * index.tsx - page where we will test our hook
 * usePersistState.ts - file where we will hold our hook

So now that we have laid out our project structure, we can start coding!

## usePresistState.ts

First of all, let's scaffold our hook function:

```
export default function usePersistState<T>(initial_value: T, id: string): [T, (new_state: T) => void] {

}
```

So we have declared a function `usePersistState` that has a generic type `T`. As arguments you can specify the state's initial value, as well as an unique id to identify our state within local storage. Finally we specify it's return type as an array containing the state and the state setter function (just like a regular `useState()`).

Next let's create a new `_initial_state` variable that will determine whether to use the value from local storage or the value passed to the hook.

```
// Set initial value
const _initial_value = useMemo(() => {
    const local_storage_value_str = localStorage.getItem('state:' + id);
    // If there is a value stored in localStorage, use that
    if(local_storage_value_str) {
        return JSON.parse(local_storage_value_str);
    } 
    // Otherwise use initial_value that was passed to the function
    return initial_value;
}, []);
```

What we do here is check if localStorage has an item already stored for our state. If yes, then we use the localStorage's value for our initial state. Otherwise we use the `initial_value` that was passed as an argument to our hook. We wrap this in `useMemo` so the `_initial_value` gets set only once, upon mount (because we only need this upon mount).

Next let's add the state itself that uses the `_initial_value` we defined in the previous code snippet.

```
const [state, setState] = useState(_initial_value);
```

Now let's add add an `useEffect` that runs every time our state changes, in order to save the state into local storage:

```
useEffect(() => {
    const state_str = JSON.stringify(state); // Stringified state
    localStorage.setItem('state:' + id, state_str) // Set stringified state as item in localStorage
}, [state]);
```

What this does is simply, whenever our `state` changes, store the `state` value in localStorage.

Finally, let's return our `state` and `setState()`.

```
return [state, setState];
```

Aaaand we're done! Our hook is ready, now we can test it out!

## index.tsx

In `index.tsx`, paste the following code:

```
import { usePersistState } from '@printy/react-persist-state/src/index';
import React from 'react'

export default function PersistStateExample() {

    const [counter, setCounter] = usePersistState(0, 'counter');

    return (
        <div>
            <button
                onClick={() => setCounter(counter + 1)}
            >
                {counter}
            </button>
        </div>
    )
}
```

What we do here is create a `counter` state using our newly made hook and a button that shows the `counter` value as well as increments the counter whenever it is clicked. Try clicking it a few times, then refreshing the page. You will see that the state has persisted. You can even completely close your browser and repoen the page, and the state will have remained!

## Conclusion

In this article we learned how to create persistant state in React that doesn't reset when we refresh the page. I hope this article was useful to you and that you will find a way to use this method.

If you have any questions or feedback, feel free to leave a comment down below!

Thanks for reading, and have a great day!


Tags:
  Article, Blog