# Speech Technology: Notes & Past Papers

Notes and past-year questions for **BSEE4001 Speech Technology** (IIT Madras BS Degree), published as a searchable website with an **Ask AI** button on every page.

- **[Open the website](https://manyakaistha.github.io/SpeechTechnology-Notes/)**
- [Past-paper topics](https://manyakaistha.github.io/SpeechTechnology-Notes/past-paper-topics.html): pick a topic (for example *sampling, aliasing and sample counts*) and jump to every question that tested it.
- [ELI12 notes](https://manyakaistha.github.io/SpeechTechnology-Notes/eli12/eli12-start.html): all five weeks in plain language.
- [Exam study route](https://manyakaistha.github.io/SpeechTechnology-Notes/exam-study-guide.html): what to learn first, ranked P1 to P3.

## What is inside

| Part | Where | Contents |
| :-- | :-- | :-- |
| ELI12 notes | `content/eli12/week-N-eli12.md` | Plain-language "explain like I'm 12" versions of all five weeks, with tiny hand-checkable examples and links to the matching past-paper questions |
| Weekly notes | `content/week-N/week-N-notes.md` | Weeks 1 to 5: theory, derivations, figures, runnable code with real outputs |
| Exam-focused guides | `content/week-N/week-N-exam-focused-notes.md` | Beginner-friendly route through each week, weighted by what the papers test (Weeks 1 to 4) |
| Past papers | `content/week-N/week-N-exam-questions.md` | 98 question occurrences from 18 papers, each with the scanned stem and a worked answer (Weeks 1 to 4) |
| Concept notes | `content/notes/concepts/` | 28 short atomic definitions that link the weeks together |
| Paper archive | `content/sources/exam-papers/` | Original PDFs, text extractions, the routing audit |
| Lab code | `labs/` | `weekN_lab.py` and `generate_weekN_plots.py`, run with `uv` |

Worked answers are **independently derived study aids, not official answer keys**. Where a source was ambiguous or its OCR looked wrong, the page says so next to the question.

## Ask AI

Every page has an **Ask AI** stamp (bottom right) and a small waveform button beside each heading. Select any text first to ask about just that passage. The helper only builds a study prompt in your browser and opens ChatGPT, Claude or Grok with it prefilled, or copies it. No API calls are made and nothing is sent anywhere until you choose a provider.

## Build the website locally

```bash
uv run scripts/build_site.py --output _site     # builds, then fails on any broken link or anchor
uv run scripts/check_render.py                  # flags leftover Markdown or TeX syntax
uv run scripts/check_links.py _site             # independent crawl of the built HTML
uv run --no-project python -m http.server 8765 --directory _site
```

GitHub Actions (`.github/workflows/pages.yml`) runs the same three checks and publishes to GitHub Pages whenever `main` changes. Math is rendered with MathJax from jsDelivr.

## Re-run the labs

```bash
cd labs
uv run python week1_lab.py
uv run python generate_week1_plots.py
```

## Layout

```text
content/            Markdown sources, figures, paper scans and PDFs (the site mirrors this tree)
labs/               Runnable code behind the notes
scripts/            Site builder and link/render checkers
site-assets/        CSS and JavaScript (themes, hero animation, search, Ask AI)
.github/workflows/  GitHub Pages deployment
```

Course lecture slide decks are cited by slide number in the notes but are not republished here.
