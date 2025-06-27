# Routing

The **App Router** (`/app`, Next.js 13) supports shared layouts, nested routing, loading states, error handling, and more.

**Folders** are used to define routes (single path of nested folders, following file-system hierarchy from root folder to a final leaf folder that includes a `page.tsx` file).

**Files** are used to create UI that is shown for a route segment.

- `layout.tsx`: Shared UI for a segment and its children
- `page.tsx`: Unique UI of a route and make routes publicly accessible
- `loading.tsx`: Loading UI for a segment and its children (React suspense boundary)
- `not-found.tsx`: Not found UI for a segement and its children (React error boundary)
- `error.tsx`: Error UI for a segment and its children (React error boundary)
- `global-error.tsx`: Global Error UI
- `route.tsx`: Server-side API endpoint
- `template.tsx`: Specialised re-rendered Layout UI
- `default.tsx`: Fallback UI for Parallel Routes

In addition to special files above, you can colocate your own files (e.g. components, styles, tests, etc.) inside folders in the `app` directory.

```
app/
|-- components/
    |-- button.tsx (not routable)
|-- lib/
    |-- constants.tsx (not routable)
|-- dashboard/
    |-- page.tsx
    |-- nav.tsx (not routable)
|-- api/
    |-- route.ts
    |-- db.ts (not routable)
```

## Component Hierarchy

```tsx
render(
  <Layout>
    <Template>
      <ErrorBoundary fallback={<Error />}>
        <Suspense fallback={<Loading />}>
          <ErrorBoundary fallback={<NotFound />}>
            {/* page.tsx or nested layout.tsx */}
            <Page />
          </ErrorBoundary>
        </Suspense>
      </ErrorBoundary>
    </Template>
  </Layout>
);

// in a nested route, the components of a segment will be
// nested inside the components of its parent segment
render(
  <Layout>
    <ErrorBoundary fallback={<Error />}>
      <Suspense fallback={<Loading />}>
        {/* Nested route (i.e. /dashboard/settings) */}
        <Layout>
          <ErrorBoundary fallback={<Error />}>
            <Suspense fallback={<Loading />}>
              <Page />
            </Suspense>
          </ErrorBoundary>
        </Layout>
      </Suspense>
    </ErrorBoundary>
  </Layout>
);
```

## Layouts

A layout is UI that is shared between multiple pages. On navigation, layouts preserve state, remain interactive, and do not re-render. Layouts can also be nested.

You can define a layout by default exporting a React component from a `layout.tsx`. The component should accept a `children` prop that will be populated with a child layout (if it exists) or a child page during rendering.

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <section>
      {/* Include shared UI here, e.g. a header or sidebar */}
      <nav></nav>
      {children}
    </section>
  );
}
```

- You can use Route Groups to opt specific route segments in and out of shared layouts.
- Layouts are Server Components by default but can be set to a Client Component.
- Layouts can fetch data.
- Passing data between a parent layout and its children is not possible. However you can fetch the same data in a route more than once, and React will automatically dedupe the requests without affecting performance.
- Layouts do not have access to the current route segments. To access route segments, you can `useSelectedLayoutSegment` or `useSelectedLayoutSegements` in a Client Component

### Root Layout

- The top-most layout is the Root Layout. This required layout is shared across all pages and must contain `html` and `body` tags.
- You can use the [built-in SEO support](https://nextjs.org/docs/app/building-your-application/optimizing/metadata) to manage `<head>` HTML elements, for example, the `<title>` element.
- You can use route groups to create multiple root layouts.
- The root layout is a Server Component and cannot be set to a Client Component.

#### Modifying `<head>`

```tsx
// app/page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Next.js'
};

