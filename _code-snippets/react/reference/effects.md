# You Might Not Need an Effect

Effects are an escape hatch that allows you to synchronize your React components with some external system like a non-React widget, network, or the browser DOM. If there is no external system involved (for example, if you want to update a component's state when some props or state change), you shouldn't need an Effect.

## Remove unnecessary effects

- **You don't need Effects to transform data for rendering.** To avoid unnecessary render passes, transform all the data at the top level of your components. That code will automatically re-run whenever your props or state change.
- **You don't need Effects to handle user events.** Handle user events in the corresponding event handlers.
- You _do_ need Effects to synchronize with external systems (jQuery widget, fetch data).

## Updating state based on props or state

When something can be calculated from the existing props or state, don't put it in state. Instead calculate it during rendering.

```js
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // Good: calculated during rendering
  const fullName = firstName + ' ' + lastName;
}
```

## Caching expensive calculations

```js
function TodoList({ todos, filter }) {
  const [newTodo, setTodo] = useState('');
  // This is fine if getFilteredTodos() is not slow
  const visibleTodos = getFilteredTodos(todo, filter);

  // Cache or memoize an expensive calculation:
  // Does not re-run unless todos or filter change
  const visibleTodos = useMemo(() => {
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
}
```

In general, unless you're creating or looping over thousands of objects, it's probably not expensive. If you want to get more confidence, you can add a console log to measure the time spent in a piece of code. Keep in mind your machine is probably faster than your users' so it's a good idea to test the performance with an artificial slowdown (Chrome offers a CPU Throttling option).

```js
console.time('filter array');
// const visibleTodos =
console.timeEnd('filter array');
```

Also note that measuring performance in development will not give you the most accurate results due to Strict Mode, which will render components twice. To get the most accurate timings, build your app for production and test it on a device like your users have.

## Adjusting some state when a prop changes

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

Storing information from previous renders like this can be hard to understand, but it's better than updating the same state in an Effect. In the above example, `setSelection` is called directly during a render. React will re-render the `List` immediately after it exits with a `return` statement. React has not rendered the `List` children or updated the DOM yet, so this lets the `List` children skip rendering the stale `selection` value.

**Although this pattern is more efficient than an Effect, most components shouldn't need it either.** No matter how you do it, adjusting state based on props or other state makes your data flow more difficult to understand and debug. Always check whether you can reset all state with a key or calculate everything during rendering instead. For example, instead of storing (and resetting) the selected _item_, you can store the selected _item ID_:

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // Best: Calculate everything during rendering
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

## Sharing logic between event handlers

```js
function ProductPage({ product, addToCart }) {
  // Good: Event-specific logic is called from event handlers
  function buyProduct() {
    addToCard(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

## Sending a POST request

```js
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // Good: This logic runs because the component was displayed
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // Good: Event-specific logic is in the event handler
    post('/api/register', { firstName, lastName });
  }
  // ...
}
```

When you choose whether to put some logic into an event handler or an Effect, the main question you need to answer is _what kind of logic_ it is from the user's perspective. If this logic is caused by a particular interaction, keep it in the event handler. If it's caused by the user _seeing_ the component on the screen, keep it in the Effect.

## Chains of computations

Calculate what you can during rendering, and adjust the state in the event handler:

```js
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // Calculate what you can during rendering
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // Calculate all the next state in the event handler
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
}
```

In some cases, you can't calculate the next state directly in the event handler. For example, imagine a form with multiple dropdowns where the options of the next dropdown depend on the selected value of the previous dropdown. Then, a chain of Effects is appropriate because you are synchronizing with network.

## Initalizing the application

If some logic must run _once per app load_ rather than _once per component mount_, add a top-level variable to track whether it has already executed:

```js
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // Only runs once per app load
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

You can also run it during module initialization and before the app renders:

```js
// Check if we're running in the browser
if (typeof window !== 'undefined') {
  // Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

Code at the top level runs once your component is imported, even if it doesn't end up being rendered. To avoid slowdown or surprising behavior when immporting arbitrary components, don't overuse this pattern. Keep app-wide initialization logic to root component modules like `App.js` or in your application's entry point.

## Notifying parent components about state changes

```js
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // Good: Perform all updates during the event that caused them
  function updateToggle(nextIsOn) {
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

```js
// Also good: "Lifting state up" - the component is fully controlled by its parent
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

## Passing data to the parent

Data flow should be predictable: flows down from the parent to the child

```js
function Parent() {
  // ...
  // Good: Passing data down to the child
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

## Subscribing to an external store

```js
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // Good: Subscribing to an external store with a built-in hook
  return useSyncExternalStore(
    // React won't resubscribe for as long as you pass the same function
    subscribe,
    // how to get the value on the client
    () => navigator.onLine,
    // how to get the value on the server
    () => true
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

## Fetching data

To fix race condition of requests, you need to add a cleanup function to ignore stale responses:

```js
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

Thie ensures that when your Effect fetches data, all responses except the last requested one will be ignored.

Issues such as race conditions are not trivial, which is why modern frameworks (such as Next.js) provide more efficient built-in data fetching mechanisms than fetching data in Effects.
