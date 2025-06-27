# useSWR

`useSWR` is a React hook for data fetching created by the Vercel team. "SWR" stands for "stale-while-revalidate". This strategy involves:

1. **Initial Fetch**: When you first use `useSWR`, it will fetch the data from the provided endpoint of function.
2. **Return Stale Data**: On subsequent renders or interactions, if the data is already fetched once, SWR will provide the "stale" (cached) data immediately.
3. **Revalidation**: Simulataneously, in the background, SWR will make a new request to refetch the data. This is not polling because it's not doing it at regular intervals. It does so based on certain triggers:
   - When the component is re-rendered
   - When the user focuses on the page (by default, SWR refetches on focus)
   - When the user's device reconnects to the internet
   - When you manually trigger it using the `mutate` function without a data argument
