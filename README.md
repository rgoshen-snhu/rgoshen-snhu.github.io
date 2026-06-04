# CS 499 Computer Science Capstone ePortfolio

The source for Rick Goshen's professional ePortfolio, completed as the final
capstone project for the Bachelor of Science in Computer Science program at
Southern New Hampshire University (SNHU). The ePortfolio showcases growth across
the program through a code review of an existing artifact, three categories of
artifact enhancement, supporting narratives, and a professional self-assessment.

## Live Site

The published ePortfolio is hosted on GitHub Pages:

**<https://rgoshen-snhu.github.io>**

> The README is for repository visitors (instructor, peers cloning the source).
> The ePortfolio itself is rendered from `index.md` and excluded files via Jekyll.

## ePortfolio Components

The CS 499 Final Project Rubric requires the deliverables below. ✅ marks
complete items; 📝 marks placeholders awaiting work.

| # | Deliverable | Status | Location |
|---|---|---|---|
| 1 | Professional Self-Assessment (presented first in ePortfolio) | 📝 TBD | `index.md` (introductory section) — Module 6 |
| 2 | Code Review Video (Milestone One) | ✅ draft | [YouTube](https://www.youtube.com/watch?v=Hk-b0DdcnT4) — pending instructor feedback revisions |
| 3 | Original Artifact — *Weight to Go* (Android, Java) | ✅ | [GitHub](https://github.com/rgoshen-snhu/WeighToGo/tree/main/android) |
| 4 | Enhancement 1 — Software Design and Engineering (Milestone Two) | ✅ | [GitHub](https://github.com/rgoshen-snhu/WeighToGo/tree/main/web) |
| 5 | Enhancement 2 — Algorithms and Data Structures (Milestone Three) | 📝 TBD | TBD — Module 4 |
| 6 | Enhancement 3 — Databases (Milestone Four) | 📝 TBD | TBD — Module 5 |
| 7 | Narrative — Enhancement 1 (Software Design and Engineering) | ✅ | `narratives/enhancement-1-software-design.md` (rendered) · `narratives/milestone_two_narrative_final.docx` (source) |
| 8 | Narrative — Enhancement 2 (Algorithms and Data Structures) | 📝 TBD | `narratives/` — Module 4 |
| 9 | Narrative — Enhancement 3 (Databases) | 📝 TBD | `narratives/` — Module 5 |

> **Note on artifact choice**: Per the rubric, enhancements may either be
> layered on the *same* artifact across all three categories (the current
> plan — extending *Weight to Go*), or applied to *different* artifacts per
> category. Rows 5 and 6 will be updated as those decisions firm up.

## Enhancement Categories

The rubric requires demonstrated skills in three key categories:

- **Software Design and Engineering** — Milestone Two (Module 3)
- **Algorithms and Data Structures** — Milestone Three (Module 4)
- **Databases** — Milestone Four (Module 5)

## Course Outcomes

The cumulative work — code review, artifacts, narratives, and self-assessment —
demonstrates the five program outcomes:

1. Employ strategies for building collaborative environments that enable
   diverse audiences to support organizational decision making in the field
   of computer science.
2. Design, develop, and deliver professional-quality oral, written, and visual
   communications that are coherent, technically sound, and appropriately
   adapted to specific audiences and contexts.
3. Design and evaluate computing solutions that solve a given problem using
   algorithmic principles and computer science practices and standards
   appropriate to its solution, while managing the trade-offs involved in
   design choices.
4. Demonstrate an ability to use well-founded and innovative techniques,
   skills, and tools in computing practices for the purpose of implementing
   computer solutions that deliver value and accomplish industry-specific
   goals.
5. Develop a security mindset that anticipates adversarial exploits in
   software architecture and designs to expose potential vulnerabilities,
   mitigate design flaws, and ensure privacy and enhanced security of data
   and resources.

## Repository Structure

```text
.
├── _config.yml        # Jekyll site configuration (theme, title, exclusions)
├── index.md           # ePortfolio landing page (rendered as the live site)
├── narratives/        # Milestone narratives (Word documents)
└── README.md          # This file — repository orientation
```

## Local Development

This site is built with [Jekyll](https://jekyllrb.com/) using the `minima`
theme and served by GitHub Pages.

### Prerequisites

- Ruby (see [GitHub Pages dependency versions](https://pages.github.com/versions/))
- Bundler
- Jekyll

### Run Locally

```bash
# Install dependencies (first time)
bundle install

# Serve the site at http://127.0.0.1:4000
bundle exec jekyll serve
```

Pushes to the `main` branch are automatically published by GitHub Pages.

## About the Author

<!-- TODO: Replace this placeholder with a 2–3 sentence professional summary.
     The rubric instructs the ePortfolio to "emphasize your selected area of
     specialization within computer science as much as possible." State your
     specialization area (e.g., full-stack web development, secure software
     engineering, mobile development) and the value you bring to employers.
     Keep this concise — the full professional self-assessment lives in
     index.md. -->

Rick Goshen — B.S. Computer Science, Southern New Hampshire University.

## License

Academic work submitted in fulfillment of CS 499 at SNHU. Source code in
linked artifact repositories is licensed separately; see each repository for
details.
