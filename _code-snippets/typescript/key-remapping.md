# Key Remapping via `as``

- https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as

```ts
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKewType]: Type[Properties];
};

// Leverage template literal types to create new property names from prior ones:
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<
    string & Property
  >}`]: () => Type[Property];
};

interface Person {
  name: string;
  age: number;
  location: string;
}

// type LazyPerson = { getName: () => string; getAge: () => number; getLocation: () => string; }
type LazyPerson = Getters<Person>;

// Filter out keys by producing `never` via a conditional type
// Remove the 'kind' property:
type RemoveKindField<Type> = {
  [Property in keyof Type as Exclude<Property, 'kind'>]: Type[Property];
};

interface Circle {
  kind: 'circle';
  radius: number;
}

// type KindlessCircle = { radius: number }
type KindlessCircle = RemoveKindField<Circle>;

// Map over arbitrary unions of any type (not just unions of `string | number | symbol`)
type EventConfig<Events extends { kind: string }> = {
  [E in Events as E['kind']]: (event: E) => void;
};

type SquareEvent = { kind: 'square'; x: number; y: number };
type CircleEvent = { kind: 'circle'; radius: number };

// type Config = { square: (event: SquareEvent) => void; circle: (event: CircleEvent) => void; }
type Config = EventConfig<SquareEvent | CircleEvent>;

// Example of a mapped type using a conditional type
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};

type DBFields = {
  id: { format: 'incrementing' };
  name: { type: string; pii: true };
};

// type ObjectsNeedingGDPRDeletion = { id: false; name: true; }
type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;
```

## Rename key of object type

```ts
// SO 52702461
type A = {
  one: number;
  two: string;
  three: boolean;
};

type RenameByT<T, U> = {
  [K in keyof U as K extends keyof T
    ? T[K] extends string
      ? T[K]
      : never
    : K]: K extends keyof U ? U[K] : never;
};

// { one: number; two: string; three: boolean; };
type RenameOne = RenameByT<{ three: 'four'; five: 'nouse' }, A>;

// { x: number; y: string; z: boolean; };
type RenameAll = RenameByT<{ one: 'x'; two: 'y'; three: 'z' }, A>;

// Older answer:
type Rename<T, K extends keyof T, N extends string> = Pick<
  T,
  Exclude<keyof T, K>
> & { [P in N]: T[K] };

type Renamed = Rename<A, 'three', 'four'>;
```
