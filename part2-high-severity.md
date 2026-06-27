# SEDNICON Project Audit — Part 2 of 4
## High Severity Issues

**[arch-3] No Server-Side Caching**
File: api/render.js
Zero server-side caching. Cache-Control headers only help CDN/browser — the edge function re-fetches every time. Every cache miss hits Iconify or Supabase directly.
Fix: Vercel KV (Redis) or Edge Config for hot icons. Even an in-memory Map with TTL helps.

**[arch-5] 7+ Separate HTML Files, No Shared Components**
Pages: index, library, docs, status, community, generate, profile, icon — each duplicates the nav, styles, and scripts.
Fix: Migrate to Next.js, Astro, or Vite+React. Share components via imports, use client-side routing.

**[api-2] Always Returns 200 OK — No Proper Errors**
File: api/render.js
Invalid icon names return a generic fallback SVG with HTTP 200. Clients can't tell real icon from placeholder. Monitoring can't detect failures.
Fix: Return 404 for unknown icons with JSON error body when Accept: application/json. Add ?fallback=false for API clients.

**[sec-1] CORS Wildcard (Access-Control-Allow-Origin: *)**
File: api/render.js
Any site can make requests. Fine for read-only icon CDN, but enables proxying/masking traffic to Iconify.
Fix: Restrict to known origins for write endpoints (AI generation, community posting).

**[sec-3] No Auth on Community Icon Write Endpoints**
Community browsing and publishing have no visible auth enforcement. Anyone could post malicious SVG content.
Fix: Enforce Google Sign-In session for all write ops. Add content moderation + rate limiting for submissions.

**[code-2] Zero Test Coverage**
No test files, no test framework, no CI. The fallback chain, color parsing, and SVG transforms have no automated verification.
Fix: Unit tests for handler, color parser, size validator, SVG transformer. Integration tests for fallback chain. Use vitest.

**[ui-1] Status Page Shows Permanent Placeholder Dashes**
File: status.html
Version, Last Checked, Region all show "—". Four service cards permanently say "Checking".
Fix: Implement /api/status fetch. Show real data with a skeleton loading → content transition.

**[ui-2] Community Page: Broken Template Literal Rendering**
File: community.html
Contains raw JS template literals like `${Array(12).fill(0).map(...)}` that are NOT being evaluated. Icon grid is completely empty.
Fix: Move to proper client-side JS with fetch calls.

**[ui-3] Icon Library Page is Nearly Empty**
File: library.html
Only a header, search bar, stat counter, and a "pro tip" — no icon grid, no categories, no pagination.
Fix: Implement icon browser with category tabs, paginated grid, lazy loading. Connect search to Iconify search API.

**[ui-4] Generate Page is a Stub**
File: generate.html
Only shows "Generate icons with AI" + "Sign in with Google to get started." No form, no text input, no preview.
Fix: Build full flow (prompt → AI provider selection → SVG preview → publish) or remove nav link until ready.
