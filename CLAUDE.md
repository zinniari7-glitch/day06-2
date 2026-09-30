# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

English vocabulary web app (영어 단어장), used on a phone. The whole app is a single `index.html` (inline CSS + JS, no build step, no package manager, no tests, no linter). Docs are written in Korean.

- `PRD.md` — requirements. `STEPS.md` — build order (1 add/check/delete → 2 tabs/count → 3 design). Steps 1–3 are all implemented.
- `DESIGN-mastercard.md` — the design system the UI follows (cream `#F3F0EE` canvas, ink `#141413`, pill/20/24/40px radii, Sofia Sans, weight 450 body / 500 headings, -2% heading tracking, no in-between radii, only soft large shadows). Check it before changing styles.

## Running

Serve over localhost (do not open via `file://`):

```bash
python -m http.server 8765
```

Port 8000 is used by another project on this machine (a guestbook app), so use a different port. Verify changes in the browser; there is no automated test suite.

## Architecture

- **Data**: Supabase project `day06` (ref `tiscdhmqhigzgjytjyno`, the only project on the account; the PRD calls it "Practice"), table `public.words` (`id`, `word`, `meaning`, `memorized`, `created_at`). The same project also holds an unrelated `guestbook` table — leave it alone.
- **Access**: the browser calls Supabase directly via the CDN `supabase-js` client using the publishable key embedded in `index.html`. There is no login; RLS is on with anon select/insert/update/delete policies, and the `anon` role was explicitly granted privileges on the table (RLS alone gave "permission denied"). Schema changes go through Supabase migrations, not the HTML.
- **Client state**: a module-level `words` array (source of truth after initial `load()`) and `tab` (`'all'` | `'todo'`). All UI comes from one `render()` that rebuilds the list and updates the "N개 중 M개 외움" count (always over all words, regardless of tab). Add/toggle/delete write to Supabase first and only update `words` + `render()` on success.

## Invariants

- User text is rendered only with `textContent` / DOM creation — never `innerHTML` — so input like `<b>x</b>` shows literally.
- Word and meaning are trimmed; if either is empty nothing is saved.
