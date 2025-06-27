```ts
// ===== Constrained type parameters
// if you need to only allow a subset of types,
// make the type extend a particular type
interface Drawable {
  draw: () => void;
}
function renderToScreen<T extends Drawable>(input: T[]) {
  input.forEach(i => i.draw());
}

// ===== Constrained and default type parameters
<T extends ConstrainedType = DefaultType>(): T

// while generics can provide more safety, the trade-off is the
// complexity, especially when you have multiple variables
// when providing APIs for others, generics offer a flexible way
// to let others use their own types with full code inference
interface CacheHost<ContentType> {
  save: (a: ContentType) => void;
}
function addObjectToCache<T, Cache extends CacheHost<T>>(obj: T, cache: Cache): Cache {
  cache.save(obj);
  return cache;
}
```

## Generic tuples

```ts
type Arr = readonly any[];
function concat<U extends Arr, V extends Arr>(a: U, b: V): [...U, ...V] {
  return [...a, ...b];
}
const strictResult = concat([1, 2] as const, ['3', '4'] as const); // type [1, 2, '3', '4']
const relaxedResult = concat([1, 2], ['3', '4']); // type (number | string)[]
```

## Arrow functions with generics

```ts
// https://stackoverflow.com/questions/32308370
function foo<T>(x: T): T {
  return x;
}

// .ts file
const foo = <T>(value: T) => {
  return value;
};
```

```tsx
// .tsx file
const foo = <T,>(value: T) => {
  return value;
};

// extends {} forces T to be object
const foo = <T extends unknown>(value: T) => {
  return value;
};
```

## Optional parameters using never as default

By using `never` as the default type for a generic parameter, you effectively make the parameter optional without having to modify the structure of the type itself. This approach allows for conditional behavior witin the type defintion based on whether or not a specific type is provided for the generic parameter.

```ts
type ColorPalette = 'red' | 'green' | 'blue';

// Since the generic type parameter is `never` by default, it
// (1) Forces users of the type to explicity specify color type
// if they want to make the property usable
// (2) Allows the definition of `Cell` without specifying color
// - rest of the structure can be used even if color is unusable
type Cell<Color extends ColorPalette = never> = {
  id: string;
  label: string;
  color?: Color;
  bgColor?: Exclude<ColorPalette, Color>;
};
```
