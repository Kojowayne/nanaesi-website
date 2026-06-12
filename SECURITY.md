# Security Setup Notes

This site is a static HTML page with no backend, forms, or user input —
which already removes most common attack surfaces (SQL injection, server-side
code execution, auth bypass, etc.). The hardening below covers what's left:
transport security, browser-enforced policies, and safe external linking.

## 1. HTTPS / TLS
Handled automatically if hosted on GitHub Pages, Netlify, Cloudflare Pages,
or Vercel — all issue free TLS certs and redirect HTTP to HTTPS by default.
No action needed unless self-hosting.

## 2. Security Headers (_headers file)
The `_headers` file in this folder is Netlify's config format for setting
HTTP response headers. Drop it in the same directory as `index.html` when
deploying to Netlify and it applies automatically.

What each header does:
- **Content-Security-Policy (CSP)**: Restricts where resources (scripts,
  styles, images, frames) can load from. Currently set to 'self' only,
  meaning only this site's own files — blocks injected third-party scripts.
- **X-Frame-Options: DENY**: Prevents the page from being embedded in an
  iframe on another site (clickjacking protection).
- **X-Content-Type-Options: nosniff**: Stops browsers from guessing file
  types, which can prevent certain MIME-based attacks.
- **Referrer-Policy**: Limits how much of your URL is leaked to other sites
  when users click outbound links.
- **Permissions-Policy**: Explicitly disables camera, mic, and geolocation
  APIs since this site doesn't need them.
- **Strict-Transport-Security (HSTS)**: Tells browsers to always use HTTPS
  for this domain, even if a user types "http://".

If hosting on GitHub Pages or Cloudflare Pages instead of Netlify, these
headers need to be configured differently:
- GitHub Pages doesn't support custom headers directly — you'd need
  Cloudflare in front of it as a reverse proxy.
- Cloudflare Pages: use a `_headers` file too (same format, supported
  natively) — or configure via Cloudflare's dashboard "Transform Rules".

## 3. In-page meta tags (already added to index.html)
A `<meta http-equiv="Content-Security-Policy">` tag is included as a
fallback CSP for hosts that don't support custom headers (like basic
GitHub Pages). Note: meta-tag CSP can't set frame-ancestors or set HSTS —
those only work via real HTTP headers, hence the _headers file above.

## 4. External link safety
All `target="_blank"` links (Calendly, TikTok) include
`rel="noopener noreferrer"` — prevents the linked page from accessing
`window.opener` (a known tab-hijacking vector) and avoids leaking referrer
info.

## 5. Account-level security (the actual weak point)
The code can be locked down perfectly and still get compromised via:
- Weak passwords / no 2FA on GitHub, Netlify, or Calendly accounts —
  enable 2FA on all of these.
- Domain registrar account (if a custom domain is purchased later) — enable
  registrar lock + 2FA to prevent domain hijacking.

## 6. Testing your setup
Once deployed, test the headers with:
- https://securityheaders.com (paste your live URL)
- https://observatory.mozilla.org (Mozilla's security scanner)

Both give a letter grade and explain anything missing.
