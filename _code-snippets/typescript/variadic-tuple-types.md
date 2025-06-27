# Variadic Tuple Types

```ts
type Tail<T extends unknown[]> = T extends [unknown, ...infer U] ? U : never;

type Arr1 = [number, string, boolean];
type Tail1 = Tail<Arr1>; // [string, boolean]
```
