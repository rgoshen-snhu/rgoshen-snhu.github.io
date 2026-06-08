---
layout: default
title: "Milestone 3: Enhancement Two: Algorithms and Data Structure"
description: "Milestone Three — CS 499 Computer Science Capstone"
date: 2026-05-30
---

Rick Goshen

CS-499: CS Capstone

May 30, 2026

> [Download the Word version](milestone_three_narrative_final.docx) (.docx, original submission format)

## 1. Briefly describe the artifact.

The artifact is Weigh to Go!, a weight-tracking application I originally built as a native Android app for CS 360 (Mobile Architecture and Programming) in November 2025, and which I am rebuilding as a full-stack web application for the CS 499 capstone. The web rebuild uses React 19 with TypeScript on the frontend and FastAPI on Python 3.12 with SQLAlchemy 2.0 and PostgreSQL on the backend, in a polyglot monorepo that preserves the original Java code under android/ and hosts the rebuild under web/. The original app logged daily weights, set goals, and tracked progress toward them, and the web rebuild keeps that same feature set while moving it onto a shared backend a future mobile client can reuse.

This narrative covers the algorithms and data structure enhancement, which adds the algorithmic core that Milestone Two deliberately deferred. Milestone Two delivered the architecture and a CRUD vertical slice, tagged v0.1.0. Milestone Three fills that architecture with the domain logic it was designed to host, including goal management, achievement detection for milestones and streaks, weekly rate-of-change and a weight-trend chart, user preferences, composite indexes for the time-series read paths, and a TTL cache for the dashboard read model. Milestone Three is tagged v0.2.0.

## 2. Justify the inclusion of the artifact in your ePortfolio.

The Milestone Two narrative made a specific promise, that Milestone Three would supply the algorithm and data-structure work the architecture was built to receive. That promise named a trend computation for the dashboard chart, a milestone-detection algorithm for goal-progress achievements, a streak-detection algorithm for consecutive-day logging, a composite-index strategy for the time-series read paths, and a TTL-based cache for the dashboard read model. This milestone delivers each of those, which makes the artifact a direct and traceable demonstration of the algorithms and data structure category. The work was named in advance, scoped in the requirements specification, and can be checked against the commits and tests that implement it.

### Components that showcase algorithms and data-structure skills

The streak-detection algorithm, documented in ADR-0022, computes consecutive-day logging streaks with a single-pass longest-run scan over a set-backed, sorted date sequence, which runs in linear time after the sort and uses linear space. The record explains the choice of a longest-run scan over a fixed rolling-window count, and why a set membership structure backs the scan rather than repeated list traversal, since the set answers the question of whether the user logged on a given day in constant time. Choosing the structures before the algorithm is what kept the scan to a single pass and made its cost easy to state.

```python
def _longest_consecutive_run(observation_dates, today):
    days = sorted(d for d in observation_dates if d <= today)
    if not days:
        return 0
    longest = 1
    current = 1
    one_day = timedelta(days=1)
    for prev, curr in zip(days, days[1:], strict=False):
        if curr == prev + one_day:
            current += 1
            longest = max(longest, current)
        else:
            current = 1
    return longest
```

The milestone-detection algorithm, documented in ADR-0019, detects goal-progress milestones on every weight entry. The record captures the detection rule, the data the algorithm reads, and the complexity of running it inline on the write path rather than batch-recomputing history. Running it inline keeps achievements current the moment an entry lands, which is what a user expects when they log a weight and see the milestone fire.

```python
def detect_milestones(goal, current_weight, already_recorded):
    if goal.goal_type == "lose":
        delta = goal.start_value - current_weight
    else:
        delta = current_weight - goal.start_value
    return [t for t in THRESHOLDS if delta >= t and t not in already_recorded]
```

The composite-index strategy, documented in ADR-0021, computes the weekly rate-of-change from two indexed lookups against composite indexes on the weight-history table rather than scanning the user's full series. The record reasons about index column order and which read paths each index serves, which is a data-structure decision expressed at the database layer. The two lookups stay fast as a user's history grows because the index, not the application, does the narrowing work.

```python
def upgrade() -> None:
    op.create_index(
        "idx_weight_entries_user_created_at",
        "weight_entries",
        ["user_id", "created_at"],
        postgresql_where=sa.text("is_deleted = FALSE"),
    )
```

The opaque compound cursor, documented in ADR-0015, carries in from Milestone Two as the data-structure exemplar this narrative is asked to cite. The cursor is opaque and compound, and it paginates the time-series table without leaking schema or skipping rows at date-tie boundaries. The goal-history and trend reads in Milestone Three build directly on that cursor design.

The TTL cache, documented in ADR-0023, caches the dashboard summary in a small structure, a dictionary keyed by user id that maps to value and monotonic-deadline pairs, with constant-time get and set, a bounded maximum size with expired-first eviction, lazy expiry on read, and invalidation on the writes that change the summary. The record documents the staleness-versus-cost trade-off and the invalidation policy explicitly. The key is strictly per-user, which is the detail that keeps one user's cached summary from ever reaching another.

The preferences storage, documented in ADR-0020, and the trend chart, documented in DDR-0006, round out the set. A key-value preference structure drives display, including preference-driven unit conversion, and the dashboard renders a weight-trend chart over the time series. Together they turn the stored series into something a user reads at a glance rather than a table of raw numbers.

### How the artifact was improved