export default function Page() {
  return '...';
}
```

## Templates

Templates are similar to layouts in that they wrap each child layout or page. Unlike layouts that persist across routes and maintain state, templates create a new instance for each of their children on navigation. This means that when a user navigates between routes that share a template, a new instance of the component is mounted, DOM elements are recreated, state is not preserved, and effects are re-synchronized.

Templates may be more suitable than layouts in some cases:

- Enter/exit animations using CSS or animation libraries
- Features that rely on `useEffect` (e.g. logging page views) and `useState` (e.g. a per-page feedback form)
- To change default framework behavior. For example, Suspense Boundaries inside layouts only show the fallback the first time the Layout is loaded and not when switching pages. For templates, the fallback is shown on each navigation.

It is recommended to use Layouts unless you have a specific reason to use Templates.

---

## Linking and Navigating

Two ways to navigate between routes: `<Link>` component and `useRouter` hook.

### Checking Active Links

```js
// app/ui/Navigation.js
'use client';

import { usePathname } from 'next/navigation';
import Link from 'next/link';

export function Navigation({ navLinks }) {
  const pathname = usePathname();

  return (
    <>
      {navLinks.map(link => {
        const isActive = pathname.startsWith(link.href);

        return (
          <Link
            className={isActive ? 'text-blue' : 'text-black'}
            href={link.href}
            key={link.name}
          >
            {link.name}
          </Link>
        );
      })}
    </>
  );
}
```

### Scrolling to an `id`

The default behavior of `<Link>` is to scroll to the top of the route segment that has changed. When there is an `id` defined in `href`, it will scroll to the specific `id`, similarly to a normal `<a>` tag.

### `useRouter` Hook

Use to programmatically change routes inside Client Components.

```js
'use client';

import { useRouter } from 'next/navigation';

export default function Page() {
  const router = useRouter();

  return (
    <button type="button" onClick={() => router.push('/dashboard')}>
      Dashboard
    </button>
  );
}
```

### How Navigation Works

- A route transition is initiated using `<Link>` or calling `router.push()`.
- The router updates the URL in the browser's address bar.
- The router avoids unnecessary work by re-using segments that haven't changed (e.g. shared layouts) from the client-side cache. This is also referred to as partial rendering.
- If the conditions of soft navigation are met, the router fetches the new segment from the cache rather than the server. If not, the router performs a hard navigation and fetches the Server Component payload from the server.
- If created, loading UI is shown from the server while the payload is being fetched.
- The router uses the cached or fresh payload to render the new segments on the client.

Server Actions can be used to revalidate data on-demand by path (`revalidatePath`) or by cache tag (`revalidateTag`).

---

## Route Groups

You can mark a folder (`(folderName)`) as a Route Gorup to prevent the folder from being included in the route's URL path. This allows you to organize your route segments and project files into logical groups without affecting the URL path structure.

Route groups are useful for:

- Organizing routes into groups, e.g. by site section, intent, or team
- Enabling nested layouts in the same route segment level
  - Creating multiple nested layouts in the same segment, including multiple root layouts
  - Adding a layout to a subset of routes in a common segment

```
app/
|--layout.js (remove this to create multiple root layouts)
|-- (marketing)/
    |-- layout.js (if root layout, add html and body tags)
    |-- about/
        |-- page.js (/about)
    |-- blog/
        |-- page.js (/blog)
|-- (shop)/
    |-- layout.js
    |-- account
        |--page.js (/account)
