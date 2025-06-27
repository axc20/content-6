## Example 2

```js
// paths.js
import { URL } from 'url';
import dayjs from 'dayjs';

const SNAPSHOT = new URL(
  `../data/snapshot/${dayjs().format('YYYY.MM')}`,
  import.meta.url
).pathname;
const CSV_OUTPUT = new URL('../data/csv', import.meta.url).pathname;

export { SNAPSHOT, CSV_OUTPUT };

// utils/csv.js
import { csvFormat } from 'd3-dsv';

export const convertDataToCsv = (tableColumns, data) => {
  const columns = tableColumns.filter(c => c.selected);
  return csvFormat(
    data.map(item => {
      return columns.reduce((acc, col) => {
        acc[col.text] = item[col.id];
        return acc;
      }, {});
    })
  );
};

// utils/filter.js
export const filterData = (data, searchTerm) => {
  return data.reduce((acc, item) => {
    for (const key in item) {
      if (String(item[key]).toLowerCase().includes(searchTerm.toLowerCase())) {
        acc.push(item);
        break;
      }
      return acc;
    }
  }, []);
};

// utils/sanitizeVersion.js
export const sanitizeVersion = version => version.replace(/\^|~/, '');

// script1.js
import chalk from 'chalk';
import { convertDataToCsv } from './utils/csv.js';
import { filterData } from './utils/filter.js';
import { CSV_OUTPUT, SNAPSHOT } from './paths.js';

const tableColumns = [
  { id: 'name', text: 'Name', selected: true },
  { id: 'description', text: 'Description', selected: false },
  { id: 'version', text: 'Version', selected: true }
];
const data = [
  {
    name: 'design-language-styles',
    description: 'Styles',
    version: '4.0.0'
  },
  {
    name: 'design-language-react',
    description: 'React Component Library',
    version: '3.9.3'
  },
  {
    name: 'design-language-styles',
    description: 'Charts Library',
    version: '3.7.2'
  }
];

const csvOutput = convertDataToCsv(tableColumns, data);
const filteredData = filterData(data, 'library');

console.log(csvOutput);
console.log(filteredData);
console.log(chalk.red(`${SNAPSHOT}\n${CSV_OUTPUT}`));
```

## Example 3

```js
import { mkdir, readFile, writeFile, rm } from 'fs/promises';

const MASTER_BRANCH = encodeURIComponent('refs/heads/master');
const INTEGRATION_BRANCH = encodeURIComponent('refs/heads/integration');
console.log(MASTER_BRANCH, INTEGRATION_BRANCH);

// create directory
mkdir('esm/someDirectory/aaa/thing', { recursive: true });
mkdir('esm/someDirectory/bbb/yada', { recursive: true });

const pkgJson = JSON.parse(
  await readFile('esm/package.json', { encoding: 'utf-8' })
);

writeFile(
  'esm/someDirectory/file.txt',
  JSON.stringify({ ...pkgJson, b: Math.random() }, null, 2)
);

const currDate = new Date().toISOString().slice(0, 10);
console.log(currDate);

// remove
rm('esm/someDirectory/bbb', { recursive: true });
```

## Example 4

```js
import got from 'got';
// import ... apiFormatters, constants

const USERNAME = process.env.USERNAME;
const PASSWORD = process.env.PASSWORD;
// const ET_BITBUCKET_URL = 'https://bitbucket.etrade.com';
// const ET_BITBUCKET_API_BASE_URL = `${ET_BITBUCKET_URL}/rest/api/1.0`;
const baseOptions = {
  headers: {
    Authorization: `Basic ${Buffer.from(`${USERNAME}:${PASSWORD}`).toString(
      'base64'
    )}`
  }
};

const fetchSomething = async postId => {
  const params = new URLSearchParams({ postId });
  const url = `https://jsonplaceholder.typicode.com/comments?${params.toString()}`;

  try {
    const { body } = await got(url, baseOptions);
    // const data = await got(url, baseOptions).json();
    // format data and return it

    // fetchArchive
    // const { body } = await got(url, { ...baseOptions, responseType: 'buffer' });
    return body;
  } catch (error) {
    // error.response.body
    // JSON.parse(error.response.body)
    console.log(error);
  }
};

await fetchSomething(1);
```

## Example 5

```js
// bump-versions.mjs
import { readFileSync, writeFileSync } from 'node:fs';
import { exec } from './utils.mjs';
import { REACT_PKG_PATH } from './paths.mjs';

