# SEDNICON Project Audit — Part 4 of 4
## Priority Action Plan & GitHub Strategy

---

## Prioritized Fix Roadmap

### 🔴 Immediate (before next deploy)
1. Sanitize SVG content from Supabase — XSS risk
2. Replace SUPABASE_SERVICE_KEY with anon key + RLS
3. Add rate limiting (Vercel KV / Upstash)
4. Fix sequential Iconify fallback — parallelize or use search API

### 🟠 Short-term (1–2 sprints)
1. Add server-side caching (Vercel KV) to achieve real <50ms
2. Fix broken UI pages: community.html template literals, populate library.html, build generate.html
3. Return proper 404/400 status codes with JSON error bodies
4. Add unit + integration tests (vitest)

### 🟡 Medium-term (next quarter)
1. Migrate frontend to Next.js, Astro, or Vite+React
2. Add TypeScript to api/render.js first
3. Implement /v1/ API versioning
4. Add CSP and security headers
5. Fix color validation regex (allow only 3/4/6/8 char hex)
6. Add OG/SEO meta tags, fix README table
7. Add CI/CD via GitHub Actions

---

## GitHub Repo Strategy for Arena.ai

To let arena.ai fetch and apply these fixes directly, create a **clean public repo** from a separate GitHub account containing only the source files it needs to modify.

### What to put in the repo
```
sednicon-refactor/
├── README.md          ← include this audit summary as context
├── api/
│   └── render.js      ← the actual edge function to be fixed
├── vercel.json
└── ISSUES.md          ← paste Part 1-3 of this audit
```

### ISSUES.md format for arena.ai
Structure it so the model can parse tasks clearly:

```markdown
## Task
Refactor the SEDNICON icon API. Fix the issues below in priority order.
Work only on files listed in this repo. Do not invent new dependencies 
without justification.

## Source file: api/render.js
[paste the actual render.js content here]

## Issues to fix (in order)
[paste Part 1 critical issues]
[paste Part 2 high issues]
```

### Tips for feeding arena.ai via GitHub
- Keep render.js as the **only** source file in scope — arena.ai can fetch its raw URL directly:
  `https://raw.githubusercontent.com/YOUR_ORG/sednicon-refactor/main/api/render.js`
- Put the audit issues in ISSUES.md and give arena.ai that raw URL too
- In your arena.ai prompt: *"Fetch [render.js URL] and [ISSUES.md URL], then fix the critical and high issues in order. Output the updated render.js."*
- For each sprint, update ISSUES.md to only include the current batch — keeps context short

### Prompt template for arena.ai
```
Fetch these two files:
- Source: https://raw.githubusercontent.com/[you]/sednicon-refactor/main/api/render.js
- Issues: https://raw.githubusercontent.com/[you]/sednicon-refactor/main/ISSUES.md

Fix ONLY the issues marked 🔴 Critical in ISSUES.md.
Output the complete updated render.js with inline comments 
explaining each change. Do not change behavior for uncovered issues.
```
