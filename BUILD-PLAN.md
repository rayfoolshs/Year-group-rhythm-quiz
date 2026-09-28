# Build Plan — Year-group Rhythm Quiz

**Project:** Year-group Rhythm Quiz ("Complete the Bar")
**Curriculum:** Western Australian Curriculum — The Arts | Music, Scope and sequence, Pre-primary–Year 10 (Addendum: elements of music — **Rhythm**)
**Version:** 1.0 · **Date:** 28 September 2026 · **Owner:** ray.foo@lakelandshs.wa.edu.au
**Repository:** https://github.com/rayfoolshs/Year-group-rhythm-quiz (private)

---

## 1. Purpose

Build a self-contained, offline, browser-based tool that helps students learn the **rhythm** content of the WA Music curriculum by completing bars. It provides both **practice** (instant feedback) and a **10-question test per year group** with results exported to CSV for the teacher.

## 2. Goals and objectives

- Present only the rhythms each year group is required to learn (no content from later years).
- Give students a clear, motivating way to practise reading and completing rhythm bars.
- Give teachers a fast, low-friction assessment tool (student name, score, question-by-question review, CSV class list).
- Run anywhere with no install, no server, and no internet.

## 3. Success criteria

| # | Criterion |
| --- | --- |
| 1 | All 11 year groups (Pre-primary–Year 10) selectable, each using only its own rhythm content |
| 2 | Bar-completion questions across 2/4, 3/4, 4/4, 5/4, 6/8, 7/8, 9/8, 12/8 |
| 3 | Notation is musically correct, including **metric beaming** (beams never break the pulse) |
| 4 | Each year has a fixed 10-question test with a student-name field and a review sheet |
| 5 | CSV export with `Year, Name, Date, Score, Q1–Q10, Wrong`, answer key on top, student rows below, appending into one class list |
| 6 | Runs from a single HTML file by double-click, offline, with no console errors |

## 4. Scope

**In scope**
- Practice mode with hints and feedback.
- Test mode (10 fixed questions per year), results review, CSV export, class list.
- High-quality vector notation (VexFlow) with correct beaming.
- Package delivery (folder), single-file build, and zip.

**Out of scope (v1)**
- Audio playback / aural training.
- Teacher accounts, cloud storage, multi-device sync.
- LMS integration (Google Classroom, SEQTA, Canvas).
- Marking of written/invented notation.

## 5. Audience and stakeholders

- **Students** (Pre-primary–Year 10) — practice and sit tests.
- **Music teachers** — run tests, export and review results.
- **School IT / data privacy** — review data handling before classroom deployment.

## 6. Curriculum mapping (source of truth)

Content is taken from the Addendum table "Examples of the elements of music … Rhythm".

| Year | Rhythm content used |
| --- | --- |
| Pre-primary | Steady beat, sound and silence, long and short |
| Year 1 | Quavers, beamed quavers, groups of 2/3/4 beats |
| Year 2 | 2/4, minim, crotchet, quaver, quaver rest |
| Year 3 | 4/4, four beamed semiquavers, minim rest, crotchet rest |
| Year 4 | 3/4, dotted minim, semibreve |
| Year 5 | Semibreve rest, anacrusis, dotted quaver + semiquaver, quaver + two semiquavers |
| Year 6 | Consolidation of 2/4, 3/4, 4/4 |
| Year 7 | 2/4, 3/4, 4/4, groups of beats, anacrusis, dotted crotchet + quaver |
| Year 8 | Compound 6/8, ties, dotted crotchet, dotted minim |
| Year 9 | Simple-time groupings, 7/8, quaver triplet, swung rhythms |
| Year 10 | Compound 12/8 and 9/8, simple 5/4, syncopation, tied/irregular rhythms |

## 7. Functional requirements

**FR-1 Practice mode** — year tabs; random question; option selection; immediate correct/incorrect feedback with explanation; hint; score/streak; teacher answer-key log; keyboard shortcuts.

**FR-2 Question generation** — build bars from a rhythm pattern library scoped by year; one blank; four options with unique durations; exactly one correct; supports "complete part of a bar" and "fill the whole bar" types.

