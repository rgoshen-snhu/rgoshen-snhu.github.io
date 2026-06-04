---
layout: page
title: "Milestone 2: Enhancement One: Software Design/Engineering"
date: 2026-06-04
---

Rick Goshen

CS-499: CS Capstone

June 4, 2026

> [Download the Word version](milestone_two_narrative_final.docx) (.docx, original submission format)

## 1. Briefly describe the artifact.

The artifact is Weigh to Go!, the weight-tracking app I built as a native Android application for CS 360 (Mobile Architecture and Programming) in November 2025. I wrote it in Java with XML layouts, used SQLite for local persistence, and pulled in Material Components for the UI. The app went through a formal three-phase build that covered proposal, UI prototype, and full functional implementation. The feature set covers user registration and login, daily weight logging with unit selection, weight history, goal setting with progress tracking, push and SMS notifications for goal achievements, and basic accessibility settings.

For the CS 499 capstone I am rebuilding Weigh to Go! as a full-stack web application while keeping the original Android codebase in the same repository as a historical reference. The web rebuild uses React 19 with TypeScript and Material UI v9 on the frontend, FastAPI on Python 3.12 with SQLAlchemy 2.0 and Pydantic v2 on the backend, and PostgreSQL for persistence. Both stacks live in a polyglot monorepo, with android/ holding the preserved Java code and web/ holding the new Python and TypeScript code.

This narrative covers Milestone Two, the software design and engineering enhancement, which delivered the architectural foundation, the authentication system, weight-entry CRUD, and the dashboard summary. That work is the vertical slice that proves the architecture end-to-end. Milestone Two is tagged v0.1.0 in the repository.

## 2. Justify the inclusion of the artifact in your ePortfolio.

I picked Weigh to Go! because the original Android version had concrete architectural debt I uncovered during the structured code review at the start of the capstone. The findings were not surface-level cleanup items. They were architectural in nature and included over-large activity classes with single-responsibility violations, model-view-controller drift, database writes on the UI thread, an ExecutorService lifecycle defect, unmasked PII in logs, username enumeration through differentiated authentication errors, a hardcoded navigation chain, and accessibility gaps. Those items could not be patched in place without leaving the underlying design problems untouched. The full-stack web rebuild lets me address every finding through new design choices rather than incremental patches. The web stack also lines up with the direction I am taking my career, which made the rebuild the natural choice for the capstone.

### Components that showcase software-engineering skills

The backend layers three architectural patterns together, documented in ADR-0012. The folder structure follows Screaming Architecture, with directories named for bounded contexts like auth/, weight_tracking/, goals/, and users/, so the project layout reads like a domain map instead of a framework template. Clean Architecture enforces the dependency rule through import-linter in CI, which means any domain code that imports FastAPI or SQLAlchemy will break the build. Hexagonal Architecture defines ports in the domain and keeps adapters in infrastructure/, so the I/O layer stays swappable without touching the business logic. The three patterns reinforce each other rather than fight, because each one targets a different axis of the design.

```toml
[[tool.importlinter.contracts]]
name = "auth: domain and application must not import frameworks"
type = "forbidden"
source_modules = [
    "weighttogo.auth.domain",
    "weighttogo.auth.application",
]
forbidden_modules = ["fastapi", "sqlalchemy", "pydantic", "alembic", "starlette"]
```

Security is applied from the first commit rather than bolted on later. Email is the primary user identifier (ADR-0009), authentication errors stay generic to prevent username enumeration (ADR-0010), every log statement masks PII (ADR-0011), and refresh tokens rotate with family-based revocation so that token replay is detectable (ADR-0013). Bcrypt at cost 12 hashes passwords, account lockout protects against brute force, and the authentication endpoints are rate-limited. Every finding from the Android code review has a matching countermeasure, with a regression test for each one. The security tests live alongside the unit tests, so anyone running the full suite exercises them by default.

```python
class AuthenticateUser:
    """Verify login credentials and return the authenticated user.

    Raises:
        InvalidCredentialsError: For any credential mismatch, unknown email,
            or inactive account (generic - no enumeration).
        AccountLockedError: When the account is in an active lockout period.
    """
```

Test-driven development carried the entire milestone. The diff is 544 files, +28,479 lines and −598 lines across 173 commits, including 277 backend tests in pytest, 241 frontend test cases in vitest, and five end-to-end Playwright specs. Coverage hits or exceeds the threshold in SRS §11. The commit history shows the red-green-refactor cadence as it happened, with the test commits landing before the implementation commits in every feature path.

