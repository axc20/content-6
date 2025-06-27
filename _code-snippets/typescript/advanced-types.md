# Advanced Types

## Example 1: Generics, Mapped Types, Type Inference, Utility Types

```tsx
export type PartialRecord<K extends string>, T> = {
  [P in K]?: T;
}

// export type PartialRecord<K extends keyof Record<string, unknown>, T> = {
//   [P in K]?: T;
// }

// Here, `PartialRecord` can have any type of key, including `number` or `symbol`
//
// export type PartialRecord<K extends any>, T> = {
//   [P in K]?: T;
// }

// Generics can be inferred from usage context, which allows you to skip
// providing them explicitly in some cases. In a React component, TS can
// use the props passed to the component to infer the types of the generics.
//
// TS is designed to be able to infer types whenever possible, even when no
// default value is provided for a generic.
//
const DateInput = <
  FieldName extends string,
  Values extends PartialRecord<FieldName, Date | undefined> &
    Record<string, unknown>
>({
  formik,
  inputName,
  isRequired = false,
  label,
}: {
  formik: FormikProps<Values>;
  inputName: FieldName;
  isRequired?: boolean;
  label: string;
}): ReactElement => {
  // React component
};

// above is better than:
//
type FormValues<T extends string, V extends Record<string, unknown>> = {
  [K in T]?: Date | undefined;
} & V;

type DateInputProps<T extends string, V extends Record<string, unknown>> = {
  formik: FormikProps<FormValues<T, V>>;
  inputName: T;
  isRequired?: boolean;
  label: string;
};

const DateInput = <T extends string, V extends Record<string, unknown>>({
  // ...
}: {
  // ...
}): ReactElement => {
  // React component
};
```
