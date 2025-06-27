## API, Data

```js
import { URL } from 'node:url';
import { mkdir, writeFile } from 'node:fs/promises';
// unlink, rmdir
import chalk from 'chalk';
import AdmZip from 'adm-zip';
import { globby } from 'globby';

// generating excel file
// import ExcelJS from 'exceljs';

// csv
// import { csvFormat } from 'd3-dsv';

// ramda
// pathOr(null, [Branches.MASTER, 'react'], repoData)

// dayjs
// import customParseFormat from 'dayjs/plugin/customParseFormat';
// dayjs.extend(customParseFormat)
// dayjs(date2, 'YYYY.M.D').diff(dayjs(date1, 'YYYY.M.D'), 'week')

const Branches = {
  MASTER: 'master',
  INTEGRATION: 'integration'
};

// below are placeholder functions related todealing with data from APIs
export const formatData = input => {
  // perform operation(s) so that output is formatted
  // i.e. format dates, filter fields, regex
  const output = { ...input, formatted: true };
  return output;
};

export const normalizeData = input => {
  // perform operation(s) so that output is normalized
  // to match format of other expected data output
  const output = { ...input, normalized: true };
  return output;
};

export const extractData = async () => {
  // fetch data from api
  // return it or write it to files
  // await fs.mkdir(SNAPSHOT, { recursive: true });
  // await fs.writeFile(`${SNAPSHOT}/myData-snapshot.json`, JSON.stringify(snapshotData, null, 2));
  const repoList = [
    { link: '', slug: 'design-language-react' },
    { link: '', slug: 'design-language-styles' }
  ];
  for (const repo of repoList) {
    console.log(repo);
    for (const branch of Object.values(Branches)) {
      console.log(branch);
    }
  }
};

export const sortData = input => {
  return input.sort((a, b) => {
    return a.value > b.value ? 1 : -1;
  });
};

export const extractDataFromZip = async () => {
  const ZIP_FILE = new URL('zoo.zip', import.meta.url).pathname;
  const ARCHIVE_DIR = new URL('zoo', import.meta.url).pathname;
  new AdmZip(ZIP_FILE).extractAllTo(ARCHIVE_DIR, true);
  const files = await globby([`${ARCHIVE_DIR}/**/*.txt`]);
  console.log(files);
};

// misc functions
export const logError = (info, error) => {
  console.log(chalk.cyan(info), chalk.red(error));
};

export const getUniqueArray = input => Array.from(new Set(input));

extractData();
logError('some info', 'some error');
console.log(sortData([{ value: 100 }, { value: 96 }]));
console.log(getUniqueArray([1, 2, 2, 5, 5, 5]));
extractDataFromZip();
```
