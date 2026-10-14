# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page React 18 + TypeScript "Treasure Hunt" game built with Vite (exported from Figma Make). Three chests are shown; one randomly holds treasure (+$100), the others hold skeletons (-$50). The game ends when the treasure is found or all chests are opened.

This repo is also a Claude Code tutorial project — `README.md` lists the intended exercises (adding sound effects, win/tie/loss result, key-cursor hover, SQLite sign-up/sign-in with guest mode, Vercel / GitHub Pages deploy commands under `.claude/commands/`).

## Commands

```bash
npm install
npm run dev     # Vite dev server on http://localhost:3000 (auto-opens browser)
npm run build   # Production build to ./build (not ./dist)
```

There is no test runner, linter, or `tsconfig.json` configured. TypeScript is only transpiled by SWC (`@vitejs/plugin-react-swc`) — type errors will not fail the build.

## Architecture

- **All game logic and UI live in `src/App.tsx`.** State is `boxes: Box[]` (`id`, `isOpen`, `hasTreasure`), `score`, and `gameEnded`. `initializeGame()` picks the treasure index and runs on mount and on "Play Again". `openBox()` updates score and checks the end condition inside the `setBoxes` updater.
- Animations use `motion/react` (Framer Motion): chests flip with `rotateY` on open, and the result panel fades in.
- Assets are imported as ES modules: chest images in `src/assets/` (`treasure_closed.png`, `treasure_opened.png`, `treasure_opened_skeleton.png`, `key.png`), sounds in `src/audios/` (`chest_open.mp3`, `chest_open_with_evil_laugh.mp3`). When a closed chest is clicked, `openBox()` plays `chest_open.mp3` for treasure and `chest_open_with_evil_laugh.mp3` for a skeleton.
- `src/components/ui/` is a shadcn/ui component library (Radix-based, `cn()` helper in `ui/utils.ts`). Only `Button` is currently used; the rest is available but unused.
- `vite.config.ts` aliases `@` → `./src` and maps version-suffixed imports (e.g. `'sonner@2.0.3'`) to plain package names — a Figma Make artifact. Keep these aliases if shadcn components import versioned specifiers.

## Styling caveat (important)

Tailwind is **not** installed as a build dependency. `src/index.css` is a **precompiled, static Tailwind v4 output** imported by `src/main.tsx`, and it only contains utility classes that were in use when it was generated (e.g. `bg-amber-200` and `bg-amber-600` exist, but other `bg-amber-*` shades do not). Adding a new Tailwind class in JSX will silently have no effect unless it already exists in `index.css`. Before using a new utility, grep `src/index.css` for it; otherwise use inline `style` or add plain CSS rules. `src/styles/globals.css` (theme tokens) is not imported anywhere.

## Notes

- `src/guidelines/Guidelines.md` is an empty Figma Make template (no active rules).
- Third-party asset attributions are tracked in `Attributions.md`; add an entry when introducing new external sounds/images.
