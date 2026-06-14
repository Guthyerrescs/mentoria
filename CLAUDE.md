# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ Critical: this repo contains a compiled build, not source

This repository holds **only the production build output of a Create React App (CRA)** —
the kind of directory CRA emits into `build/`. There is **no application source tree**
committed here: there is no `package.json`, no `src/`, no `tailwind.config.js`, and no
raw `.css`/`.jsx` files. Everything tracked in git is a deployed static artifact:

- `index.html` — single HTML entry, loads the hashed JS/CSS bundles
- `static/js/main.<hash>.js` (+ `453.<hash>.chunk.js`) — minified, bundled React app
- `static/css/main.<hash>.css` — compiled, minified Tailwind output
- `static/media/*` — fingerprinted images, video, and `@fontsource` font files
- `asset-manifest.json`, `manifest.json`, `favicon*`, `robots.txt` — CRA/PWA scaffolding

**Do not hand-edit the minified bundles in `static/`** to make feature changes. They are
generated, fingerprinted (content-hash filenames), and not meant to be edited by hand.
A real change requires the original source, which is **not in this repo**.

### Recovering the original source

The original, unminified source is embedded in the sourcemaps and can be extracted:

```bash
# List the original source files that produced the bundle
python3 -c "import json;m=json.load(open('static/js/main.d9ee19cd.js.map'));\
print('\n'.join(s for s in m['sources'] if 'node_modules' not in s))"

# Dump the original content of a specific source file (e.g. App.js)
python3 -c "import json;m=json.load(open('static/js/main.d9ee19cd.js.map'));\
d=dict(zip(m['sources'],m['sourcesContent']));print(d['App.js'])"
```

(Adjust the hashed `.map` filename if the bundle has been rebuilt.) If the task is to
modify the app, the right move is usually to **reconstruct the CRA source project**
(`src/`, `package.json`, `tailwind.config.js`) from these sourcemaps, make changes there,
run `npm run build`, and replace this directory with the fresh build output — rather than
patching compiled assets.

## What the app is

A single-page, Portuguese-language **marketing/landing site for "Mentoria Médica"** — a
medical mentorship program by Dr. Guthyerres Carvalho (page title in `index.html`:
"Dr. Guthyerres Carvalho - Mentoria Médica"). It is a static, mostly-presentational site
with one interactive flow: an application/enrollment form modal.

## Application architecture (from source)

- **Stack:** React 19 + Create React App (Webpack) + Tailwind CSS, with
  `@fontsource/montserrat` and `@fontsource/poppins` for self-hosted fonts.
- **Entry:** `index.js` → `<App />` inside `<React.StrictMode>`, mounted on `#root`.
- **`App.js` is the single orchestrator.** It renders the page sections in order and owns
  the only piece of cross-cutting state: `showApplicationForm` (boolean). It passes an
  `onOpenForm` callback down to the sections that have a call-to-action button, and
  conditionally renders `<ApplicationForm onClose={...} />` as a modal overlay.
- **Section components** (each a self-contained presentational block, in `src/components/`):
  `Header`, `Hero`, `About`, `Mentorship`, `Topics`, `ImplantCourse`, `Locations`,
  `Contact`, `Footer`. Most are static content; CTA-bearing ones receive `onOpenForm`.
- **`ApplicationForm.jsx`** is the only stateful/networked component. It is a **two-step
  wizard**:
  1. Step 1 (`contactData`: nome, email, telefone, especialidade) — submits as
     `type: 'contact_data'`.
  2. Step 2 (`questionsData`: pergunta1–4 qualification questions) — submits as
     `type: 'questions_data'` with the contact data nested.

  Both steps `POST` JSON to the external backend
  **`https://ogh5izcvmqvw.manus.space/send-email`** (a Manus.space–hosted email endpoint).
  This is the app's only external dependency/integration. Success/error is surfaced via
  `alert()`; there is no local backend in this repo.

## Styling conventions

Tailwind utility classes with a **custom theme** (defined in the missing `tailwind.config.js`,
visible in the compiled CSS and class usage). When reconstructing source, preserve these
semantic tokens rather than hardcoding hex values:

- Color tokens: `primary` / `primary-medium` / `primary-dark`, `secondary`,
  `neutral-dark` / `neutral-medium` / `neutral-lightest`. Palette in the compiled CSS
  centers on deep navy (`#1a2347`, `#2a2a6d`) with a gold/tan accent (`#c9a77c`).
- Font families: `font-sans` (Montserrat, headings/nav), `font-body` (Poppins, body).
- Custom shadows: `shadow-soft`, `shadow-medium`.
- Layout is responsive with Tailwind's `md:` breakpoints; `Header` has separate
  desktop nav and a mobile hamburger menu toggled by local `isMenuOpen` state.

In-page navigation uses anchor links to section ids: `#sobre`, `#mentoria`, `#temas`,
`#curso`, `#locais`, `#contato`.

## Working with this repo

- This is a **static site** — to preview it, serve the directory over HTTP, e.g.
  `python3 -m http.server 8000` (note `index.html` references absolute `/static/...`
  paths, so serve from the repo root).
- There are **no build, lint, or test commands available in this repo** because the
  tooling and `package.json` are not committed. Do not invent npm scripts; if a task
  needs them, first reconstruct the CRA source project as described above.
- Content is in Brazilian Portuguese — keep new copy in Portuguese to match.
- Asset and bundle filenames are content-hashed; if you regenerate the build, update
  `index.html` and `asset-manifest.json` references accordingly (a fresh `npm run build`
  does this automatically).
