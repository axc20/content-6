# Rendering

## Server Components

Server and Client Components allow developers to build applications that span the server and client, combining the rich interactivity of client-side apps with the improved performance of traditional server rendering.

Consider a page in your application: navbar (with search), sidebar, main (with buttons). From these smaller components, you'll notice the majority are non-interactive and can be rendered on the server as Server Components. For smaller pieces of interactive UI (such as the search or buttons in our example), we can sprinkle in Client Components. Next.js is a server-first approach.

With Server components, you can move data fetching to the server (closer to your database), and keep large dependencies on the server (improving performance). The inital page load is faster, and the client-side JS bundle size is reduced. The base client-side runtime is cacheable and predictable in size, and does not increase as your application grows. Additional JS is only added as client-side interactivity is used in your application through Client Components.

When a route is loaded with Next.js, the initial HTML is rendered on the server. This HTML is then **progressively enhanced** in the browser, allowing the client to take over and add interactivity, by asynchronously loading the Next.js and React client-side runtime. All components inside the App Router (`/app`) are Server Components by default. You can optionally opt-in to Client Components using the `use client` directive.

## Client Components

In Next.js, Client Components are pre-rendered on the server and hydrated on the client.

```tsx
// convention to declare a boundary between a Server and Client Component module graph
// once defined in a file ("entry point"), all other modules imported into it,
// including child components, are considered part of the client bundle
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

Use Client Components for:

- Adding interactivity and event listeners (`onClick`, `onChange`, etc.)
- Using state and lifecycle effects (`useState()`, `useReducer()`, `useEffect()`, etc.)
- Use browser-only APIs
- Use custom hooks that depend on state, effects, or browser-only APIs

## Patterns

To improve performance, move Client Components to the leaves of your component tree where possible. For example, if you have a Layout that has static elements (logo, links, etc.) and an interactive search bar that uses state, move the interactive logic to a Client Component (e.g. `<SearchBar />`) and keep your layout as a Server Component.

```tsx
// SearchBar is a Client Component
import SearchBar from './searchbar';
// Logo is a Server Component
import Logo from './logo';

// Layout is a Server Component by default
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />
        <SearchBar />
      </nav>
      <main>{children}</main>
    </>
  );
}
```

**Importing Server Components inside Client Components is an unsupported pattern.** Instead, when designing Client Components, you can use React props to mark "slots" for Server Components. The Server Component will be rendered on the server, and when the Client Component is rendered on the client, the "slot" will be filled in with the rendered result of the Server Component.

```tsx
'use client';

import { useState } from 'react';

// A common pattern is to use React `children` prop to create the "slot"
export default function ExampleClientComponent({
  children
}: {
  children: React.ReactNode;
}) {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      {children}
    </>
  );
}
```

```tsx
import ExampleClientComponent from './example-client-component';
import ExampleServerComponent from './example-server-component';

export default function Page() {
  return (
    <ExampleClientComponent>
      <ExampleServerComponent />
    </ExampleClientComponent>
  );
}
```

This pattern is already applied in layouts and pages with the `children` prop. Passing React components (JSX) to other components has always been part of the React composition model. You are not limited to the `children` prop. You can use any prop to pass JSX.

Props passed from the Seerver to Client Components need to be serializable. This means that values such as functions, Dates, etc. cannot be passed directly to Client Components.

### Keeping Server-Only code out of Client Components (Poisoning)

```ts
// lib/data.ts
export async function getData() {
  const res = await fetch('...', {
    headers: {
      // env variables not prefexied with `NEXT_PUBLIC` will only be
      // accesible on server to prevent leaking secure information
      authorization: process.env.API_KEY
    }
  });
}
```

To prevent unintended client usage of server code, the `server-only` package can be used to give build-time errors if a server-only module is imported into a Client Component. The corresponding `client-only` package can be used to do the reverse.

### Context

Context is fully supported within Client Components, but it cannot be created or consumed directly within Server Components. Server Components have no React state (since they're not interactive), and context is primarily used for rerendering interactive components deep in the tree after some React state has been updated.

```tsx
// app/sidebar.tsx
'use client';

import { createContext, useContext, useState } from 'react';

const SidebarContext = createContext();

export function Sidebar() {
  const [isOpen, setIsOpen] = useState();

  return (
    <SidebarContext.Provider value={{ isOpen }}>
      <SideBarNav />
    </SidebarContext.Provider>
  );
}

function SidebarNav() {
  let { isOpen } = useContext(SidebarContext);

  return (
    <div>
      <p>Home</p>
      {isOpen && <Subnav />}
    </div>
  );
}
```

```tsx
// app/theme-provider.tsx
'use-client';

import { createContext } from 'react';

export const ThemeContext = createContext({});

export default function ThemeProvider({ children }) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>;
}
```

```tsx
// app/layout.tsx
import ThemeProvider from './theme-provider';

export default function RootLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

### Sharing data between Server Components

Use native JS patterns (global singletons) for common data that multiple Server Components need to access. For example, a module can be used to share a database connection across multiple components:

```ts
// utils/database.ts
export const db = new DatabaseConnection();

// app/users/layout.tsx
import { db } from '@utils/database';

export async function UsersLayout() {
  let users = await db.query();
  // ...
}

// app/users/[id]/page.tsx
import { db } from '@utils/database';

export async function DashboardPage() {
  let user = await db.query();
  // ...
}
```

---

## Static and Dynamic Rendering on the Server

With **Static Rendering**, both Server and Client Components can be prerendered on the server at **build time**. The result of the work is cached and reused on subsequent requests. The cached result can also be revalidated.

With **Dynamic Rendering**, both Server and Client Components are rendered on the server at **request time**. The result of the work is not cached.

## Edge and Node.js Runtimes

On the server, there are two runtimes where your pages can be rendered:

- The **Node.js Runtime** (default) has access to all Node.js APIs and compatible packages from the ecosystem.
- The **Edge Runtime** is based on Web APIs.

Both runtimes support streaming from the server, depending on your deployment infrastructure.
