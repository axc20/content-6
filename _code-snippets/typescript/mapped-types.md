# Mapped types

Mapped types allow for the creation of new types based on existing types by mapping properties of an existing type to a new type.

```ts
// common cases for using a mapped type is dealing with partial
// subsets of an existing type
interface Artist {
  id: number;
  name: string;
  bio: string;
}

// if you were to send an update to the API which only changes
// a subset of Artist, you would typically have to create an
// additional type
interface ArtistForEdit {
  id: number;
  name?: string;
  bio?: string;
}

// it's very likely that this would get out of sync with Artist
// mapped types let you create a change in an existing type
type MyPartialType<Type> = {
  // for every existing property inside the type of Type
  // convert it to be a ?: verison
  [Property in keyof Type]?: Type[Property];
};

// now we can use the mapped type instead to create our edit interface
type MappedArtistForEdit = MyPartialType<Artist>;

// this is close to perfect, but it does allow id to be undefined
// using an intersection type, we can improve this:
type MyPartialTypeForEdit<Type> = {
  [Property in keyof Type]?: Type[Property];
} & { id: number };

type CorrectMappedArtistForEdit = MyPartialTypeForEdit<Artist>;

// ==========
// Stringify will have same properties as T,
// but those properties will all have values of type string
type Stringify<T> = {
  [P in keyof T]: string;
};

interface Point {
  x: number;
  y: number;
}
type StringPoint = Stringify<Point>;
const point A: StringPoint = { x: '4', y: '3' }
```

## How to use a generic parameter as object key

```ts
type OptionFlags<Type> = {
  [Property in keyof Type]: boolean;
};
type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};
type FeatureOptions = OptionFlags<FeatureFlags>;
```

```ts
interface GraphQLResponse<QueryKey extends string, ResponseType> {
  data: {
    [key in QueryKey]: ResponseType;
  };
}

interface User {
  id: string;
  username: string;
}

type UserByIdResponse = GraphQLResponse<'userById', User>;
type UserByUsernameResponse = GraphQLResponse<'userByUserName', User>;

const userByIdResult: UserByIdResponse = {
  data: {
    userById: {
      id: 123,
      username: 'joseph'
    }
  }
};

const userByUsername: UserByUsernameResponse = {
  data: {
    userByUsername: {
      id: 123,
      username: 'joseph'
    }
  }
};
```

## Add or remove readonly or ? modifiers from mapped properties

```ts
// makes all properties of its source type non-optional and writable
type MutableRequired<T> = { -readonly [P in keyof T]-?: T[P] };
// makes all properties optional and readonly
type ReadonlyPartial<T> = { +readonly [P in keyof T]+?: T[P] };

interface Point {
  readonly x: number;
  y: number;
}
const pointA: ReadonlyPartial<Point> = { x: 4 };
pointA.y = 3; // Error: readonly
const pointB: MutableRequired<Point> = { x: 4, y: 3 };
pointB.x = 2; // valid
```

## Map over a tuple type and return a new tuple type

```ts
type ToPromise<T> = { [K in typeof T]: Promise<T[K]> };
type Point = [number, number];
type PromisePoint = ToPromise<Point>;
const point: PromisePoint = [Promise.resolve(2), Promise.resolve(3)];
```

## Remap keys into new type using `as`

```ts
interface Person {
  name: string;
  age: number;
  location: string;
}

// allow keys to be derived from keys of a generic source:
// combine literal types and re-mapping to create a generic getter type
// whose keys represent accessor methods for the source type's fields
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
type LazyPerson = Getters<Person>;
```

## Built-in types (for common mapped type patterns)

```ts
Partial<T>; // all properties of T set to optional
Required<T>; // all properties of T set to required
Readonly<T>; // all properties of T set to readonly
Record<K, T>; // property names from K, where each property has type T
Pick<T, K>; // just the properties from T specified by K
OMit<T, K>; // all the properties from T except those specified by K
```
