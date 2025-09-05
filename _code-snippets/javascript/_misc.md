# Misc

```js
const params = new URLSearchParams({
  v: FINVIZ_SCREENER_TYPES[screenerType],
  f: 'cap_microover,sh_avgvol_o50',
  ...pageParam
});

const pages = [...new Array(Math.ceil(universeCount / 20))].map(
  (_, i) => i + 1
);
// alternative way:
const pageCount = Math.ceil(universeCount / 20);
const pages = Array.from({ length: pageCount }, (_, i) => i + 1);

// extract numeric count from the HTML response body by searching for
// specific pattern `class="count-text"` followed by digits
const universeCount = Number(
  /class=\"count-text\"[^0-9]*([0-9]*)/g.exec(response.body)[1]
);
```
