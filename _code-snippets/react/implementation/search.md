# Search

1. Use `useEffect`
   - Dependency on search text
   - Debouncing can also ensure the search function is not called too frequently, which is particularly important for performance when dealing with large datasets or server-side searching.
   - Separation of concerns: Keeps the logic for filtering data separate from the rendering logic
2. Controlled component with direct state management
   - Handle the search logic directly within the event handler of the input field
   - Can be more straightforward for simple searches but cumbersome for complex scenarios
3. Custom hooks
   - Abstract search logic into custom hook for more complex logic or to reuse search functionality across components.
4. Server-Side Searching
   - Often more efficient to perform search queries on the server side when dealing with large datasets
5. Using libraries or frameworks
   - For complex data structures or additional features like sorting and pagination in addition to searching and filtering
6. Memoization
   - Use `useMemo` to avoid unnecessary recalculations if the search operation is computationally expensive.
