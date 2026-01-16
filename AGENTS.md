# Hono Documentation for Agents

**Version:** 4.11.4

## Overview

Hono is a fast, lightweight web framework designed to run on any JavaScript runtime. It aims to work on Cloudflare Workers, Fastly Compute, Deno, Bun, Vercel, AWS Lambda, and more.

## Core Features

- **Ultra Fast**: Uses a RegExpTree-based router, making it extremely fast.
- **Lightweight**: The `hono/tiny` preset is very small and has no dependencies.
- **Multi-runtime**: Works on Cloudflare Workers, Fastly Compute, Deno, Bun, Vercel, Netlify, AWS Lambda, Lambda@Edge, and Node.js.
- **Middleware**: Provides a rich set of built-in and custom middleware.
- **Developer Experience**: Supports TypeScript for a comfortable development experience.

---

## Adapters

Hono provides adapters to easily run your application on various platforms.

### `aws-lambda`
Adapter for AWS Lambda and API Gateway.

```typescript
import { Hono } from 'hono'
import { handle } from 'hono/aws-lambda'

const app = new Hono()
app.get('/', (c) => c.text('Hello AWS Lambda!'))

export const handler = handle(app)
```

### `bun`
Adapter for Bun's native HTTP server.

```typescript
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('Hello Bun!'))

export default {
  port: 3000,
  fetch: app.fetch,
}
```

### `cloudflare-pages`
Adapter for Cloudflare Pages Functions.

```typescript
// functions/[[path]].ts
import { Hono } from 'hono'
import { handle } from 'hono/cloudflare-pages'

const app = new Hono().basePath('/functions')
app.get('/hello', (c) => c.json({ message: 'Hello Cloudflare Pages!' }))

export const onRequest = handle(app)
```

### `cloudflare-workers`
Adapter for Cloudflare Workers.

```typescript
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('Hello Cloudflare Workers!'))

export default app
```

### `deno`
Adapter for Deno and Deno Deploy.

```typescript
import { Hono } from 'hono'
import { serve } from 'hono/deno'

const app = new Hono()
app.get('/', (c) => c.text('Hello Deno!'))

serve(app)
```

### `lambda-edge`
Adapter for AWS Lambda@Edge.

```typescript
import { Hono } from 'hono'
import { handle } from 'hono/lambda-edge'

const app = new Hono()
app.get('/', (c) => c.text('Hello Lambda@Edge!'))

export const handler = handle(app)
```

### `netlify`
Adapter for Netlify Functions.

```typescript
// netlify/functions/api.ts
import { Hono } from 'hono'
import { handle } from 'hono/netlify'

const app = new Hono().basePath('/.netlify/functions/api')
app.get('/hello', (c) => c.json({ message: 'Hello Netlify!' }))

export const handler = handle(app)
```

### `node.js`
Starts a server in a Node.js environment using `hono/node-server`.

```typescript
import { Hono } from 'hono'
import { serve } from '@hono/node-server'

const app = new Hono()
app.get('/', (c) => c.text('Hello Node.js!'))

serve({
  fetch: app.fetch,
  port: 3000,
})
```

### `vercel`
Adapter for Vercel Serverless Functions.

```typescript
// api/index.ts
import { Hono } from 'hono'
import { handle } from 'hono/vercel'

export const config = {
  runtime: 'edge',
}

const app = new Hono().basePath('/api')
app.get('/hello', (c) => c.json({ message: 'Hello Vercel!' }))

export default handle(app)
```

---

## Middleware

Hono provides a number of reusable middleware to handle requests and responses.

### `basicAuth`
Provides Basic Authentication.

```typescript
import { Hono } from 'hono'
import { basicAuth } from 'hono/basic-auth'

const app = new Hono()

app.use('/admin/*', basicAuth({
  username: 'admin',
  password: 'password',
}))

app.get('/admin/dashboard', (c) => c.text('Welcome, admin!'))
```

### `bearerAuth`
Provides Bearer Token Authentication.

```typescript
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'

const app = new Hono()

const token = 'my-secret-token'

app.use('/api/*', bearerAuth({ token }))

app.get('/api/users', (c) => c.json([{ id: 1, name: 'John Doe' }]))
```

### `bodyLimit`
Limits the size of the request body.

```typescript
import { Hono } from 'hono'
import { bodyLimit } from 'hono/body-limit'

const app = new Hono()

app.use('/upload', bodyLimit({
  maxSize: 1024 * 1024, // 1MB
  onError: (c) => c.text('Request body is too large', 413),
}))

app.post('/upload', (c) => c.text('File uploaded!'))
```

### `cache`
Caches responses using the Cache API.

```typescript
import { Hono } from 'hono'
import { cache } from 'hono/cache'

const app = new Hono()

app.get(
  '/posts/*',
  cache({
    cacheName: 'my-app-cache',
    cacheControl: 'max-age=3600', // 1 hour
  })
)

app.get('/posts/:id', (c) => {
  const { id } = c.req.param()
  return c.text(`Post content for ID: ${id}`)
})
```

### `compress`
Compresses the response body (gzip, deflate, brotli).

```typescript
import { Hono } from 'hono'
import { compress } from 'hono/compress'

const app = new Hono()

app.use('*', compress())

app.get('/', (c) => c.text('This content will be compressed.'))
```

### `cors`
Manages CORS (Cross-Origin Resource Sharing) headers.

```typescript
import { Hono } from 'hono'
import { cors } from 'hono/cors'

const app = new Hono()

app.use('/api/*', cors({
  origin: 'https://example.com',
  allowMethods: ['GET', 'POST'],
}))

app.get('/api/data', (c) => c.json({ message: 'CORS enabled!' }))
```

### `csrf`
Provides protection against CSRF (Cross-Site Request Forgery) attacks.