**FR-3 Notation** — vector SVG; time signatures; dashed blank box; correct stems, flags, beams, ties, tuplets, rests; **beams grouped by the metric pulse** (simple = crotchet, compound = dotted crotchet, 7/8 = 2+2+3).

**FR-4 Test mode** — fixed 10 questions per year (deterministic), student-name input, progress bar, no marking until the end, results review table `Q | Question | Your answer | Correct answer | ✓/✗`.

**FR-5 Results and export** — store results in-browser; export CSV with a Year column, answer-key row on top, student rows below, ✗ marks and a Wrong list; single-year and all-years exports; class-list panel; clear results.

**FR-6 Delivery** — `index.html` + `lib/vexflow.min.js` + docs; a generated single-file `rhythm-quiz-standalone.html`; a clean `rhythm-quiz-package.zip`.

## 8. Non-functional requirements

- **Offline / zero-install:** works over `file://`; no build step; no network.
- **Performance:** instant question rendering; no perceptible lag on classroom laptops.
- **Accessibility:** keyboard operable, sufficient colour contrast, responsive to one column on small screens, print-friendly.
- **Portability:** modern evergreen browsers (Chrome, Edge, Safari, Firefox).
- **Maintainability:** plain HTML/CSS/JS; no framework; clearly sectioned code; documented rhythm library.
- **Privacy:** no transmission; results held locally by default (see §10 risks).

## 9. Technical architecture

```
rhythm-quiz-package/
├── index.html          UI + CSS + app logic; loads the notation engine
├── lib/vexflow.min.js  VexFlow 3.0.9 (MIT) — vector notation
├── README.md
└── LICENSES.md
```

**Modules inside `index.html`**
1. **SVG fallback renderer** — used only if VexFlow fails to load.
2. **VexFlow layer** — note construction, rests, dots, ties, tuplets.
3. **Metric beaming engine** — `beatGrid()`, `metricBeams()`; confines every beam to one beat.
4. **Pattern library** — named rhythm cells with a duration and a `since` (year introduced).
5. **Year configuration** — time signatures + rhythm vocabulary per year.
6. **Generator** — bar partitions, blank selection, distractor selection with unique durations.
7. **Practice UI** — tabs, question card, feedback, score, answer key.
8. **Test engine** — deterministic 10-question paper, flow, review table.
9. **Results/CSV** — `localStorage` records, CSV builder/escaping, download, class list.

**Data model (abridged)**
- `PATTERNS`: `{ id, atoms, since, note }`
- Question: `{ yearIdx, sig, cells[], target, options[], total, prompt, explain }`
- Record: `{ name, yearIdx, yearLabel, date, score, total, answers[], correct[], wrong[] }`

## 10. Data, privacy and compliance

- The app stores student **name, date, score and answers** in the browser's `localStorage` and exports them as CSV. Nothing is transmitted.
- Risks on shared devices: the class list and any CSV are readable by the next user.
- **Mitigations to schedule (see Phase 9):** optional anonymous mode, session-only storage, PIN on the teacher panel, auto-clear, privacy notice.
- Note: the app is **not** SOC 2 certified (SOC 2 is an organisational attestation, not a property of a file). Apply school privacy policy (e.g. Australian Privacy Principles / WA DoE policy) before storing student names.

## 11. Phases and milestones

Status key: ✅ done · 🟡 in progress · ⬜ planned

