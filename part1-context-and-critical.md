# SEDNICON Project Audit — Part 1 of 4
## Context & Critical Issues

### What is SEDNICON?
A headless icon API serving dynamic SVGs via URL. Combines Material Design, FontAwesome, Lucide, and brand icons into one endpoint. Stack: Vercel Edge Functions, Iconify API, Supabase, Google Sign-In, Gemini/GPT-4o for AI generation. Frontend is raw HTML/CSS/JS with no framework.

### Repo Structure
```
sednicon/
├── api/render.js     ← single Edge Function (entire backend)
├── index.html, library.html, docs.html
├── status.html, community.html, generate.html
├── profile.html, icon.html
├── package.json      ← zero dependencies
└── vercel.json
```

### Overall Assessment
33 issues across 6 categories. The concept is solid but execution has critical gaps — security vulnerabilities and unfinished UI pages mean it is NOT production-ready.

---

## 🔴 CRITICAL Issues (fix before next deploy)

**[arch-1] Monolithic Edge Function**
File: api/render.js
All logic (brand icons, Iconify routing, AI generation, Supabase, color/size transforms) in one file. Violates separation of concerns. Untestable.
Fix: Split into brandResolver(), iconifyResolver(), aiResolver(), svgTransformer().

**[arch-2] Sequential Fallback Chain = Latency Spikes**
File: api/render.js
Makes up to 16+ sequential HTTP requests to Iconify on cache miss. The "<50ms latency" claim is impossible — one Iconify round-trip from most regions is already 50-200ms.
Fix: Parallel-probe top 3 sets simultaneously, or use Iconify's collection search API to resolve prefix in one call. Add LRU cache.

**[api-1] No Rate Limiting — Open to DDoS**
File: api/render.js
README says "no rate limits for normal use" — literally zero rate-limiting code. Any bot can flood the API.
Fix: Vercel Edge Middleware + Upstash Redis for IP-based rate limiting.

**[api-4] No SVG Sanitization — XSS Vulnerability**
File: api/render.js
AI-generated icons fetched from Supabase are served directly with no sanitization. A malicious SVG with <script> tags or onload handlers would attack every user who loads that icon.
Fix: Sanitize all SVG via DOMPurify or a custom sanitizer before serving.

**[api-6] Supabase Service Key in Edge Function**
File: api/render.js
Uses SUPABASE_SERVICE_KEY (admin-level, bypasses all RLS). One code vulnerability = full database compromise.
Fix: Switch to anon key + Row Level Security policies. Narrow scope if service access is truly needed.
