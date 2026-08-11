---
date: 2026-06-17
type: BUG
tech: [Next.js, PM2, aaPanel, Nginx]
repos: [www.sarasavi.lk]
tags: [nextjs, pm2, deploy, permissions, 502-error, frontend]
status: final
---

# Root-Owned Next.js Build Caused 502 And Client Crash

## Problem

`https://sarasavi.org` returned `502 Bad Gateway`, then later showed a client-side application error.

## Context / Reproduction

Service impact:

```text
curl 127.0.0.1:3002 -> Connection refused
PM2 process -> restart loop
Browser -> TypeError: Cannot read properties of undefined
```

The issue caused intermittent site downtime for about 18 hours.

## What I Tried

Checked Nginx, PM2, local port `3002`, `.next` ownership, frontend API responses, and React component handling of empty API data.

## Root Cause

Primary cause:

`.next` was deleted during debugging and rebuilt as `root`. PM2 runs the Next.js process as `www`, so it could not read root-owned build artifacts. The process exited immediately, PM2 restart-looped, port `3002` refused connections, and Nginx returned 502.

Secondary cause:

Frontend components had missing null checks. After API instability, code attempted operations like `data.map()` on undefined values, causing the page to blank with a client-side exception.

## Fix

Rebuild as the runtime user and restart the correct PM2 process.

```bash
cd /www/wwwroot/www.sarasavi.lk

chown -R www:www .
sudo -H -u www npm run build
sudo -u www PM2_HOME=/home/www/.pm2 pm2 restart sarasavi-web-org
sudo -u www PM2_HOME=/home/www/.pm2 pm2 save
```

Defensive frontend pattern:

```javascript
const [products, setProducts] = useState([]);

useEffect(() => {
  axios.get('/api/get-all-products')
    .then((res) => setProducts(res.data?.data?.data || res.data?.data || []))
    .catch(() => setProducts([]));
}, []);

return products?.map((product) => <Card key={product.id} {...product} />);
```

## Verification

```bash
ls -la .next | head -2
curl -I https://sarasavi.org
sudo -u www PM2_HOME=/home/www/.pm2 pm2 logs sarasavi-web-org --lines 5
```

Expected:

```text
.next owned by www:www
HTTP/2 200
sarasavi-web-org online
```

## Lesson

Build Next.js artifacts as the same user that runs PM2, and default frontend list state to `[]` so failed API responses do not crash the page.

## Impact

Restored the storefront and documented both the server-side build ownership fix and the frontend null-safety pattern needed to prevent full-page crashes.

## Resume / Interview Bullet

Resolved a production Next.js outage by tracing PM2 crash loops to root-owned build artifacts, rebuilding as the runtime user, and hardening frontend data handling against failed API responses.
