# Project structure

## Top-level files

- `instrumentation.ts`: OpenTelemetry and instrumentation

## Top-level folders

- `app`: app router
- `public`: static assets to be served
- `src`: optional application source folder

## `app` routing conventions

### Parallel and intercepted routes

- `@folder`: Named slot
- `(.)folder`: Intercept same level
- `(..)folder`: Intercept one level above
- `(..)(..)folder`: Intercept two levels above
- `(...)folder`: Intercept from root

## Folder structure (older)

Structuring your project properly is crucial for maintainability and scalability. The specific structure might depend on your project's specific needs and team's preferences.

```
src/
|-- components/
    |-- Button/
        |-- index.tsx
        |-- Button.test.tsx
        |-- Button.module.css
|-- lib/
    |-- api.ts
    |-- context.ts
|-- styles/
    |-- globals.css
    |-- Home.module.css
|-- types/
    |-- index.d.ts
|-- tests/
    |-- setupTests.ts
    |-- testUtils.ts
|-- utils/
```

- **components**: Reusable components across your app
- **lib**: Misc code that's used in more than one place but doesn't fit into the "components" concept. Things like custom hooks, context providers, or utility functions. In some projects, `lib` can be used for backend utilities or libraries, while `utils` is used for UI utilities.
- **types**: Type defintions for your project, in one or multiple `.d.ts` files
- **tests**: A place to put test setup files and utilities
- **utils**: Utility functions that might be used across your app. If there's a lot of overlap between `lib` and `utils`, you might consider combining them into one directory.
