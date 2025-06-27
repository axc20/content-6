# Configuring

- [Absolute Imports and Module Path Aliases](https://nextjs.org/docs/app/building-your-application/configuring/absolute-imports-and-module-aliases)

## ESLint

```shell
pnpm i -D eslint-config-prettier
```

```json
{
  "extends": ["next", "prettier"]
}
```

## Environment Variables

Next.js has built in support for loading environment variables from `.env.local` into `process.env`.

`.env`, `.env.development`, and `.env.production` files should be included in your repository as they define defaults. `.env*.local` should be added to `.gitignore` as those files are intended to be ignored. `.env.local` is where secrets can be stored.

```
# .env.local
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword

TWITTER_USER=nextjs
# Next.js will automatically expand variables that use $
# to reference other variables
# If you need need to use variable with a $ in actual value,
# it needs to be escaped `\$`
TWITTER_URL=https://twitter.com/$TWITTER_USER
```

```js
// app/api/route.js
export async function GET() {
  const db = await myDB.connect({
    host: process.env.DB_HOST,
    username: process.env.DB_USER,
    password: process.env.DB_PASS
  });
  // ...
}
```

### Bundling Environment Variables for the Browser

```
# Prefix env variables that should be accessible in browser
# with `NEXT_PUBLIC_`
NEXT_PUBLIC_ANALYTICS_ID=abcedefghijk
```

After being built, your app will no longer respond to changes to these environment variables. If you need access to runtime environment values, you'll have to setup your own API to provide them to the client (either on demand or during initialization).

## MDX

```shell
pnpm i @next/mdx @mdx-js/loader @mdx-js/react @types/mdx
```

```tsx
// mdx-components.tsx (top-level file)
import type { MDXComponents } from 'mdx/types;

// this file allows you to provide custom React components
// to be used in MDX files

// this file is required to use MDX in `app` directory
export function useMDXComponents(components: MDXComponents): MDXComponents {
  return {
    // customize built-in components, e.g. to add styling
    // h1: ({ children }) => <h1 style={{ fontSize: '100px' }}>{children}</h1>
    ...components
  }
}
```

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    mdxRs: true
  }
};

const withMDX = require('@next/mdx')();
module.exports = withMDX(nextConfig);
```

```mdx
{/* app/hello.mdx */}
Hello, Next.js!

You can import and use React components in MDX files.
```

```tsx
// app/page.tsx
import HelloWrold from './hello.mdx';

export default function Page() {
  return <HelloWrold />;
}
```

### Remote MDX

If your Markdown or MDX files do not live inside of your application, you can fetch them dynamically on the server. This is useful for fetching content from a CMS or other data source.

```tsx
// app/page.tsx
import { MDXRemote } from 'next-mdx-remote/rsc';

export default async function Home() {
  // Proceed with caution
  // MDX compiles to JS and is executed on the server
  // You should only fetch MDX content from a trusted source,
  // otherwise this can lead to remote code execution (RCE)
  const res = await fetch('...');
  const markdown = await res.text();
  return <MDXRemote source={markdown} />;
}
```

## Static Exports

Next.js enables starting as a static site or SPA, then later optionally upgrading to use features that require a server. When running `next build`, Next.js generates an HTML file per route. This static export can be deployed and hosted on any web server that can serve HTML/CSS/JS static assets.

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export'
  // Optional: Add a trailing slash to all paths `/about` -> `/about/`
  // trailingSlash: true
  // Optional: change the output directory `out` -> `dist`
  // distDir: 'dist'
};

module.exports = nextConfig;
```
