# Examples

## Type Alias, Record Type, Function Type, Partial Type

```ts
type SetRowStyleFunc = (values: Record<string, unknown>) => React.CSSProperties;

const highlightRows: SetRowStyleFunc = (values: Partial<ReviewsForWEPS>) =>
  !values.reviewer?.id ? { backgroundColor: '#fff8e5' } : {};
```

## Type Guard

```ts
// Use filter with type guard to remove null and undefined values, and to assure
// TypeScript's type system that the resulting array will not contain these types
const categories = notes
  .map(note => note.provisionCategory)
  .filter(
    (category): category is NonNullable<SelectableNote['provisionCategory']> =>
      !!category
  );
```

## Conditional Types

```ts
// This checks if the `setEditProvisionModal` property of the `ProvisionDraggableDisplay`
// component's prop is a dispatch function for a React state setter. If it is, it infers
// the type `T` that the state setter uses, and `EditProvisionModalType` becomes that type
type EditProvisionModalType = ComponentProps<
  typeof ProvisionDraggableDisplay
>['setEditProvisionModal'] extends Dispatch<SetStateAction<infer T>>
  ? T
  : never;
```