Milestone Three turned a CRUD application into one that reasons about a user's data, detecting achievements, computing rate of change, recognizing streaks, and visualizing trends. Each capability is framework-free domain logic with documented time and space complexity, written test-first. The work landed as four independent vertical slices, each independently code-reviewed and security-reviewed before merge. That review pass caught a real concurrency defect in the cache, a non-atomic expiry that could raise under the threadpool, and it confirmed the cache key is strictly per-user, which matters because a shared cache is exactly where a cross-user data leak would hide. The backend carries 592 tests at 97 percent coverage, the frontend carries 377 tests, and the end-to-end suite carries 19 Playwright specs.

## 3. Did you meet the course outcomes you planned to meet in Module One?

**Design and evaluate computing solutions using algorithmic principles while managing trade-offs. Met.** This is the milestone's category and the outcome that Milestone Two left only partially addressed. Each algorithm ships with a decision record that names the alternatives and justifies the choice against explicit trade-offs, including longest-run versus rolling-window for streaks (ADR-0022), inline-on-write versus batch recomputation for milestone detection (ADR-0019), index column order and coverage for the composite-index strategy (ADR-0021), and staleness-bound versus compute-cost for the TTL cache (ADR-0023). The managing-trade-offs language in the outcome is satisfied because each record states, for its algorithm, what was given up and why.

**Well-founded and innovative techniques, skills, and tools. Met.** The trend chart uses Recharts, the read paths use composite indexes proven with EXPLAIN tests, and the cache uses a monotonic-clock TTL structure rather than wall-clock arithmetic. Each is a current and defensible tool choice rather than a hand-rolled substitute.

**Security mindset. Met and reinforced.** Every Milestone Three slice passed an independent security review before merge. The reviews verified per-user cache-key isolation, cleared the goal-history and achievement listings of object-level authorization gaps, and confirmed the new query filters are parameterized. The one concurrency defect found was fixed with a test that reproduces it.

**Professional-quality communication. Met.** Five new architecture decision records (0019 through 0023) and three new database design records (0006 through 0008) document the milestone's decisions. The requirements specification, README set, OpenAPI snapshot, and this narrative communicate them to instructors, maintainers, and portfolio readers.

**Collaborative environments. Partial.** As in Milestone Two, the capstone is solo, but the decision records, the reviewable per-slice pull requests, the conventional-commit history, and the deferred-work issues filed during closeout all demonstrate work produced in a form that supports collaboration.

I have no updates to the Module One outcome-coverage plan. The milestone schedule, with Milestone Two covering software engineering, Milestone Three covering algorithms and data structures, and Milestone Four covering databases, continues to map cleanly to the program outcomes, and the Milestone Three work is exactly the algorithm and data-structure backlog the Milestone Two narrative named. The one shift worth noting is in execution rather than coverage, since the security outcome earned more than I planned through the per-slice review that caught the cache defect.

## 4. Reflect on the process of enhancing and modifying the artifact.

### What I learned

The most useful lesson was that the cost of an algorithm lives in its invariants. The happy path is the cheap part. The TTL cache read clean in isolation, but because the sync request handlers run in a threadpool, two requests reading the same just-expired key could both try to delete it and the loser would raise. The fix was one line, an eviction with a pop that tolerates a missing key, but the lesson was that any structure with lazy eviction and shared state needs to be reasoned about under concurrency, not just under a single caller. I now treat the question of what happens when two callers hit the same code at the same instant as a default for any cache or counter.

```python
def get(self, key):
    entry = self._store.get(key)
    if entry is None:
        return None
    if self._now() >= entry.expires_at:
        # pop(key, None) makes lazy eviction idempotent: under the AnyIO
        # threadpool, two threads can read the same expired entry; del would
        # raise KeyError on the loser whereas pop is a no-op.
        self._store.pop(key, None)
        return None
    return entry.value
```

A second lesson was about where a data-structure decision actually belongs. The weekly rate-of-change could have been computed in Python by pulling the user's series and walking it. Instead it became a composite-index decision at the database layer in ADR-0021, so the work the index does never reaches the application. Writing that as a decision record forced me to articulate the index column order against the specific read paths, which is a more honest demonstration of data-structure thinking than the equivalent in-memory loop would have been.

### Challenges I faced

The streak-detection design was the most instructive. The naive version recomputed a rolling window on every read. The ADR-0022 work replaced it with a single-pass longest-run scan over a set-backed sorted date sequence, which is both cheaper and a clearer expression of what a streak actually is. Choosing the data structure first, a set for membership and a sorted sequence for the scan, made the algorithm fall out almost for free, which reinforced that the structure choice is the real design decision.

The closeout itself surfaced a different kind of challenge, documentation drift. The OpenAPI snapshot had fallen four phases behind the live routes, the database-design-record index was missing its two newest records, and the requirements specification appendix still listed the Milestone Three records under their planned and since-reassigned numbers. None of these were code defects, but each would mislead a future reader. The closeout pass exists to catch exactly that class of rot, and running it as a deliberate step, regenerating the snapshot from the live app and reconciling the indexes against what is on disk, turned the assumption that documentation is probably fine into the confidence that documentation is verified.

### How this work prepares me for the rest of the capstone

Milestone Three leaves the same kind of clean seams for Milestone Four that Milestone Two left for Milestone Three. The composite indexes and the EXPLAIN-backed proof of their use are the natural starting point for the Milestone Four indexing and constraint work. The TTL cache's documented invalidation policy, along with the two deferred items filed as issues during closeout, the weight update and delete invalidation and the cross-worker cache question, give Milestone Four a concrete and tracked backlog rather than an implicit one. And the per-slice review discipline that caught the cache concurrency defect is the habit I most want to carry into the database work, where the failure modes are quieter and the cost of finding them late is higher.
