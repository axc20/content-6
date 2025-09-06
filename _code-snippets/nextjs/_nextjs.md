# NextJS

- [NextJS Boilerplate](https://github.com/ixartz/Next-js-Boilerplate)
- can absolutely use NextJS as just a frontend
  - `/api` is optional for creating routes that are not react pages
  - all you have to do to use your backend is just make async fetch calls either on the server in `getServerSideProps` for SSR or do API communication client side
- [short polling (Next 13 Server Component)](https://stackoverflow.com/questions/75955126)
  - Server components are not designed to maintain a connection with client. To implement short polling, you have to convert your server component to a client component. The client component should use [react query](https://tanstack.com/query/v4/docs/react/guides/initial-query-data) or [swr](https://swr.vercel.app/docs/with-nextjs#pre-rendering-with-default-data).