const updateDependencies = (path, version) => {
  const data = readFileSync(path, { encoding: 'utf-8' });
  const pkg = JSON.parse(data);
  const majorVersion = version.match(/[0-9]+/)[0];
  if (pkg.devDependencies?.['@campfire/3d-web-components']) {
    pkg.devDependencies['@campfire/3d-web-components'] = version;
  }
  if (pkg.peerDependencies?.['@campfire/3d-web-components']) {
    pkg.peerDependencies['@campfire/3d-web-components'] = `>=${majorVersion}`;
  }
  writeFileSync(path, JSON.stringify(pkg, null, 2), { encoding: 'utf-8' });
  console.log(`✓ ${pkg.name}@${pkg.version}`);
};

try {
  const version = process.argv[2];
  if (!version) {
    throw new Error(
      'Please provide a version, i.e. npm run bump-version 1.0.1'
    );
  }
  exec(
    `npx lerna version ${version} --no-private --no-push --no-git-tag-version`
  );
  console.log(`/nUpdating devDependencies and peerDependencies`);
  updateDependencies(REACT_PKG_PATH, version);
} catch (err) {
  console.log(err);
}

// et-publish.mjs
import { exec } from './utils.mjs';

try {
  // NOT using lerna publish here because
  // - https://github.com/lerna/lerna/issues/2730
  //   resolving may require scm to change how Jenkins is currently doing things
  // - see npm-publish.mjs comments
  exec('cd packages/3d-web-components && npm publish');
  exec('cd packages/3d-react && npm publish');
} catch (err) {
  console.log(err);
}

// npm-publish.mjs
import { readFileSync, writeFileSync } from 'node:fs';
import { globby } from 'globby';
import { exec } from './utils.mjs';
import { WC_PKG_PATH, REACT_PKG_PATH } from './paths.mjs';

const updateFiles = async () => {
  console.log('Updating files (@campfire/3d-* -> @etrade/campfire-3d-*)');

  // files that potentially require @campfire/3d-* -> @etrade/campfire-3d-* update
  const files = await globby([
    'packages/**/*.{json,md}',
    'packages/3d-react/.storybook/preview.js',
    'packages/3d-react/stories/**/*.{js,ts,tsx}',
    '!packages/**/node_modules',
    '!packages/**/dist',
    '!packages/**/loader',
    '!packages/**/package-lock.json',
    '!packages/**/tsconfig*',
    '!packages/3d-web-components/**/readme.md'
  ]);

  files.forEach(file => {
    let data = readFileSync(file, { encoding: 'utf-8' });

    if ([WC_PKG_PATH, REACT_PKG_PATH].some(p => p.includes(file))) {
      // update publish config
      data = data.replace(
        /registry":.*/,
        'registry": "https://registry.npmjs.org"'
      );
    }

    if (data.includes('@campfire/3d')) {
      data = data.replace(/@campfire\/3d/g, '@etrade/campfired-3d');
    }

    console.log(`✓ ${file}`);
    writeFileSync(file, data, { encoding: 'utf-8' });
  });
};

async () => {
  try {
    await updateFiles();

    exec('git branch');
    exec('npm config set registry https://registry.npmjs.org');
    exec('npm whoami');
    exec('npm install');
    exec('npx lerna bootstrap');
    exec('npm run build');
    exec('git add -A');
    exec('git status');
    exec('git commit -m "prepare for npm publish');

    // NOT using lerna publish here because
    // the 3d-angular package ("packages/3d-angular-workspace/projects/*")
    // that is specified as a lerna package location should not be published
    // its build output ("packages/3d-angular-workspace/dist/3d-angular")
    // is what should be published instead
    exec('cd packages/3d-web-components && npm publish');
    exec('cd packages/3d-react && npm publish');
  } catch (err) {
    console.error(err);
  }
};

// paths.mjs
import { URL } from 'node:url';

export const WC_PKG_PATH = new URL(
  '../packages/3d-web-components/package.json',
  import.meta.url
).pathname;
export const REACT_PKG_PATH = new URL(
  '../packages/3d-react/package.json',
  import.meta.url
).pathname;

// utils.mjs
import { execSync } from 'node:child_process';

export const exec = command => {
  execSync(command, { stdio: 'inherit' });
};
```
