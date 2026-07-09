# Architecture decisions: build‑time inlining of the shared design system

This document explains a non‑obvious design choice that is documented in `family/README.md` but may not be immediately clear to a newcomer.

---

## The problem

The PurplePincher family of sites (e.g., purplepincher.org, deckboss.net, cocapn.com, fishinglog.ai) share a common visual identity – a palette of CSS custom properties and a set of reusable component classes (defined in `family/tokens.css` and `family/base.css`).  
These sites are each deployed as a separate Cloudflare Worker, and each Worker serves static HTML (and WASM) without any runtime fetch to a shared CSS host.

How should the common CSS be made available to each site’s HTML?

### Option A: `<link>` to a shared stylesheet host

- Requires adding a new origin to the Content‑Security‑Policy (`style-src`).
- The CSP is intentionally strict as part of the honesty architecture (it prevents the page from loading unexpected resources). Adding a shared host weakens that guarantee.
- The original purplepincher landing‑page reference (`purplepincher-landing-reference`) already has a strict CSP with `style-src 'self' 'unsafe-inline' …` and no provision for an external stylesheet host.

### Option B: Inline at build time – **chosen**

- Each site’s build step (hypothetically a `build.mjs` script) reads the shared CSS files (tokens.css, base.css, optionally provenance-panel.css) and concatenates them into a `<style>` block in the site’s own HTML.
- The deployed Worker is a **hermetic** artifact – it contains all the CSS it needs, referenced by `'unsafe-inline'`. No new origin is added to the CSP.
- The site remains *checkable*: the HTML served is the complete, authoritative source. There is no runtime dependency on a second server whose content could drift.

---

## What this repository actually does today

The `public/index.html` file in this repository already contains an inline `<style>` block that includes the full contents of both `family/tokens.css` and `family/base.css` (plus site‑specific styles).  
This was done **manually** – there is no automated build script that concatenates the files. The decision to inline matches Option B, but the process is not yet automated.

The implications:

- The design‑system CSS is applied to the served page and is verifiable by examining the single HTML file. ✓ **Real today**
- Adding the provenance‑panel component would require manually inserting both the CSS (already present in `family/provenance-panel.css`) and the HTML fragment (`family/provenance-panel.html`) into the page. No automation. ◐ Conditional
- When multiple family sites need to share the design system, a build script that performs this inlining automatically will become necessary. → Aspirational

---

## Why this matters for honesty

The “honesty perimeter” mentioned in `family/README.md` and in the broader PurplePincher documentation is a core principle: every site should be auditable at the HTTP‑response level.  
If a site declares “this page is the real HTML, nothing fetched,” then any CSS fetched from a separate host would break that claim – the page would be incomplete without that second host.

Inlining eliminates the second host, making each Worker truly self‑contained. The current manual approach preserves that property, but it is not repeatable without a build step.

---

## Honesty marker for this document

- **✓ Real today** – the rationale matches the contents of `family/README.md` and the actual implementation in `public/index.html`.
- **◐ Conditional** – adding the provenance‑panel is possible but not done; the CSS for it exists but is not deployed.
- **→ Aspirational** – automating the inlining with a build script is documented but not implemented.

---

## Further reading

- `family/README.md` – describes the design‑system files, the per‑site accent table, and how a minimal build step would work.
- `ROUND_1_glm.md` (not in this repo) – the original plan that justifies the architecture.
