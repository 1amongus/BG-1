# Bhagavad Gita Learning Path (BG-1)

A single-file, phone-first guided reader for the Bhagavad Gita. It renders the
Samsaadhanii e-reader's word-by-word grammatical analysis — pada tables,
anvaya (prose order), and karaka graphs — as a Duolingo-style learning path
with one lesson per verse.

- **Live site:** https://1amongus.github.io/BG-1/
- **Repository:** https://github.com/1amongus/BG-1
- **Coverage:** all 18 chapters, 700 verses, verified live.

## Principles

These rules govern everything in this project. Please keep them if you build
on it.

1. **Stick to the source.** The pada tables, anvaya, and karaka relations
   reproduce the Samsaadhanii e-reader's analysis verbatim. The app's
   contribution is interface and presentation only: no builder-composed
   roles, no silent "corrections" of the source's tagging, no recomposed
   anvaya. Where the source has no analysis, the app shows verse text only
   and never invents analysis to fill the gap.
2. **Attribute.** Every screen credits the Samsaadhanii e-reader, and the
   home screen carries a thank-you tile linking to
   https://sanskrit.uohyd.ac.in/scl/ with the sentence: "With gratitude, all
   verse analysis here is from the Samsaadhanii e-reader (Computational
   Linguistics Group, University of Hyderabad)."
3. **Permission.** Prof. Amba Kulkarni granted full permission to render
   these books (14 November 2026), including rendering other Samsaadhanii
   books as well. At her suggestion, relations containing द्योतक will be
   grayed in the graphs (presentation only, source data unchanged) — counted
   (443 displayed relations in the Gita data) but not yet applied.
4. **Non-commercial, open source.** The author intends this project to stay
   non-commercial and open source. No license file has been added yet — ask
   the author before redistributing.

## What's in this repo

| File | What it is |
|---|---|
| `index.html` | The app. One self-contained file (~5.3 MB). |
| `kriyapada-quiz.html` | Sibling page: an all-Sanskrit quiz — a sloka is shown, you identify the kriyapadam. Covers Chapter 1, verses 1–10; finite verbs only. Linked from the home screen. |
| `README.md` | This file. |
| `samsaadhanii-source-data.md` | The complete retrieved dataset: all 11,719 analysis records with all 25 fields, grouped by sloka entry, plus the field glossary and provenance notes. |

## Architecture

### One file, no build, no server

`index.html` is a single self-contained HTML document: one `<style>` block,
one `<script>` block, no framework, no bundler, no build step, no backend,
no database. It runs anywhere a static file runs — open it directly in a
browser, serve the folder over HTTP, or publish it with GitHub Pages (which
is how the live site is served).

- **Zero runtime network calls.** The page makes no `fetch`/`XHR` requests.
  Its only external dependency is Google Fonts (Spectral, Tiro Devanagari
  Sanskrit, Noto Serif Devanagari), with serif/system fallbacks, so the app
  works offline except for font loading.
- **No images.** There are no `<img>` tags and no image assets; the only
  inline SVG is a decorative motif. All artwork is CSS/SVG woven into
  backgrounds.
- **No persistence.** Lesson progress and XP live in memory only — the page
  uses no `localStorage` or `sessionStorage`, so reloading resets the
  session. (Adding persistence is a natural first contribution; see below.)
- **Responsive, phone-first.** 15 media queries; the drill-down navigation
  (home → section → chapter → lesson) is designed so a whole chapter's verse
  buttons fit one phone screen with no page scroll. The page declares
  `color-scheme: light dark` support.
- **Accessibility basics:** semantic buttons, `aria-pressed`/`aria-current`
  states, and keyboard-focusable lesson controls.

### Data embedded in the page

All verse data is embedded as JavaScript literals in the `<script>` block.
There are four data arrays per chapter, plus one registry:

- `VERSES`, `VERSES2` … `VERSES18` — verse records: number `n`, `speaker`,
  verse `text`, an app-side English `translation` (see below), and `words`
  triples for the Study Reader. 700 translations, one per verse.
