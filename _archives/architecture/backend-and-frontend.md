# Backend and Frontend

A backend request lifecycle looks roughly like:

- Authentication (who you are)
- Authorization (what you're allowed to do)
- Request deserialization (implicit in NodeJS / dynamic languages)
- Request validation
- Retrieve relevant domain objects
- Business rules
- Side effects (not just storing in DB, but sending email, calling other external service, etc.)
- Map data model to API response

These tasks are the same no matter how the backend and frontend communicate, whether they use technologies like GraphQL, REST, or gRPC.

Why backend and frontend should be separate (especially true for bigger projects):

- Frontend cannot be trusted
- Better clarity and easier management
- Can keep API design as a first-class citizen (important for backwards compatibility)
- Helpful if more ways to interact with backend are needed (like mobile experience, for example)

It is ok to use NextJS's backend API as a middleman (BFF - Backend For Frontend) between the frontend and other services in a big business environment. There should still be a separate, main backend, and this middleman should follow certain rules like not accessing the database directly. This pattern only makes sense in large software ecosystems.