```typescript
import { Hono } from 'hono'
import { csrf } from 'hono/csrf'

const app = new Hono()

app.use('*', csrf())

app.get('/', (c) => c.html('<form method="post"><button>Submit</button></form>'))
app.post('/', (c) => c.text('CSRF token is valid!'))
```

### `etag`
Generates an ETag header and returns a `304 Not Modified` response.

```typescript
import { Hono } from 'hono'
import { etag } from 'hono/etag'

const app = new Hono()

app.use('*', etag())

app.get('/data', (c) => c.text('This response has an ETag.'))
```

### `ipRestriction`
Restricts access based on IP address.

```typescript
import { Hono } from 'hono'
import { ipRestriction } from 'hono/ip-restriction'

const app = new Hono()

app.use('/admin/*', ipRestriction({
  allow: ['192.168.0.1', '127.0.0.1'],
}))

app.get('/admin/secret', (c) => c.text('You are allowed!'))
```

### `jsxRenderer`
Renders JSX components on the server side.

```typescript
import { Hono } from 'hono'
import { jsxRenderer } from 'hono/jsx-renderer'

const app = new Hono()

app.get(
  '*',
  jsxRenderer(({ children }) => (
    <html>
      <body>{children}</body>
    </html>
  ))
)

app.get('/', (c) => {
  return c.render(<h1>Hello JSX!</h1>)
})
```

### `jwt`
Performs authentication with JWT (JSON Web Token).

```typescript
import { Hono } from 'hono'
import { jwt } from 'hono/jwt'

const app = new Hono()
const secret = 'my-secret-key'

app.use('/auth/*', jwt({ secret }))

app.get('/auth/profile', (c) => {
  const payload = c.get('jwtPayload')
  return c.json(payload)
})
```

### `logger`
Logs request and response information.

```typescript
import { Hono } from 'hono'
import { logger } from 'hono/logger'

const app = new Hono()

app.use('*', logger())

app.get('/', (c) => c.text('Logged!'))
```

### `prettyJson`
Formats JSON responses for readability.

```typescript
import { Hono } from 'hono'
import { prettyJson } from 'hono/pretty-json'

const app = new Hono()

app.use('*', prettyJson())

app.get('/user', (c) => c.json({ id: 1, name: 'John Doe', active: true }))
```

### `secureHeaders`
Automatically adds security-related HTTP headers.

```typescript
import { Hono } from 'hono'
import { secureHeaders } from 'hono/secure-headers'

const app = new Hono()

app.use('*', secureHeaders())

app.get('/', (c) => c.text('Secure headers are set!'))
```

### `serveStatic`
Serves static files.

```typescript
import { Hono } from 'hono'
import { serveStatic } from 'hono/serve-static'

const app = new Hono()

// Set the root directory to `assets`
app.use('/static/*', serveStatic({ root: './assets' }))
// e.g., /static/style.css -> ./assets/style.css

// Specify a file path
app.get('/favicon.ico', serveStatic({ path: './assets/favicon.ico' }))
```

### `timing`
Measures processing time with the `Server-Timing` header.

```typescript
import { Hono } from 'hono'
import { timing } from 'hono/timing'

const app = new Hono()

app.use('*', timing())

app.get('/', async (c) => {
  await new Promise(resolve => setTimeout(resolve, 100))
  return c.text('Timing header is set.')
})
```

### `trailingSlash`
Unifies the trailing slash of a URL.

```typescript
import { Hono } from 'hono'
import { trailingSlash } from 'hono/trailing-slash'

const app = new Hono()

// Add a trailing slash if it's missing
app.use('*', trailingSlash({
  action: 'add'
}))

app.get('/path/', (c) => c.text('Path with slash')) // Also accessible via /path
```

---

## Helpers

A collection of useful helper functions to aid development.

### `cookie`
Easily get and set cookies.

```typescript
import { Hono } from 'hono'
import { getCookie, setCookie } from 'hono/cookie'

const app = new Hono()

app.get('/set', (c) => {
  setCookie(c, 'my-cookie', 'hello hono', {
    path: '/',
    secure: true,
    httpOnly: true,
    maxAge: 3600,
  })
  return c.text('Cookie set!')
})

app.get('/get', (c) => {
  const cookie = getCookie(c, 'my-cookie')
  return c.text(`Cookie value: ${cookie}`)
})
```

### `html`
Provides HTML escaping and template literals.

```typescript
import { Hono } from 'hono'
import { html } from 'hono/html'

const app = new Hono()

app.get('/', (c) => {
  const unsafeScript = '<script>alert("xss")</script>'
  return c.html(html`
    <h1>Safe HTML</h1>
    <p>${unsafeScript}</p>
  `)
})
```

### `streaming`
Assists in implementing streaming APIs (SSE, ReadableStream).

```typescript
import { Hono } from 'hono'
import { stream, streamText, streamSSE } from 'hono/streaming'

const app = new Hono()

// Server-Sent Events (SSE)
app.get('/sse', (c) => {
  return streamSSE(c, async (stream) => {
    for (let i = 0; i < 5; i++) {
      await stream.writeSSE({
        data: `Message ${i}`,
        event: 'message',
        id: String(i),
      })
      await stream.sleep(1000)
    }
  })
})
```

### `testing`
Makes it easy to test the application.

```typescript
import { hc } from 'hono/testing'
import { Hono } from 'hono'

const app = new Hono()
app.get('/hello', (c) => c.json({ message: 'Hello!' }))

describe('My App', () => {
  it('should return Hello!', async () => {
    const res = await hc(app).hello.$get()
    expect(res.status).toBe(200)
    expect(await res.json()).toEqual({ message: 'Hello!' })
  })
})
```