All API paths return RFC 7807 problem-details errors, with field-level validation on 422 responses. Both the backend emission and the frontend parsing are tested. Pagination on GET /api/v1/weight-entries uses an opaque compound cursor (ADR-0015) that came out of the PR #30 code review, where the original transparent-cursor design was leaking schema and skipping rows at page boundaries. I authored the ADR mid-PR, and that let me capture the bug and the corrected design in the same document. The sequence of bug, ADR, fix, regression test became the template I now use whenever a code review surfaces a design-level problem. It is faster than rewriting the design twice.

```python
def build_problem_detail(
    *, status: int, title: str, detail: str, instance: str,
    errors: list[dict[str, str]] | None = None, request_id: str | None = None,
) -> dict[str, object]:
    return {
        "type": "about:blank", "title": title, "status": status,
        "detail": detail, "instance": instance,
        "errors": errors, "request_id": request_id,
    }

def encode_cursor(observation_date: date, entry_id: int) -> str:
    payload = f"{observation_date.isoformat()}:{entry_id}".encode("ascii")
    return base64.urlsafe_b64encode(payload).rstrip(b"=").decode("ascii")
```

Documentation discipline runs through the whole milestone. A versioned Software Requirements Specification at /docs/specs/WeighToGo_Web_SRS_v2.md and nine new Architecture Decision Records (ADR-0007 through ADR-0015) together cover the architecture, requirements, API contracts, quality gates, and every M2 decision along with the alternatives considered. A reverse-chronological SUMMARY.md records the rationale and bug-fix context for each commit, and it has become my go-to document when re-entering the project after a break.

### How the artifact was improved

The web rebuild closes every finding from the Android code review. The three-pattern backend, ports and adapters, async I/O, and structured logging eliminate the structural issues. The security work closes the PII exposure and the authentication enumeration vulnerabilities that the original Android app had, and the TDD cadence produces a codebase verified by tests rather than by hand. The OpenAPI snapshot from FastAPI gives any future client a contract to consume without guessing, whether that client is a mobile rebuild or a third-party integration.

## 3. Did you meet the course outcomes you planned to meet in Module One?

The Milestone Two work targets three of the five program outcomes most directly and makes incidental progress on the other two. The three direct targets are the software engineering, security, and communication outcomes, which align with the focus of the software design and engineering enhancement category. The algorithm work and the collaborative outcome show up in partial form, with the bulk of the algorithm work scheduled for Milestone Three.

**Well-founded techniques and tools (Outcome 4). Met.** The web rebuild stack is React with TypeScript and Material UI on the frontend, FastAPI with Pydantic and SQLAlchemy on the backend, PostgreSQL for persistence, and a tooling layer with Vite, ruff, mypy, eslint, prettier, pre-commit, and Playwright. These are all current industry-standard tools that match the kind of stack I work with in production. The three-pattern backend, strict typing on both stacks, and a test-first cadence are practices that hold up past a course project. None of the choices here would look out of place in a code review at work.

**Security mindset (Outcome 5). Met.** The authentication system addresses every security finding from the Android code review. Each finding has a documented countermeasure and a regression test backing it up. Bcrypt password hashing, refresh-token rotation with family revocation, generic error messages, PII masking, rate limiting, account lockout, strict CORS, and security headers are all in place from the first commit, with an ADR for each decision.

**Professional written and visual communication (Outcome 2). Met.** The SRS, the ADR set, the OpenAPI-generated API documentation, the README, the CONTRIBUTING guide, the ARCHITECTURE summary, the documentation index, and this narrative together show that I can communicate engineering decisions in writing for both internal and external audiences. The same discipline runs through the commit history and PR descriptions. Every PR explains the change, the tests it touches, and the trade-offs a reviewer should look at first. Documentation is not an afterthought on the project, it is part of the deliverable.

**Algorithmic principles and trade-offs (Outcome 3). Partial.** Milestone Two defers most of the algorithm and data-structure work to Milestone Three by design. The M3 backlog (named in the M2 implementation brief §7 Out of Scope and in the SRS) covers a sliding-window moving average for trend smoothing on the weight chart, a milestone-detection algorithm for goal-progress achievements, a streak-detection algorithm for consecutive-day logging at 7 and 30 days, a composite-index strategy for trend queries on the time-series table, and TTL-based server-side caching for the dashboard read model. The M2 contribution to this outcome is the opaque compound cursor for weight-entry pagination in ADR-0015. The ADR documents the trade-offs between offset and cursor pagination and between transparent and opaque cursors, then justifies the choice. That cursor design is meant to generalize to the broader time-series pagination work coming in M3.

**Collaborative environments (Outcome 1). Partial.** The ADRs, the self-applied code review checklist, and the README written for an external audience all contribute. The capstone is a solo project, so the full collaborative outcome will show up at work rather than in this artifact. The work itself is structured the way collaborative work needs to be, with decision records, documentation, a clean PR history, and conventional commits.

