# Jest - GQL helpers

## `spyOn`

```js
beforeEach(() => {
  // `jest.spyOn` here is used replace `console.warn`
  // this will suppress all warnings from being shown in the console
  jest.spyOn(console, 'warn').mockImplementation(() => {});
});
afterEach(() => {
  // restore the original `console.warn` after each test is done
  console.warn.mockRestore();
});
```

**Why you might suppress warnings:**

- Reduce noise: non-critical warnings may make test output difficult to read and focus on tests' results
- Third-party warnings: coming from libraries you have no control over

**Why you might NOT suppress warnings:**

- Useful feedback: warnings can be useful in highlighting potential problems or deprecated usage in your code
- Can hide real issues that you should address

Instead of suppressing warnings entirely, a better practice is to minimize the number of warnings in your code by fixing the issues that cause them. If there are warnings you can't do anything about (like those from third-party library), or warnings that you've consciously decided are acceptabble in your specific case, those could be candidates for suppression.

## GraphQL-related

This function is not needed - import the GraphQL `DocumentNode`s from the auto-generated file directly.

```ts
import fs from 'fs';
import path from 'path';
import { DocumentNode, parse, print } from 'graphql';
import gql from 'graphql-tag';

type OperationToFileMapping = Record<string, string>;
type OperationToGqlDocumentNodeMapping<T extends OperationToFileMapping> =
  Record<keyof T, DocumentNode>;

/**
 * Get GraphQL `DocumentNode`s from a mapping of operation names to file paths.
 * Use for mocking any v2-generated GraphQL queries/mutations.
 *
 * @param operationToFileMapping - An object mapping operation names to file paths,
 * where the file paths are relative to `frontend/src`.
 * @returns An object mapping the provided operation names to the corresponding
 * GraphQL `DocumentNode`s
 *
 * @example
 * const gqlQueries = getGqlDocumentNodes({
 *   'GetProjections': 'pages/estates/index.graphql',
 *   'UpdateDocumentCopyRequest': 'features/DocumentAbstraction/common/graphql/copyDocument.graphql'
 * });
 */
function getGqlDocumentNodes<T extends OperationToFileMapping>(
  operationToFileMapping: T
): OperationToGqlDocumentNodeMapping<T> {
  const fileCache: Record<string, string> = {};
  const result: OperationToGqlDocumentNodeMapping<T> = {} as OperationToGqlDocumentNodeMapping<T>;

  Object.entries(operationToFileMapping).forEach(
    ([operation, filePath]: [keyof T, string]) => {
      if (!fileCache[filePath]) {
        const fileContent == fs.readFileSync(
          path.join(__dirname, '..', filePath),
          'utf8'
        );
        fileCache[filePath] = fileContent;
      }

      const ast = parse(fileCache[filePath]);
      const operationAst = ast.definitions.find(
        def => def.kind === 'OperationDefinition' && def.name?.value === operation
      );

      if (!operationAst) {
        throw new Error(`Operation ${String(operation)} not found in \`${filePath}\``);
      }

      result[operation] = gql`
        ${print(operationAst)}
      `;
    }
  );

  return result;
}

export default getGqlDocumentNodes;
```
