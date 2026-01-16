# Hono エージェント向けドキュメント

**バージョン:** 4.11.4

## 概要

Honoは、Web標準API上で動作するように設計された、高速で軽量なWebフレームワークです。Cloudflare Workers、Fastly Compute、Deno、Bun、Vercel、AWS Lambdaなど、あらゆるJavaScriptランタイムで動作することを目指しています。

## コア機能

- **超高速**: ルーターにRegExpTreeを使用しており、非常に高速です。
- **軽量**: `hono/tiny`プリセットは非常に小さく、依存関係もありません。
- **マルチランタイム**: Cloudflare Workers, Fastly Compute, Deno, Bun, Vercel, Netlify, AWS Lambda, Lambda@Edge, Node.jsで動作します。
- **ミドルウェア**: 豊富な組み込みミドルウェアとカスタムミドルウェアを提供します。
- **開発者体験**: TypeScriptをサポートし、快適な開発体験を提供します。

---

## アダプター (Adapters)

Honoは、さまざまなプラットフォームでアプリケーションを簡単に実行するためのアダプターを提供します。

### `aws-lambda`
AWS LambdaおよびAPI Gateway用のアダプター。

```typescript
import { Hono } from 'hono'
import { handle } from 'hono/aws-lambda'

const app = new Hono()
app.get('/', (c) => c.text('Hello AWS Lambda!'))

export const handler = handle(app)
```

### `bun`
BunネイティブのHTTPサーバー用アダプター。

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
Cloudflare Pages Functions用アダプター。

```typescript
// functions/[[path]].ts
import { Hono } from 'hono'
import { handle } from 'hono/cloudflare-pages'

const app = new Hono().basePath('/functions')
app.get('/hello', (c) => c.json({ message: 'Hello Cloudflare Pages!' }))

export const onRequest = handle(app)
```

### `cloudflare-workers`
Cloudflare Workers用アダプター。

```typescript
import { Hono } from 'hono'

const app = new Hono()
app.get('/', (c) => c.text('Hello Cloudflare Workers!'))

export default app
```

### `deno`
DenoおよびDeno Deploy用アダプター。

```typescript
import { Hono } from 'hono'
import { serve } from 'hono/deno'

const app = new Hono()
app.get('/', (c) => c.text('Hello Deno!'))

serve(app)
```

### `lambda-edge`
AWS Lambda@Edge用アダプター。

```typescript
import { Hono } from 'hono'
import { handle } from 'hono/lambda-edge'

const app = new Hono()
app.get('/', (c) => c.text('Hello Lambda@Edge!'))

export const handler = handle(app)
```

### `netlify`
Netlify Functions用アダプター。

```typescript
// netlify/functions/api.ts
import { Hono } from 'hono'
import { handle } from 'hono/netlify'

const app = new Hono().basePath('/.netlify/functions/api')
app.get('/hello', (c) => c.json({ message: 'Hello Netlify!' }))

export const handler = handle(app)
```

### `node.js`
`hono/node-server` を使用してNode.js環境でサーバーを起動します。

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
Vercel Serverless Functions用アダプター。

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

## ミドルウェア (Middleware)

Honoは、リクエストとレスポンスを処理するための再利用可能なミドルウェアを多数提供しています。

### `basicAuth`
基本認証を提供します。

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
Bearerトークン認証を提供します。

```typescript
import { Hono } from 'hono'
import { bearerAuth } from 'hono/bearer-auth'

const app = new Hono()

const token = 'my-secret-token'

app.use('/api/*', bearerAuth({ token }))

app.get('/api/users', (c) => c.json([{ id: 1, name: 'John Doe' }]))
```

### `bodyLimit`
リクエストボディのサイズを制限します。

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
Cache APIを使用してレスポンスをキャッシュします。

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
レスポンスボディを圧縮します (gzip, deflate, brotli)。

```typescript
import { Hono } from 'hono'
import { compress } from 'hono/compress'

const app = new Hono()

app.use('*', compress())

app.get('/', (c) => c.text('This content will be compressed.'))
```

### `cors`
CORS (Cross-Origin Resource Sharing) ヘッダーを管理します。

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
CSRF (Cross-Site Request Forgery) 対策を提供します。

```typescript
import { Hono } from 'hono'
import { csrf } from 'hono/csrf'

const app = new Hono()

app.use('*', csrf())

app.get('/', (c) => c.html('<form method="post"><button>Submit</button></form>'))
app.post('/', (c) => c.text('CSRF token is valid!'))
```

### `etag`
ETagヘッダーを生成し、`304 Not Modified`レスポンスを返します。

```typescript
import { Hono } from 'hono'
import { etag } from 'hono/etag'

const app = new Hono()

app.use('*', etag())

app.get('/data', (c) => c.text('This response has an ETag.'))
```

### `ipRestriction`
IPアドレスに基づいてアクセスを制限します。

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
JSXコンポーネントをサーバーサイドでレンダリングします。

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
JWT (JSON Web Token) による認証を行います。

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
リクエストとレスポンスの情報をログに出力します。

```typescript
import { Hono } from 'hono'
import { logger } from 'hono/logger'

const app = new Hono()

app.use('*', logger())

app.get('/', (c) => c.text('Logged!'))
```

### `prettyJson`
JSONレスポンスを整形して出力します。

```typescript
import { Hono } from 'hono'
import { prettyJson } from 'hono/pretty-json'

const app = new Hono()

app.use('*', prettyJson())

app.get('/user', (c) => c.json({ id: 1, name: 'John Doe', active: true }))
```

### `secureHeaders`
セキュリティ関連のHTTPヘッダーを自動で追加します。

```typescript
import { Hono } from 'hono'
import { secureHeaders } from 'hono/secure-headers'

const app = new Hono()

app.use('*', secureHeaders())

app.get('/', (c) => c.text('Secure headers are set!'))
```

### `serveStatic`
静的ファイルを配信します。

```typescript
import { Hono } from 'hono'
import { serveStatic } from 'hono/serve-static'

const app = new Hono()

// ルートディレクトリを `assets` に設定
app.use('/static/*', serveStatic({ root: './assets' }))
// 例: /static/style.css -> ./assets/style.css

// ファイルパスを指定
app.get('/favicon.ico', serveStatic({ path: './assets/favicon.ico' }))
```

### `timing`
`Server-Timing`ヘッダーで処理時間を計測します。

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
URL末尾のスラッシュを統一します。

```typescript
import { Hono } from 'hono'
import { trailingSlash } from 'hono/trailing-slash'

const app = new Hono()

// URLの末尾にスラッシュがなければ追加する
app.use('*', trailingSlash({
  action: 'add'
}))

app.get('/path/', (c) => c.text('Path with slash')) // /path でもアクセス可能
```

---

## ヘルパー (Helpers)

開発を補助するための便利なヘルパー関数群です。

### `cookie`
クッキーの取得と設定を簡単に行います。

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
HTMLエスケープやテンプレートリテラルを提供します。

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
ストリーミングAPI (SSE, ReadableStream) の実装を支援します。

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
アプリケーションのテストを容易にします。

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