## regex

```js
const commentRegex = new RegExp(/<!--([\s\S]*?)-->/g);
const TEST_STRING = `
<div>
  <!-- blah blah blah -->
  <h1>Hello World</h1>
</div>`;

const testString1 = TEST_STRING.replace(commentRegex, '');
// $1 refers to first capture group
const testString2 = TEST_STRING.replace(commentRegex, '<!-- COMMENT: $1-->');

// result[1] is the first capture group
const result = commentRegex.exec(TEST_STRING);
```

```js
const DESIGN_LANGUAGE = 'design-language';
const DLR = `${DESIGN_LANGUAGE}-react`;
const DLC = `${DESIGN_LANGUAGE}-charts`;
const importRegex = new RegExp(
  `import[\\sa-zA-Z,\\{]*.*${DESIGN_LANGUAGE}.*`,
  'g'
);
const refineRegex = new RegExp(/\s+as[A-Z a-z]*|\n/, 'g'); // match any import as alias and new lines
const getDlRegex = value =>
  new RegExp(`import[\\s{]*(.*?)[\\s\\}]*from.*${value}[/libes]*([A-Za-z]*).*`);

const contents = `
  import { 
    Container,
    Col,
    Row,
    TextInput,
    Button,
    Dropdown 
  } from 'design-language-react';
  import { ProgressBar as DLProgressBar, UniversalMessage } from '@etrade/design-language-react';
  import { CompoundingChart } from '@etrade/design-language-charts';
  import DLSpinner from 'design-language-react/es/Spinner';
  import { default as DLSpinner } from '@etrade/design-language-react/lib/Spinner';
  import { Component, OtherComponent, Hello } from 'some-other-package';
`;

const importsData = contents.match(importRegex);

console.log(importsData);

const getComponentsFromMatch = matchResult => {
  const captureGroup1 = matchResult[1];
  const captureGroup2 = matchResult[2];
  let components = captureGroup1
    .split(',')
    .map(c => c.trim().replace('default', captureGroup2));

  // "import DLSpinner from 'design-language-react/es/Spinner';" --> Spinner
  if (
    components.length === 1 &&
    captureGroup2 &&
    components[0] !== captureGroup2
  ) {
    components = [captureGroup2];
  }

  return components;
};

const result = importsData.reduce(
  (acc, currValue) => {
    const refinedValue = currValue.replace(refineRegex, '');
    const dlrResult = getDlRegex(DLR).exec(refinedValue);
    const dlcResult = getDlRegex(DLC).exec(refinedValue);

    if (dlrResult) {
      getComponentsFromMatch(dlrResult).forEach(component => {
        acc[DLR][component] = !acc[DLR][component]
          ? 1
          : acc[DLR][component] + 1;
      });
    }

    if (dlcResult) {
      getComponentsFromMatch(dlcResult).forEach(component => {
        acc[DLC][component] = !acc[DLC][component]
          ? 1
          : acc[DLC][component] + 1;
      });
    }

    return acc;
  },
  { [DLR]: {}, [DLC]: {} }
);

console.log(result);
```

## remove item from array

```ts
// SO 5767325
function removeItem<T>(arr: T[], value: T): T[] {
  const index = arr.indexOf(value);
  if (index > -1) {
    arr.splice(index, 1);
  }
  return arr;
}
```
