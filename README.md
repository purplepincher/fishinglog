# fishinglog

> **Real today (✅)** – Worker serves static assets from `./public/`; the served page (`public/index.html`) already inlines the family design‑system CSS (tokens + base) as a manual `<style>` block.
> **Conditional (⚠️)** – The design‑system source files (`family/tokens.css`, `family/base.css`, `family/provenance-panel.css`) exist but have no automated build pipeline to incorporate them into the served page; the current page is hand‑curated.
> **Aspirational (🔮)** – Provenance‑panel component, automation of design‑system inlining across multiple family sites, and any dynamic behaviour beyond static file serving.

---

## 1. What this repository actually is (today)

This is a **Cloudflare Worker** that serves static files from a `./public` directory.  
The Worker entry point is `src/index.ts`; it calls `env.ASSETS.fetch(request)` and returns the response, with fallback 404/500 handling.  
Configuration lives in `wrangler.jsonc` (worker name `fishinglog`, asset binding `ASSETS`).

The repository **also contains** (and the served page already uses) the family design‑system files:

- `family/tokens.css` – custom properties palette and per‑site accent variables (aubergine, antifoul, depth, reef).
- `family/base.css` – reset, type scale, and reusable component classes (`.eyebrow`, `.chain`, `.ledger`, `.demo-card`, `.bar-list`).
- `family/provenance-panel.css` – styles for the “what’s real here” honesty component.
- `family/provenance-panel.html` – HTML fragment template for that component (not incorporated into any page).
- `family/README.md` – full documentation of the design‑system rationale and intended usage.

**I have read every source file that was provided:**
- `src/index.ts`
- `wrangler.jsonc`
- `family/base.css`
- `family/provenance-panel.css`
- `family/tokens.css`
- `family/provenance-panel.html`
- `family/README.md`
- `public/index.html`

Every capability claim below is derived from those files.

---

## 2. Problem this repository solves (current scope)

- **Static‑file serving for the `fishinglog` site** – a minimal, deploy‑ready Cloudflare Worker.
- **Design‑system integration** – the design‑system CSS is already inlined into the served page (`public/index.html`), so the page uses the shared tokens (––ink, ––shell, ––claw, etc.) and component classes (`.eyebrow`, `.chain`, `.demo-card`, etc.).  
  The page describes a real PyPI package (`fishinglog-agent`) and its capabilities; those claims are made by the page content, not by code in this repository. The repository does *not* contain the package’s source.

---

## 3. How to run and deploy

### Prerequisites

- [Node.js](https://nodejs.org/) ≥ 18
- [Wrangler](https://developers.cloudflare.com/workers/wrangler/) installed globally, or run via `npx`

### Local development

```bash
# install dependencies (none required for the current worker, but a package.json may be added later)
npm install

# start the development server on localhost:8787
npx wrangler dev
```

The Worker will serve whatever files are present in `./public`.  
Visiting `http://localhost:8787` will show `public/index.html` (or the directory listing depending on configuration).

### Deploy to Cloudflare

```bash
npx wrangler deploy
```

After deployment, the Worker is live at the subdomain defined by the `name` field in `wrangler.jsonc` (e.g. `fishinglog.<your-subdomain>.workers.dev`).

---

## 4. What this repository explicitly does **NOT** do (yet)

| Capability | Marker | Reason |
|------------|--------|--------|
| Use the provenance‑panel component in the served page | ⚠️ Conditional | The CSS and HTML for the component exist in `family/`, but `public/index.html` does **not** include any `<aside class="provenance-status">` or `.provenance` elements. The component is not wired into any served page. |
| Automate design‑system inlining via a build script | 🔮 Aspirational | The design‑system CSS is currently in the page because it was manually pasted into the `<style>` block of `index.html`. No `build.mjs` or similar script exists. |
| API endpoints, state‑machine logic, or dynamic data | ✅ Real today (does not exist) | The Worker only forwards to static assets. There is no routing logic, no database binding, no environment variables beyond `ASSETS`. |
| Serve content for multiple family sites | 🔮 Aspirational | The `family/` directory is designed for reuse, but no other repository is set up to consume it. |
| Host the `fishinglog-agent` Python package source | ⚠️ Not applicable | The package is an external artifact published on PyPI. The landing page describes it, but its source is not part of this repository. |

---

## 5. Honesty markers used in this document

- **✅ Real today** – I traced the claim to actual source code (`src/index.ts`, `wrangler.jsonc`, `public/index.html`, the inline CSS block).
- **⚠️ Conditional** – The claim is true in principle (the files exist) but the feature does **not** work in the current deployment because something missing (a build step, live server, API key, or manual inclusion of the component).
- **🔮 Aspirational** – The claim describes a direction that is documented in `family/README.md` but is not implemented.

If you spot a claim that should be marked differently, please open an issue.

---

## 6. Where to go next

- Read `family/README.md` for the full rationale behind the design‑system skeleton and the per‑site accent pattern.
- If you want to add the provenance‑panel to this page, insert the HTML from `family/provenance-panel.html` into the `<body>` of `public/index.html` and ensure the CSS classes are already covered by the existing inline style block.
- To automate the design‑system inlining across multiple family sites, write a `build.mjs` script as described in `family/README.md` (“How a family site uses this”).
