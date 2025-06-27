# Conditional Types

Conditional types provide a way to do simple logic (`A extends B ? C : D`), where the condition is whether a type extends an expression and if so what type should be returned.

```ts
type Cat = { meows: true };
type Dog = { barks: true };
type Cheetah = { meows: true; fast: true };
type Wolf = { barks: true; howls: true };

// extract types which only conform to something which barks
type ExtractDogish<A> = A extends { barks: true } ? A : never;

// then create types which ExtractDogish wraps:
type NeverCat = ExtractDogish<Cat>; // never
type Wolfish = ExtractDogish<Wolf>; // Wolf

// useful when you want to work with a union of many types
// and reduce the number of potential options in a union
type Animals = Cat | Dog | Cheetah | Wolf;

// ===== distributive conditional type
// when applying ExtractDogish to a union type, it is same
// as running the conditional against each member of the type
type Dogish = ExtractDogish<Animals>; // Dog | Wolf

// ===== deferred conditional types
// conditional types can be used to tighten your APIs which
// can return different types depending on the inputs
declare function getID<T extends boolean>(
  fancy: T
): T extends true ? string : number;

let stringReturnValue = getID(true);
let numberReturnValue = getID(false);
let stringOrNumber = getID(Math.random() < 0.5);

// for functions where the type isn't known yet,
// you can use deferred conditional types
declare function isCatish<T>(x: T): T extends { meows: true } ? T : undefined;

// to tell TS to infer the type when deferring, use 'infer'
// infer is typically used to create meta-types which inspect
// the existing types in your code; think of it as creating a
// new variable inside the type

// the conditional checks if the type is a function, and if so
// create a new type called R based on return value of function
type GetReturnValue<T> = T extends (...args: any[]) => infer R ? R : T;

type getIDReturn = GetReturnValue<typeof getID>;
type getCat = GetReturnValue<Cat>; // Cat
```

```ts
// types here are fine but do not convey intent of code
declare function addOrConcat(x: number | string): number | string;

// function overloading, but this is verbose and tedious
declare function addOrConcat(x: string): string;
declare function addOrConcat(x: number): number;
declare function addOrConcat(x: number | string): number | string;

// conditional types!
declare function addOrConcat<T extends number | string>(
  x: T
): T extends number ? number : string;

// infer can be used in conditional types to introduce a type
// variable that the TS compiler will infer from its context
function first<T extends [any, any]>(
  pair: T
): T extends [infer U, infer U] ? U : any {
  return pair[0];
}

first([3, 'foo']); // type will be number | string
first([0, 0]); // type will be number
```

## Conditional mapped types

```ts
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}

type StringProps<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
};

type PersonStrings = StringProps<Person>; // 'firstName' | 'lastName'
```
