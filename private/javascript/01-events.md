# Events

- `event.stopPropagation()` prevents further propagation of the current event in the capturing and bubbling phases

```javascript
event.getModifierState('Caps-Lock'); // KeyboardEvent
target.addEventListener(type, listener [, options]);
target.addEventListener(type, listener [, useCapture]);
```

## Reference

- [Events](https://developer.mozilla.org/en-US/docs/Web/Events)
- Mouse events:
  - `click`, `dblclick`
  - `mousedown`, `mouseup`: a pointing device is pressed/released on an element
  - `mouseenter`, `mouseleave`: a pointing device is moved onto/off the element that has the listener attached
  - `mousemove`: a pointing device is moved over an element (fired continuously as the mouse moves)
  - `mouseover`, `mouseout`: a pointing device is moved onto/off the element that has listener attached or onto/off one of its children
  - `select`: some text is being selected
  - `wheel`: a wheel button of a pointing device is rotated in any direction
- Keyboard events:
  - `keydown`: any key is pressed
  - `keypress`: any key (except shift, fn, or capslock) is in pressed position (fired continuously)
  - `keyup`: any key is released
- Focus events:
  - `focus`, `blur`: element has received/lost focus (does not bubble)
- Form events: `reset`, `submit`
- Value change events: `input`
- View events: `resize`, `scroll`
- Drag & Drop events:
  - `drag`: an element or text selection is being dragged (fired continuously every 350ms)
  - `dragend`: a drag operation is being ended (by releasing a mouse button or hitting the escape key)
  - `dragenter`: a dragged element or text selection enters a valid drop target
  - `dragstart`: the user starts dragging an element or text selection
  - `dragleave`: the dragged element of text selection leaves a valid drop target
  - `dragover`: an element or text selection is being dragged over a valid drop target
  - `drop`: an element is dropped on a valid drop target
- Clipboard events: `cut`, `copy`, `paste`
- Tab events: `visibilitychange`
- Resource events:
  - `load`: a resource and its dependent resources have finished loading
- DOM mutation events: `DOMContentLoaded`
- Sensor events: `devicemotion`, `deviceorientation`, `orientationchange`
- Touch events: `touchcancel`, `touchend`, `touchmove`, `touchstart`