- `ANVAYA`, `ANVAYA2` … `ANVAYA18` — anvaya (prose-order) strings per verse.
- `KARAKA`, `KARAKA2` … `KARAKA18` — karaka graph data per verse: hubs of
  the form `{ kriya, sub, nodes: [[relation, word, sub, class, depth], …] }`.
  The `depth` value drives the vertical-flow tree layout.
- `PADA`, `PADA2` … `PADA18` — pada tables per verse: rows of
  `[word, form, morph]` triples.
- `CHAPTERS` — registry with 18 keys. Each entry holds `{ name, verses,
  anvaya, karaka, pada, chipCh, chipRange, pathSub, … }`, where `chipCh` /
  `chipRange` / `pathSub` are the app-side chapter titles and introductions
  shown on tiles and the learning path.

Merged multi-verse entries in the source (e.g. 1.4–1.6) share one analysis
block; the app expands them so each display verse carries the shared anvaya
and graph.

### Rendering engine

All rendering is plain functions in the one `<script>` block. The main ones:

| Area | Functions | What they do |
|---|---|---|
| Navigation | `showPage`, `showScreen`, `mkScreen`, `openSection`, `setChapter`, `openLesson`, `closeLesson` | Switch between the guided path and the Study Reader; drill down home → section (Karma 1–6, Bhakti 7–12, Jñāna 13–18) → chapter → verse lesson overlay. Screens are built dynamically with `mkScreen`. |
| Lesson flow | `renderRead`, `renderPadaStep`, `renderAnvayaStep`, `renderQuiz`, `renderComplete`, `setAction`, `setProgress` | The four lesson steps (`read` → `pada` → `anvaya` → `quiz` → `done`): verse reading, word-form table, anvaya + karaka graph, translation quiz, completion. The overlay header shows a progress bar and XP. |
| Karaka graphs | `karakaHTML`, `kFlowTree`, `kHubBlock`, `kNodeBox` | Draw the vertical-flow tree from the relation data: the verb hub on top, each related word hanging below on a visible connector line with an arrow and the karaka label. Multi-hub verses use side-by-side columns on wide screens and stack on phones. Same source data, only the drawing changes. |
| Pada tables | `padaHTML`, `padaBadge` | Word-form tables naming each word's form (नामपदम् / क्रियापदम्). |
| Quiz | `quizOptions`, `checkAnswer` | Translation multiple-choice: the verse's translation plus two distractors drawn from other verses in the same chapter, shuffled deterministically (seeded by verse number). |
| Study Reader | `renderReader`, `renderReaderList`, `selectVerse`, `verseCardHTML` | The classic reader view (`#option-b`, currently unlinked from home): chapter list plus verse detail with a word table. |
| Helpers | `esc`, `devNum`, `dkey`, `chDone`, `nextUndone`, `refreshPath` | HTML escaping, Devanagari numerals, `chapter:verse` progress keys, chapter completion, learning-path rendering. |

### Source data vs app-side text

Keep this distinction when editing. It is the heart of the project.

- **Verbatim from Samsaadhanii:** verse text, pada words, morph analyses,
  karaka relations and head indices, anvaya order, graph structure.
