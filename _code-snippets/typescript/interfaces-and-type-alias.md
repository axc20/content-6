# Interface and Type Alias

## Interface

Reuse object types with interfaces, which is, in essence, a named object type literal. Interfaces can extend other interfaces or classes using `extends` in order to compose more complex types out of simple types.

```ts
interface Point {
  x: number;
  y: number;
}

interface Point3d extends Point {
  z: number;
}

interface Point {
  x: number;
  y: number;
  z?: number;
  // shorthand for: toGeo: () => Point;
  // optional method: toGeo?(): Point
  toGeo(): Point;
}
```

By default, optional properties are treated as having a type like `[original type] | undefined`. Normally, this would be OK, but some built-in functions like `Object.assign` and `Object.keys` behave differently whether or not a property exists (and is undefined) or not. You can use the option `exactOptionalPropertyTypes` to tell TS to not allow undefined values in thses cases.

### Index Signatures

Objects that are intended to be used as hash maps can be given an index signature, which enables arbitrary keys to be defined on an object.

```ts
interface HashMapOfPoints {
  [key: string]: Point;
}

// index signatures can also include Symbols and template string patterns
// in addition to string and number
const serviceUrl = Symbol('ServiceUrl');
const servicePort = Symbol('ServicePort');

interface Configuration {
  [key: symbol]: string | number;
  [key: `service-${string}`]: string | number;
}

const config: Configuration = {};
config[serviceUrl] = 'my-url';
config[servicePort] = 8080;
config['service-host'] = 'host';
config['serivce-port'] = 8080;
config['host'] = 'host'; // error
```

### Constant Values

Interface properties can also be named using constant values, similar to computed property names on normal objects. Must be constant strings, numbers, or symbols.

```ts
const Foo = 'Foo';
const Bar = Symbol();

interface MyInterface {
  [Foo]: number;
  [Bar]: boolean;
}
```

### Extends

```ts
interface Child extends Parent, SomeClass {
  property: Type;
  optionalProp?: Type;
  optionalMethod?(arg1: Type): ReturnType;
}
```

## Type Alias

A type alias is just a reference to a specific type.

```ts
import * as foo from './foo';

type Foo = foo.Foo;
type Bar = () => string;
type StringOrNumer = string | number;
type PromiseOrValue<T> = T | Promise<T>;

type Name = string;
type Direction = 'left' | 'right';
type ElementCreator = (type: string) => Element;
type Point = { x: number; y: number };
// extended using intersection operator
type Point3D = Point & { z: number };
type PointProp = keyof Point; // 'x' | 'y'

const point: Point = { x: 1, y: 2 };

type PtValProp = keyof typeof point; // 'x' | 'y'
```

Unlike interfaces, aliases are not subject to declaration merging. When an interface is defined multiple times in a single scope, the declarations will be merged into a single interface. A type alias, on the other hand, is a named entity, like a variable. As with variables, type declarations are block scoped, and you can't declare two types with the same name in the same scope.

For publicly exposed types, it's better to make them an interface. Interfaces are also recommended because you will get better error messages (?).

TS is a structural type system, so it is possible to intermix the use of interfaces and types.

```ts
type BirdType = {
  wings: 2;
};
interface BirdInterface {
  wings: 2;
}

const bird1: BirdType = { wings: 2 };
const bird2: BirdInterface = { wings: 2 };
const bird3: BirdInterface = bird1;

type Owl = { nocturnal: true } & BirdType;
type Robin = { nocturnal: true } & BirdInterface;

interface Peacock extends BirdType {
  colorful: true;
  files: false;
}
interface Chicken extends BirdInterface {
  colorful: false;
  flies: false;
}

let owl: Owl = { wings: 2, nocturnal: true };
let chicken: Chicken = { wings: 2, colorful: false, flies: false };
```

You can use computed properties via the `in` keyword. This programmatically generates mapped types.

```ts
type FruitColors<T> = { [P in keyof T]: string[] };

const fruitCodes = {
  apple: 11123,
  banana: 33323,
  pear: 33343
};

// must include all keys present in fruitCodes
const fruitColors: FruitColors<typeof fruitCodes> = {
  apple: ['red', 'green'],
  banana: ['yellow'],
  pear: ['green']
};
```
