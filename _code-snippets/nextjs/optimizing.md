# Optimizing

- Analytics
- OpenTelemetry
- Instrumentation

## Images

## Fonts

## Scripts

```tsx
// app/dashboard/layout.tsx
import Script from 'next/script';

export default function DashboardLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <>
      <section>
      {/* Load a third-party script (for multiple routes) */}
      {/* Next.js will ensure the script will only load once */}
      <Script src="https://example.com/script.js" />
    </>
  );
};
```

```tsx
// app/layout.tsx
import Script from 'next/script';

export default function RootLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
      {/* Load a third-party script for all routes */}
      {/* Recommend only including third-party scripts in specific */}
      {/* pages or layours to minimize impact to performance */}
      <Script src="https://example.com/script.js" />
    </html>
  );
}
```

Fine tune loading behavior with the `strategy` property:

- `beforeInteractive`: Load script before any Next.js codde and before any page hydration occurs
- `afterInteractive` (default): Load script early but after some hydration on the page occurs
- `lazyOnload`: Load script later during browser idle time
- `worker` (experimental): Load script in a web worker

### Inline Scripts

```tsx
<Script id="show-banner">
  {`document.getElementById('banner').classList.remove('hidden')`}
</Script>

<Script
  id="show-banner"
  dangerouslySetInnerHTML={{
    __html: `document.getElementById('banner').classList.remove('hidden')`
  }}
/>
```

### Executing Additional Code

```tsx
'use client';

import Script from 'next/script';

export default function Page() {
  return (
    <>
      <Script
        src="https://example.com/script.js"
        // onLoad: execute code after script has finished loading
        onLoad={() => {
          console.log('Script has loaded');
        }}
        // onReady: execute code after script has finished loading
        // and every time the component is mounted
        // onError: execute code if script fails to load
      />
    </>
  );
}
```

## Metadata

Next.js has a Metadata API that can be used to define your application metadata (e.g. `meta` and `link` tags inside your HTML `head`) for improved SEO and web shareability.

- Both static and dynamic metadata through `generateMetadata` are only supported in Server Components.
- When rendering a route, Next.js will automatically dedup fetch requests for the same data across `generateMetadata`, `generateStaticParams`, Layouts, Pages, and Server Components. React cache can be used if fetch is unavailable.
- Next.js will wait for data fetching inside `generateMetadata` to complete before streaming UI to the client. This guarantees the first part of a streamed response includes `<head>` tags.

### Static Metadata

```tsx
// layout.tsx / page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: '...',
  description: '...'
};

export default function Page() {}
```

### Dynamic Metadata

```tsx
// app/products/[id]/page.tsx
import { Metadata, ResolvingMetadata } from 'next';

type Props = {
  params: { id: string };
  searchParams: { [key: string]: string | string[] | undefined };
};

export async function generateMetadata(
  { params, searchParams }: Props,
  parent?: ResolvingMetadata
): Promise<Metadata> {
  // read route params
  const id = params.id;

  // fetch data
  const product = await fetch('...').then(res => res.json());

  // optionally access and extend (rather than replace) parent metadata
  const previousImages = (await parent).openGraph?.images || [];

  return {
    title: product.title,
    openGraph: {
      images: ['/some-specific-page-image.jpg', ...previousImages]
    }
  };

  export default function Page({ params, searchParams }: Props) {}
}
```

### File-based metadata

### Behavior

### Dynamic Image Generation

### JSON-LD

[JSON-LD](https://json-ld.org/) is a format for structured data that can be used by search engines to understand your content. For example, you can use it to describe a person, an event, an organization, a movie, a book, a recipe, and many other types of entities.

Current recommendation for JSON-LD is to render structured data as a `<script>` tag in your `layout.js` or `page.js` components:

```tsx
// app/product/[id]/page.tsx
export default async function Page({ params }) {
  const product = await getProduct(params.id);
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    image: product.image,
    description: product.description
  };

  return (
    <section>
      {/* Add JSON-LD to your page */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      {/* ... */}
    </section>
  );
}
```

- Can validate and test your structured data with the [Rich Results Test](https://search.google.com/test/rich-results) for Google or the generic [Schema Markup Validator](https://validator.schema.org/)
- Can type your JSON-LD with TypeScript using `scheam-dts`

```tsx
import { Product, WithContext } from 'schema-dts';

const jsonLd: WithContext<Product> = {
  '@context': 'https://schema.org',
  '@type': 'Product',
  name: 'Next.js Sticker',
  image: 'https://nextjs.org/imgs/sticker.png',
  description: 'Dynamic at the speed of static.'
};
```

## Statc Assets

## Lazy Loading
