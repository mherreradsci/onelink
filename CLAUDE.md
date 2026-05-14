# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
yarn install   # install dependencies
yarn dev       # dev server at http://localhost:3000
yarn build     # production build
yarn generate  # static site generation
yarn preview   # preview production build
```

No test suite or linter is configured.

## Architecture

Onelink is a **serverless link-in-bio tool** built with Nuxt 3. All user data lives entirely in the URL as a base64-encoded JSON query parameter (`?data=...`). There is no backend, database, or authentication.

### Data flow

1. User fills out the editor at `/` (`pages/index.vue`)
2. On "Publish", `encodeData()` (`utils/transformer.js`) serializes the reactive `data` object → `JSON.stringify` → base64
3. The resulting URL is copied to clipboard: `<origin>/1?data=<base64>`
4. `/1` (`pages/1.vue`) reads `route.query.data`, calls `decodeData()`, and passes the result as `:acc` to `<templates-simple>`

### Data schema (short keys are intentional — minimizes URL length)

| Key | Field |
|-----|-------|
| `n` | Name |
| `d` | Description/bio |
| `i` | Avatar image URL |
| `f`, `t`, `ig`, `gh`, `tg`, `l`, `e`, `w`, `y` | Social links (Facebook, Twitter, Instagram, GitHub, Telegram, LinkedIn, Email, WhatsApp, YouTube) |
| `ls` | Custom links array: `[{ l: label, i: icon, u: url }]` |

### Key files

- **`utils/transformer.js`** — `encodeData` / `decodeData` via `js-base64`
- **`components/AppForm/`** — editor UI split into Profile, SocialLinks, Links, Hr, Preview sections
- **`components/Templates/Simple.vue`** — the rendered public profile; receives the full decoded object as `acc` prop
- **`components/ExternalLink.vue`** — renders each item in the custom `ls` links array

### Adding a new template

The `/1` route maps to `Templates/Simple.vue`. New templates follow the pattern: add `/2` page → new `Templates/<Name>.vue` component receiving `acc` as a prop.

### Nuxt 3 conventions used

- Components are auto-imported; file path determines component name (`AppForm/Profile.vue` → `<app-form-profile>`)
- `<script setup>` is used throughout — no explicit `defineComponent` or `setup()` 
- Icons use `nuxt-icon` with Iconify names (e.g. `ph:github-logo-duotone`)
