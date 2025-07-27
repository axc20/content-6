## Destructuring

```js
const [a, b] = [1, 2];
// omit certain values
const [aa, , cc] = [1, 2, 3];
// combine with spread/rest operator
const [aaa, ...bbb] = [1, 2, 3];
// swap variables without temp variable
let d = 1;
let e = 2;
[d, e] = [e, d];
// advanced deep arrays
const [w, [x, [y, z]]] = [1, [2, [[[3, 4], 5], 6]]];

// for (let key in obj) { ... }

const suspects = [
  { name: 'Rusty', color: 'orange' },
  { name: 'Clone', color: 'red' }
];
const [{ color: firstColor }, { color: secondColor }] = suspects;
```

## Standard built-in objects

### Date

```javascript
const date = new Date(Date.UTC(2021, 11, 20, 3, 0, 0));
date.toLocaleString('en-US'); // → "12/19/2012, 10:00:00 PM"
// Date.now();
// new Date('1/7/2021') // .getFillYear(), .getMonth(), .getDate()
```

### Intl

```javascript
const date = new Date(Date.UTC(2020, 11, 20, 3, 23, 16, 738));
new Intl.DateTimeFormat('en-US').format(date); // → "12/19/2020"
new Intl.DateTimeFormat('en-US', { dateStyle: 'full' }).format(date); // → "Saturday, December 19, 2020"
// Intl.NumberFormat
```

### Other

```javascript
// encodeURI(), encodeURIComponent(), decodeURI(), decodeURIComponent()
```