I have no updates to the Module One outcome-coverage plan. The milestone schedule still maps cleanly to the program outcomes, with M2 covering software engineering, M3 covering algorithms, and M4 covering databases. The items deferred from M2 are exactly the ones planned to land in M3.

## 4. Reflect on the process of enhancing and modifying the artifact.

### What I learned during the rebuild

The biggest technical surprise was how much architectural drift becomes visible the moment I started writing the test before the code and enforcing the dependency rule in CI. The three-pattern backend (ADR-0012) sounded like ceremony on paper, but once import-linter was enforcing the rule that domain code cannot import FastAPI or SQLAlchemy, the architecture stopped being aspirational. The first time I tried to shortcut by reaching into the ORM from a use case, the CI run failed in under a minute. After a few of those, the pattern stuck. The same thing happened with mypy --strict. It forced me to make every public function's contract explicit, and the cost up front paid back every time I refactored a caller and the compiler caught the break instead of a runtime exception.

The non-technical lesson was the value of writing the Architecture Decision Record before the code that depends on it. ADR-0012 was committed before Phase 4 wrote a single line of the domain folder structure, and ADR-0013 (refresh-token rotation) was committed before Phase 6 began the auth backend. In both cases the act of writing the ADR forced me to think through the alternatives, and in both cases the chosen alternative ended up slightly different from what I would have built on instinct. If I had written the ADR after the code, the record would have read as a justification for what I already built, instead of a real decision I had worked through up front. The SUMMARY.md engineering log proved more useful than I expected for the same reason. Commit messages tell you what changed. SUMMARY.md entries tell you why it changed and what went wrong along the way, with enough room to actually explain. By the end of Phase 9, that log had become the single most useful document for re-entering work after a break.

### Challenges I faced

The most instructive challenge was the cursor pagination contract on GET /api/v1/weight-entries. The first-cut implementation in Phase 8 used a transparent entry_id cursor, and the PR #30 review surfaced two problems with that design. The cursor was leaking schema, and it was skipping rows at page boundaries when two entries shared a date. The fix took two passes. First I patched the boundary condition, then I realized the design itself was wrong and authored ADR-0015 (opaque compound cursor) to capture the corrected contract. The lesson stuck. Pagination contracts are public API surface and need an ADR up front, before a code review has to catch the holes.

The same PR #30 review caught a soft-delete filter bug in get_by_id. The method was returning soft-deleted entries, which meant a DELETE followed by a PUT would silently resurrect a deleted row. Commit ec22cf2 fixed it with regression tests for GET, PUT, and DELETE on soft-deleted entries, plus a separate get_by_id_including_deleted helper for the genuinely idempotent DELETE path. The class of bug stuck with me too. Soft delete is a domain invariant. I had been treating it as a query filter, and that is something I now look for explicitly whenever I see is_deleted on a model.

```python
def get_by_id(self, entry_id: int, user_id: int) -> WeightEntry | None:
    """Look up an active entry by primary key, scoped to user_id.

    Soft-deleted rows are excluded per the port contract.
    """
    row = (
        self._session.query(WeightEntryModel)
        .filter_by(entry_id=entry_id, user_id=user_id, is_deleted=False)
        .first()
    )
    return _entry_to_domain(row) if row else None
```

A third challenge worth naming is the Phase 9 release-automation work itself. My first design used git-cliff with a manual git tag step to publish v0.1.0. A closeout review caught the problem. The manual git tag step reintroduces the human-error vector the release pipeline was supposed to remove. I reverted three commits and switched to release-please, which moves the version decision into a reviewable Release PR. The meta-lesson was about how I choose tooling in the first place. When I am building automation, my default question now is which human decisions are still in the loop and whether they actually need to be there. The simplest renderer that fits the task is the wrong starting question.

### How this work prepares me for the rest of the capstone

Milestone Two was scoped to leave clean seams for M3 and M4 to plug into instead of forcing later retrofit work. The opaque compound cursor from ADR-0015 generalizes to the other time-series queries M3 will need, including trend windows and goal-history pagination. The three-pattern backend gives M3's algorithm modules a domain folder to live in, where the sliding-window moving average, milestone detection, and streak detection can be tested in isolation without the framework. The structured logging strategy from ADR-0011 is the natural hook point for the M4 audit log, since the routing layer already produces the events and M4 just needs to persist them. The release-please pipeline that landed in Phase 9 means v0.2.0 and v0.3.0 ship on the same automation as v0.1.0 with no additional release engineering. The marginal cost of shipping a milestone is now reviewing one PR.