- **App-side, not from the source:** the English verse translations (the
  source's `english_meaning` field is empty for every record), chapter
  introductions and section names, quiz distractors, XP/progress mechanics,
  and all UI copy.
- **The only two adjustments to source data** (both explicitly authorized,
  12 November 2026): a विधीयते row added to the merged 2.42–44 entry, and an
  implied (इति) row added to 1.20–21, marked अध्याहृतम् (supplied). Both are
  documented here so a future editor does not "fix" them back.
- **Disclosed presentation choices:** repeated च nodes in Chapter 1 graphs
  are folded for display; verses whose source record has no graph
  (`graph: "-"`) get a graph drawn from the relation data, with an on-screen
  note saying so; verses 7.30 and 13.27 have no analysis in the source and
  show text only.

## Data: how it was retrieved and generated

### Retrieval

The data was downloaded on 10 November 2026 from the public data directory
of the Samsaadhanii e-reader (a plain Apache listing — no login, no API key,
no scraping of rendered pages):

```
https://ereaders.samsaadhanii.in/books/data/books/श्रीमद्भगवद्गीता/
```

Three files were retrieved:

| File | Contents |
|---|---|
| `slokas.json` | 658 sloka entries: `chaptno`, `slokano`, and `sparts` (the verse lines; the legacy `spart1`/`spart2` fields are always empty). |
| `analysis.json` | 11,719 word records, 25 fields each (see the field glossary in `samsaadhanii-source-data.md`). |
| `chapters.json` | The chapter list `01`–`18` (plus a stray trailing `"3"` entry, preserved as retrieved). |

Filtering and quirks a reuser must know:

- **Filter by book.** Only 11,695 records belong to श्रीमद्भगवद्गीता. The
  export also carries 20 records of रघुवंश filed under key 07.006 (alongside
  the 15 Gita records for that verse) and 4 records of "1000 Sentences"
  filed under 07.302 (no matching sloka entry). Always filter
  `book = श्रीमद्भगवद्गीता`.
- **A duplicate verse.** Chapter `3` (unpadded) duplicates Gita 3.9: entry
  3.009 carries padaccheda-style text with 16 records, alongside the
  canonical 03.009 with 17 records. The app uses the padded `03` chapter.
- **Merged entries.** 13 entries cover several verses each (`slokano` like
  `004-006`, including two sub-verse splits `027_2-028` and `028_2-029`).
  They expand to the standard 700 display verses.
- **Gaps.** Entries 7.30 and 13.27 have no analysis records. 422 Gita
  records carry `graph: "-"` (no graph in the source).
- **Graph paths are names, not files.** The `graph` field names
  `assets/graphs/<chapter>_<sloka>_1.svg`, but no static graph files exist
  at any public URL: the e-reader is a Flutter web app that draws each graph
  at runtime from the relation data and exports PNG or SVG on demand (no
  JPG). This app likewise draws its graphs from the relation data.
- **`-` means absent.** The source's own marker for a missing value; it is
  preserved everywhere, never replaced with a guess. Five fields
  (`english_meaning`, `samAsa`, `prayoga`, `sarvanAma`,
  `name_classification`) are `-` for every record, and `part1`/`part2` are
  `null` throughout.

The complete dump, with every field of every record, is in
`samsaadhanii-source-data.md` in this repo.

### Generation pipeline

The embedded JS data was generated in four stages. The build scripts live
outside this repo (in the author's workspace); this section documents the
pipeline so it can be reproduced or rebuilt:

1. **Extract** — `extract_gita_tables.py` reads the JSONs and writes
   per-chapter Markdown tables (`gita-ch*.md`) plus the full corpus
   (`gita-full-corpus.md`), with match checks per verse: surface-form
   reconstruction against `slokas.json`, implied-word counts, field
   completeness, and prose-order (`poem`) running 1..N.
2. **Emit** — `gen2.py` plus per-chapter `emit_ch*.py` scripts parse the
   Markdown tables and emit the `ANVAYA` / `KARAKA` / `PADA` JS literals
   (`final_ch*.js`), validated against the already-built chapters.
3. **Splice** — `splice_ch*.py` scripts insert the emitted JS into
   `index.html` at the chapter's data block.
4. **Verify** — `verify_ch*.js` scripts check each chapter after splicing:
   record counts, merged-entry anvaya identity across display verses, and
   graph/hub structure.

**Hard-won lesson:** the intermediate Markdown tables can drop records for
merged multi-verse entries (one chapter file listed 12 of 31 records for a
merged entry). Merged entries must always be verified against
`analysis.json` directly, never only against an extraction file. Likewise,
after publishing to GitHub Pages, allow ~15 minutes for CDN cache lag before
concluding a deploy is stale — verify with a cache-busting `?v=` URL.

## How to build on top

### Run it locally

No install, no build. Either open `index.html` in a browser, or serve the
folder:

```sh
python3 -m http.server 8000
# then open http://localhost:8000/
```

### Make changes

- **Styles:** everything is in the single `<style>` block — CSS variables
  for the parchment palette and fonts at the top, component rules below.
- **UI copy:** search the `<script>` block for the Devanagari or English
  string you want to change. Lesson-step headings, button labels, and
  section names are string literals in the render functions.
- **Data fixes:** if you believe a verse's analysis is wrong, check
  `samsaadhanii-source-data.md` (or `analysis.json` at the source URL)
  first. If the app differs from the source, that is a bug — fix the app.
  If the source itself looks wrong, leave it: the project's rule is to
  reproduce the source, and any exception needs the author's explicit call
  (see the two authorized adjustments above).

### Ideas with pointers

- **Persist progress.** `done` (an in-memory object keyed by `dkey()`) and
  `xp` are the whole progress state. Saving/restoring them with
  `localStorage` in `openLesson` / `renderComplete` is a small, contained
  change.
- **Search.** Iterate `CHAPTERS` and match against verse text, pada words,
  or morph strings; link hits to `openLesson` via `setChapter` first.
- **Audio / chanting.** Add a per-verse audio URL to the verse records and
  a play control in `renderRead`. No audio exists in the source data, so
  recordings would be new, separately sourced material.
- **New quiz modes.** Follow `quizOptions` / `checkAnswer`: the lesson quiz
  is one function that returns options and one that grades them. The
  sibling `kriyapada-quiz.html` shows a second pattern (tap the kriyapada
  chips; finite verbs only, decided by lakara in the morph, never by graph
  hub).
- **Gray the द्योतक relations.** The commitment is made and the count is
  known (443 displayed relations in the Gita data, across 346 of 700
  verses). Implementation: in `kFlowTree`/`kNodeBox`, render nodes whose
  displayed relation contains द्योतक in a muted style — presentation only,
  source data unchanged. On the two dual-relation records, gray only the
  dyotaka edge.
- **Relink or remove the Study Reader.** The `#option-b` reader view is
  fully built but unlinked from home. Either add its tile back or remove
  the view and its `renderReader*` functions.
- **Dark-mode polish.** The page declares `color-scheme: light dark` but
  only partially adapts; the parchment palette needs a dark variant.

### Add another book

Prof. Kulkarni's permission covers rendering other Samsaadhanii books. The
recipe mirrors the Gita pipeline:

1. Fetch that book's `slokas.json` / `analysis.json` from the same
   `books/data/books/<Book>/` directory pattern.
2. Extract per-chapter tables with the match checks, then emit
   `VERSES` / `ANVAYA` / `KARAKA` / `PADA` blocks in the same shapes.
3. Add a `CHAPTERS` entry and a home-screen section tile.
4. Keep the attribution, the stick-to-the-source rule, and the
   verse-text-only treatment of gaps.

### Use the data elsewhere

`samsaadhanii-source-data.md` is the complete, citable dump — reuse it
directly, or re-download the JSONs from the source URL above. Respect the
`-` convention, filter by `book`, and keep the Samsaadhanii attribution
with any reuse.

### Publish

GitHub Pages serves this repo's root; `index.html` is the entry point. The
file is ~5.3 MB, so prefer `git` CLI or the web upload for updates, and
avoid editing large files in the web file editor. After a deploy, verify
live with a `?v=<commit>` URL (see the cache-lag note above).

## Current state and known limitations

- Complete and live: all 18 chapters, 700 verses, plus the kriyapada quiz
  page (Chapter 1, verses 1–10).
- The lesson overlay scrolls internally on phones for the largest karaka
  graphs and the 14 largest pada tables (27–34 rows). The rest of the app
  has zero page scroll by design.
- Progress resets on reload (no persistence yet).
- द्योतक graying is committed and counted but not yet applied.
- The Study Reader view is built but unlinked from home.

## Credits

- Verse analysis: the Samsaadhanii e-reader, Computational Linguistics
  Group, University of Hyderabad — https://sanskrit.uohyd.ac.in/scl/ —
  developed under the guidance of Prof. Amba Kulkarni. E-reader:
  https://ereaders.samsaadhanii.in/books/ereaders/
- Permission to render these books, including other Samsaadhanii books,
  granted by Prof. Kulkarni on 14 November 2026.
- Built by Sarath Oruganti. Non-commercial, open-source intent; see
  Principles above.
