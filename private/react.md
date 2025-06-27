# react

### What is `useEffect` for?

```js
// synchronization
useEffect(() => {
  const sub = createThing(input).subscribe(value => {
    // do something with value
  });
  return sub.unsubscribe();
}, [input]);

// tracking things in the DOM
useEffect(() => {
  const handler = event => {};
  elRef.current.addEventListener('...', handler);
  return () => {
    elRef.current.removeEventListener('...', handler);
  };
}, []);
```

### Two types of effects

1. Action effects - "Fire and forget"
2. Activity effects - "Synchronized" (safe for React 18)

If we consider UI as a function of state, `(state, event) => nextState`, action effects can go on event handlers (`onSubmit`, `onClick`, etc.). These effects happen outside of rendering, and the always happen during state transitions.

State machines model effects with `(state, event) => (nextState, effects)` (declarative effects). Effects are defined in the transition ("when this event happens, then this effect happens").

```js
import { useState, useCallback } from 'react';

function useSpicyReducer(reducer, initialState, executeEffect) {
  const [state, setState] = useState(initialState);
  const spicyDispatch = useCallback(
    event => {
      // calculate next state
      const nextState = reducer(state, event);

      // execute effect based on transition
      executeEffect(state, event, nextState);

      // commit next state
      setState(nextState);
    },
    [reducer, state, executeEffect]
  );
  return [state, spicyDispatch];
}
```

Thus, action effects happen in state transitions, which happen to be executed at same time as event handlers.

## Render as you fetch with Suspense

?