```

---

## Dynamic Routes

Dynamic Segments are passed as the `params` prop to `layout`, `page`, `route`, and `generateMetadata` functions.

```tsx
// app/blog/[slug]/page.tsx
export default function Page({ params }: { params: { slug: string } }) {
  return <div>My Post: {params.slug}</div>;
}
```

### Generating Static Params

The `generateStaticParams` function can be used to statically generate routes at build time instead of on-demand at request time.

```tsx
export async function generateStaticParams() {
  const posts = await fetch('...').then(res => res.json());

  return posts.map(post => ({
    slug: post.slug
  }));
}
```

### Catch-all Segments

- `[...folderName]`
- `[[...folderName]]`: optional catch-all segments - The route without the parameter is also matched.

--

## Streaming with Suspense

You can manually create Suspense Boundaries for your own UI components.

With Server-Side Rendering (SSR), there are a series of steps that need to be completed before a user can see and interact with a page:

1. All data for a given page is fetched on the server.
2. The server then renders the HTML for the page.
3. The HTML, CSS, and JS for the page are sent to the client.
4. A non-interactive UI is shown using the generated HTML and CSS.
5. React hydrates the UI to make it interactive (attaching event listeners and so on).

These steps are sequential and blocking. SSR with React and Next.js helps improve perceived loading performance by showing a non-interactive page to the user as soon as possible. However, it can still be slow as all the data fetching on the server needs to be completed before the page can be shown to the user.

**Streaming** allows you to break down the page's HTML into smaller chunks and progressively send those chunks from the server to the client. Components that have higher priority (e.g. product information) or that don't rely on data can be sent first (e.g. layout), and React can start hydration earlier. Components that have lower priority (e.g. reviews, related products) can be sent in the same server request after their data has been fetched.

Streaming is particularly beneficial when you want to prevent long data requests from blocking the page from rendering as it can reduce the Time to First Byte (TTFB) and First Contentful Paint (FCP). It also helps improve Time to Interactive (TTI).

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';
import { PostFeed, Weather } from './Components';

export default function Posts() {
  return (
    <section>
      <Suspense fallback={<p>Loading feed...</p>}>
        <PostFeed />
      </Suspense>
      <Suspense fallback={<p>Loading weather...</p>}>
        <Weather />
      </Suspense>
    </section>
  );
}
```

---

## Error Handling

```tsx
// app/dashboard/error.tsx
'use client'; // Error components must be Client Components

export default function Error({
  error,
  reset
}: {
  error: Error;
  reset: () => void;
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  );
}
```

---

## Parallel Routes

---

## Intercepting Routes

---

## Route Handlers

---

## Middleware

Middleware allows you to run code before a request is completed. Then, based on the incoming request, you can modify the response by rewriting, redirecting, modifying the request or response headers, or responding directly. Middleware runs before cached content and routes are matched.

```ts
// middelware.ts (top-level file)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

// this function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url));
}

// matcher allows to filter Middleware to run on specific paths
export const config = {
  matcher: '/about/:path*'
  // Match multiple paths:
  // matcher: ['/about/:path*', '/dashboard/:path*']
  //
  // Regex:
  // Match all request paths except for ones starting with:
  // - api (API routes)
  // - _next/static (static files)
  // - _next/image (image optimization files)
  // - favicon.ico (favicon file)
  // matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)']
};
```

- [path-to-regexp](https://github.com/pillarjs/path-to-regexp#path-to-regexp-1)

### Matching Paths

Middleware will be invoked for every route in your project. Execution order:

1. `headers` from `next.config.js`
2. `redirects` from `next.config.js`
3. Middleware (`rewrites`, `redirects`, etc.)
4. `beforeFiles` (`rewrites`) from `next.config.js`
5. Filesystem routes (`public/`, `_next/static/`, `pages/`, `app/`, etc.)
6. `afterFiles` (`rewrites`) from `next.config.js`
7. Dynamic routes (`/blog/[slug]`)
8. `fallback` (`rewrites`) from `next.config.js`

### Conditional Statements

```ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url));
  }

  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url));
  }
}
```

### NextResponse

### Using Cookies

### Setting Headers

### Producing a Response

### Advanced Middelware Flags

---

## Project Organization

- Safe colocation by default
- Private folders: `_folderName`
  - Opt the folder and all its subfolders out of routing
  - Can be useful for separating UI logic from routing logic, consistently organizing internal files across a project and the Next.js ecosystem, sorting and grouping files in code editors, and avoiding potential naming conflicts with future Next.js file conventions
