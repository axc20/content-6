# Overlap Elements

Use CSS grid to overlap elements instead of absolute position.

```html
<div class="wrap">
  <img src="https://..." />
  <div class="blur">
    <p>LISTEN NOW></p>
  </div>
</div>

<style>
  .wrap {
    display: grid;
    & > * {
      grid-column: 1;
      grid-row: 1;
    }
  }
  .blur {
    background: linear-gradient(
      22deg,
      rgba(255, 123, 0, 0.8),
      rgba(255, 199, 0, 0.8)
    );
  }

  img {
    width: 100%;
  }
</style>
```
