# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page web tool that arranges uploaded card-art images onto A4 sheets sized for Yoto cards (54×85mm by default), then uses the browser's native print dialog to output a PDF or printed page.

## Architecture

The entire app lives in `index.html` — there is no build step, no framework, no dependencies. Three concerns coexist in that one file:

- **CSS layout** uses physical units (`mm`) and CSS Grid so that the on-screen preview matches the printed output 1:1. The `.page` element is a fixed `210mm × 297mm` (A4) grid; card dimensions are driven by the CSS custom properties `--card-w`, `--card-h`, `--gap`, which JS rewrites on each render.
- **`@media print` + `@page { size: A4; margin: 0 }`** strips the controls UI and removes browser-injected page margins so cards land at the user-specified position. Changing page margins must be done via the `--page-margin` variable AND the `@page` rule together, otherwise screen and print diverge.
- **JS render loop** (`render()` in `index.html`) recomputes columns/rows from the current input dimensions (`calcPerPage()`), slices the image list per page, and rebuilds the DOM. Images are held as object URLs (`URL.createObjectURL`) and explicitly revoked before reassignment to avoid leaks. Files are sorted with `localeCompare(..., { numeric: true })` so `card2.png` comes before `card10.png`.

The fixed page margin inside `calcPerPage()` is hardcoded to `10` (mm) and must stay in sync with `--page-margin` in CSS.

## Deployment

Pushes to `main` trigger `.github/workflows/static.yml`, which uploads the entire repo root as a GitHub Pages artifact and deploys it. There is no build, lint, or test step — the served file is `index.html` as-is. To preview locally, open `index.html` directly in a browser or serve the directory with any static server (e.g. `python3 -m http.server`).
