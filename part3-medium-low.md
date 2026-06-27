# SEDNICON Project Audit — Part 3 of 4
## Medium & Low Severity Issues

### Architecture
**[arch-4] Brand SVGs as Inline String Literals** (medium)
File: api/render.js
BRAND_ICONS object contains full SVG markup as JS string literals. Bloats edge function. Any SVG edit needs a code deploy.
Fix: Store brand SVGs in Supabase or a static asset bucket. Load at build time or cache on first request.

**[arch-6] No Build Pipeline** (medium)
package.json has zero dependencies — no bundler, transpiler, or minifier. Raw files deployed to Vercel as-is.
Fix: Add Vite. Enables minification, tree-shaking, code splitting, modern JS/CSS.

### API
**[api-3] Weak Color Validation** (medium)
File: api/render.js
Regex `^[0-9a-fA-F]{3,8}$` allows lengths 3,4,5,6,7,8. Valid CSS hex is only 3, 4, 6, or 8. Length 5 and 7 are always invalid — no error given.
Fix: Validate specifically for 3/4/6/8 chars. Return 400 for invalid colors.

**[api-5] ?set= Allows Arbitrary Iconify Prefix** (medium)
File: api/render.js
User can specify any Iconify collection prefix, bypassing curation.
Fix: Whitelist allowed prefixes. Validate against Iconify's published collection list.

**[api-7] No API Versioning** (medium)
File: vercel.json
Docs mention /api/v1/render as a future path but no version prefix exists. Any breaking change hits all consumers at once.
Fix: Add /v1/ prefix now. Redirect unversioned path. Do this before gaining wide adoption.

**[api-8] Dead Code: ai:{uuid} Strategy Has No UI Entry Point** (low)
File: api/render.js
API checks for q.startsWith('ai:') but no UI page constructs URLs with the ai: prefix. generate.html is a stub.
Fix: Complete the AI generation flow end-to-end, or remove the dead code and add a feature flag.

### Security
**[sec-2] No Content Security Policy Headers** (medium)
File: api/render.js
No CSP, X-Content-Type-Options, or other security headers.
Fix: Add CSP, X-Content-Type-Options: nosniff, X-Frame-Options: DENY, Referrer-Policy: strict-origin-when-cross-origin.

**[sec-4] AI Key Handling Claim Unverified** (medium)
Homepage claims "we never store it unencrypted" but no code in repo shows how keys are handled.
Fix: Publish key handling code. Use client-side encryption (encrypt in browser before sending). Use Supabase Vault.

### UI / UX
**[ui-5] Profile & Icon Pages: Permanent Loading States** (medium)
profile.html shows "Loading profile..." and icon.html shows "Loading icon..." — no actual content ever resolves.
Fix: Add 3-5 second timeout → error state with retry. Use skeleton animations.

**[ui-6] No 404 Page** (medium)
File: vercel.json
vercel.json defines rewrites for 7 routes but no catch-all 404. Invalid URLs show raw Vercel 404.
Fix: Add 404.html, configure Vercel to use it.

**[ui-7] No Responsive Design Validation** (medium)
No evidence of mobile testing. Icon generator, docs tables, and library browser likely break on mobile.
Fix: Viewport meta tags, test 375px-768px widths, responsive breakpoints.

**[ui-8] Missing OG/SEO Meta Tags** (medium)
No og:image, og:title, Twitter Card tags, or JSON-LD structured data.
Fix: Add complete meta tags, og:image with branded preview.

**[ui-10] Inconsistent Navigation Across Pages** (medium)
Each HTML file implements its own nav bar independently — styling, link order, active states may drift.
Fix: Shared nav component. Highlight active page consistently.

**[ui-9] No Loading Animations / Skeleton States** (low)
Static "Loading..." text makes app feel frozen rather than actively loading.
Fix: Shimmer skeleton animations, progress indicators for API calls.

### Code Quality
**[code-1] No TypeScript — Zero Type Safety** (medium)
File: api/render.js
All plain JS with no type annotations. The color validation bug (api-3) would have been caught by TypeScript.
Fix: Migrate to TypeScript. Start with API handler. Enable strict + noImplicitAny.

**[code-3] No Linting or Formatting Config** (low)
No ESLint, no Prettier. Code style depends on discipline.
Fix: Add ESLint + Prettier. Pre-commit hooks with lint-staged.

**[code-4] No CI/CD Pipeline** (medium)
No GitHub Actions, no CI config. Deploys are manual pushes.
Fix: GitHub Actions for lint/test/build. Vercel preview deployments for PRs. Branch protection rules.

**[code-5] CSS/JS/HTML Mixed in Single Files** (medium)
Inline `<style>` and `<script>` in every HTML page. Styles/scripts duplicated across all pages.
Fix: Extract shared CSS. Move JS to separate modules. Use a build tool.

### Documentation
**[doc-1] README API Table is Misformatted** (low)
Markdown table has single header row concatenating all columns — doesn't render as a table on GitHub.
Fix: Proper pipe-separated columns with aligned headers.

**[doc-2] Referenced OpenAPI Spec Doesn't Exist in Repo** (medium)
Docs page links to /openapi.json but no such file exists in the repository.
Fix: Create and commit openapi.json. Keep it version-controlled and validate with spectral.

**[doc-3] Contradictory Performance Claims** (medium)
README claims "<50ms latency worldwide" while the fallback chain makes 16+ sequential HTTP requests.
Fix: Be honest about cache-hit vs. cache-miss latencies. Document expected response times by region.

**[doc-4] No CONTRIBUTING.md or Code of Conduct** (low)
README says "open a PR" but no contributing guide, PR template, or code of conduct exists.
Fix: Add CONTRIBUTING.md + CODE_OF_CONDUCT.md.
