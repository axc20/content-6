# API endpoints

Any file inside the `pages/api` folder will be treated as a `/api/*` endpoint.

```js
// pages/api/user.js
// request handler:
// req - http.IncomingMessage
// res - http.ServerResponse
export default function handler(req, res) {
  res.status(200).json({ name: 'John Doe' });
}

// to handle different HTTP methods in an API route:
export default function handler(req, res) {
  if (req.method === 'POST') {
    // process a POST request
  } else {
    // handle any other HTTP method
  }
}
```

## Dynamic routes

```js
// pages/api/post/[pid].js
export default function handler(req, res) {
  const { pid } = req.query;
  res.send(`Post: ${pid}`);
}
```

Common RESTful pattern:

- `GET api/posts`: get a list of posts, probably paginated
- `GET api/posts/12345`: get post id 12345

Implementing in NextJS:

1. `/api/posts.js` + `/api/posts/[postId].js`
2. `/api/posts/index.js` + `/api/posts/[postId].js`

## Catch all routes

`pages/api/post/[...slug].js` matches `/api/post/a`, `/api/post/a/b`, `/api/post/a/b/c`, etc. The query parameter sent will be `{ "slug": ["a"] }`, `{ "slug": ["a", "b"] }`, etc.

Optional catch all, `pages/api/post/[[...slug]].js`, will match `/api/post`, `/api/post/a`, `/api/post/a/b`, etc.

## Caveats

Precedence: predefined routes > dynamic routes > catch all routes

## Built-in middlewares (parse incoming req)

- `req.cookies`: an object containing cookies sent by the request; default is `{}`
- `req.query`: an object containing the query string; default is `{}`
- `req.body`: an object containing the body parsed by `content-type` or `null` if no body was sent

## Response helpers

- `res.status(code)`: code must be valid HTTP status code
- `res.json(body)`: sends JSON response
- `res.send(body)`: sends HTTP response; can be string, object, or Buffer
- `res.redirect([status,] path)`: redirect to specified path or URL; if not specified, `status` defaults to "307" - "Temporary redirect"

```js
export default async function handler(req, res) {
  try {
    const result = await someAsyncOperation();
    res.status(200).send({ result });
  } catch (err) {
    res.status(500).send({ error: 'failed to fetch data' });
  }
}
```
