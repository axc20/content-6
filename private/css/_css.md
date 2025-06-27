# CSS

- Box model
  - Border: around element; specify width, color, style
  - Padding: space between content and element's boundary
  - Margin: space between elements
  - Width of element: border + padding + width
- Flexbox
  - main axis: defined by flex direction: `row`, `row-reverse`, `column`, `column-reverse`
  - cross axis: runs perpendicular to main axis
- Alignment
  - `justify-items`/`align-items`: align content in a grid along row/column; `start`, `end`, `center`, `stretch`
  - `justify-content`/`align-content`: justifies all grid content on row/column axis when total grid size is smaller than container; `start`, `end`, `center`, `stretch`, `space-around`, `space-between`, `space-evenly`
- (?) container queries

```css
/* for transparent images, create a shadow on image's content */
.class {
  filter: drop-shadow(2px 4px 8px #585858);
}

/* CSS modal */
.modal {
  visibility: hidden;
}
.modal:target {
  visibility: visible;
}
```

## Scrolling

- [fixed position div that needs to scroll if content overflows](https://stackoverflow.com/questions/16094785/have-a-fixed-position-div-that-needs-to-scroll-if-content-overflows)

```css
/* smooth scrolling */
html {
  scroll-behavior: smooth;
}

/* scroll snap */
.wrapper {
  height: 100vh;
  overflow: auto;
  scroll-snap-type: y mandatory;
}
.section {
  scroll-snap-align: center;
}
```

## Dynamic tooltips

```html
<span class="tooltip" data-tooltip="Tooltip Content">Hover me</span>
```

```css
.tooltop:before {
  content: attr(data-tooltip);
}
```

## Gradients

```css
/* rounded gradient border */
.gradient-border {
  border: solid 5px transparent;
  border-radius: 10px;
  background-image: linear-gradient(white, white), linear-gradient(315deg, #833ab4, #fd1d1d
        50%, #fcb045);
  background-origin: border-box;
  background-clip: content-box, border-box;
}

/* radial gradient */
.class {
  background: radial-gradient(circle, #0000ff 66%, transparent 66%),
    linear-gradient(to left, #0000ff10 50%, #0000ff10 50%);
}
```

## Center a div

```css
body {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

body {
  height: 100vh;
  display: grid;
  justify-content: center;
  align-items: center;
}

body {
  height: 100vh;
  display: grid;
  place-items: center;
}

.container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.parent {
  display: grid;
}
.child {
  margin: auto;
}

.parent {
  display: flex;
}
.child {
  margin: auto;
}
```
