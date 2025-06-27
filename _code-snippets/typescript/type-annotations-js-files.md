# Type annotations for JS code

- use to provide type annotations for JS code that doesn't have type information
- `/* @type */` notation can be used to provide inline annotations directly in your code if you don't have type declaration files
- note that this notation is not part of standard TS syntax but is special syntax recognized by some tooling (such as VS Code)

```js
/** @type {number} */
let x = 42;

function add(a, b) {
  /** @type {number} */
  let result = a + b;
  return result;
}
```

```js
/** @type {import('tailwindcss').Config} */
```