| Phase | Deliverable | Key tasks | Est. | Status |
| --- | --- | --- | --- | --- |
| 0 | Content extraction | Parse the PDF; render pages to read image-based time signatures; build the year→rhythm table | 0.5 d | ✅ |
| 1 | Notation prototype | VexFlow integration; notes, rests, beams, ties, tuplets; blank box overlay | 1 d | ✅ |
| 2 | Practice mode | Pattern library, year scoping, generator, tabs, feedback, hints, score | 1.5 d | ✅ |
| 3 | Test mode | Fixed 10-question paper, name entry, progress, review table | 1 d | ✅ |
| 4 | CSV & class list | Records storage, CSV builder, answer-key row, single/all-year export, class panel | 1 d | ✅ |
| 5 | Correctness pass | Year 4 scope fix (dotted crotchet moved to Y7); whole-bar questions for focus rhythms; **metric beaming** | 1 d | ✅ |
| 6 | Packaging & docs | Package folder, standalone build, zip, README, licences, build prompt | 0.5 d | ✅ |
| 7 | QA & accessibility | Validation loop, cross-browser checks, keyboard, responsive, print | 0.5 d | ⬜ |
| 8 | Release & backup | Git init, GitHub remote, push; version tag | 0.25 d | ✅ |
| 9 | Hardening / v1.1 | Anonymous mode, session-only option, PIN, auto-clear, privacy notice | 1 d | ⬜ |
| 10 | Enhancements / v2 | Audio playback, teacher dashboard, LMS export | TBD | ⬜ |

**Total to a classroom-ready v1: ≈ 8–9 person-days** (Phases 0–8), of which the build itself is complete.

## 12. Test plan

- **Content validation (automated):** for hundreds of generated questions per year, assert (a) bar totals equal the time signature, (b) at least one given cell, (c) exactly one option matches the target, (d) four distinct option durations.
- **Notation checks:** beaming per metre (4/4 four quavers → 2+2; 6/8 six quavers → 3+3; 7/8 → 2+2+3); rests break beams; triplets beamed as a group; blank box aligns to the beat.
- **Test engine:** every year returns 10 questions with a stable answer key; CSV header/rows correct; results append.
- **Browser matrix:** Chrome/Edge/Safari/Firefox; 1366×768 laptop and tablet; `file://` launch.
- **Accessibility:** tab/Enter navigation, visible focus, contrast, print to PDF.

## 13. Risks and mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Incorrect beaming (beams crossing the pulse) | Teaches wrong notation | Metric beaming engine + visual checks (fixed in v1) |
| Out-of-scope rhythms in a year | Too hard / off-curriculum | `since` tags per pattern; verified against the PDF (Y4 fixed) |
| Student privacy on shared devices | Data exposure | Anonymous/session-only options, PIN panel, auto-clear (Phase 9) |
| VexFlow unavailable/offline | No notation | Library bundled (standalone) or vendored locally (package) |
| Test paper changes after printing | Mismatch with marked papers | Fixed per-year seed; version-stamp the paper in a future release |
| Question ambiguity | Two correct answers | Unique option durations enforced by validation |

## 14. Acceptance criteria (definition of done for v1)

- [ ] All success criteria in §3 met.
- [ ] Automated content validation passes with zero failures.
- [ ] No console errors on load in the supported browsers.
- [ ] Package, standalone file and zip all open and work offline.
- [ ] README and licences present; repo pushed to GitHub.

## 15. Future roadmap (v2 ideas)

- **Aural mode:** play the bar (Web Audio) so students identify rhythms by ear.
- **Teacher dashboard:** import CSV, class analytics, per-question difficulty.
- **LMS export:** Google Classroom / Canvas / SEQTA-friendly formats.
- **Custom papers:** teacher selects question types and difficulty per year.
- **Versioned papers:** stamp each generated paper so printed tests map to a CSV revision.
- **Anonymous-first mode:** student IDs instead of names, with a privacy notice.

## 16. Build and release commands

```bash
# regenerate the single-file build from the package
python3 - <<'PY'
idx=open("rhythm-quiz-package/index.html").read()
vf=open("rhythm-quiz-package/lib/vexflow.min.js").read()
tag='<script src="lib/vexflow.min.js"></script>'
inline='<script>\n/* VexFlow 3.0.9 - MIT - inlined */\n'+vf+'\n</script>'
open("rhythm-quiz-standalone.html","w").write(idx.replace(tag,inline,1))
PY

# rebuild the distributable zip (no macOS junk)
zip -rX rhythm-quiz-package.zip rhythm-quiz-package

# commit and back up
git add -A && git commit -m "..." && git push
```