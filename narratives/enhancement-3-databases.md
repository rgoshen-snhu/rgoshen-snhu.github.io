---
layout: default
title: "Milestone 4: Enhancement Three: Databases"
description: "Milestone Four — CS 499 Computer Science Capstone"
date: 2026-06-04
---

Rick Goshen

CS-499: CS Capstone

June 4, 2026

> [Download the Word version](milestone_four_narrative_final.docx) (.docx, original submission format)

**Milestone 4: Enhancement Three: Databases**

**1. Briefly describe the artifact. What is it? When was it created?**

The artifact is Weigh to Go!, a weight-tracking application I originally
built as a native Android app for CS 360 (Mobile Architecture and
Programming) in November 2025, and which I am rebuilding as a full-stack
web application for the CS 499 capstone. The web rebuild uses React 19
with TypeScript on the frontend and FastAPI on Python 3.12 with
SQLAlchemy 2.0 and PostgreSQL on the backend, in a polyglot monorepo
that preserves the original Java code under android/ and hosts the
rebuild under web/.

This narrative covers the database enhancement, which hardens and
operationalizes the persistence layer the earlier milestones built on.
Milestone Two delivered the architecture and a CRUD vertical slice,
tagged v0.1.0, and Milestone Three filled it with algorithms and data
structures, tagged v0.2.1. Milestone Four turns the database from a
place the application stores rows into a layer that enforces its own
integrity and can be operated, including a security and compliance audit
trail, database-level constraint hardening across all seven tables, a
migration-discipline review of the full 0001 through 0010 chain, the
final web database-architecture document, and a documented backup and
restore procedure. Milestone Four is tagged v0.3.0.

**2. Justify the inclusion of the artifact in your ePortfolio.**

The category for this milestone is databases, and the most honest way to
demonstrate database competency on this artifact was to make the
existing schema trustworthy rather than to add another feature. The
Android original carried a specific and documented weakness, since value
validation lived only in the application and the database itself would
accept contradictory or impossible rows. Milestone Four is the direct
answer to that weakness, because it pushes integrity down to the
database where it cannot be bypassed and adds the security and
operational concerns that distinguish a database that merely stores data
from one that can be run in production. The work was named in advance in
the SRS (§13.3.1) and the Milestone Four brief, and every piece of it is
traceable to a specific migration, ADR, and test.

***Components that showcase database skills***

The security and compliance audit trail, documented in ADR-0024, adds
the seventh and final table, audit_log, an append-only record of
authentication outcomes and data mutations. Two design choices carry the
database-security argument. The first is that the event vocabulary is a
closed set enforced by the database rather than a free-text column, so
an unknown event type cannot be written:

```sql
CONSTRAINT audit_log_event_type_valid CHECK (event_type IN (
  'auth.register', 'auth.login_succeeded', 'auth.login_failed',
  'auth.logout', 'auth.token_refreshed', 'auth.token_reuse_detected',
  'auth.account_locked', 'weight_entry.created', 'weight_entry.updated',
  'weight_entry.deleted', 'goal.created', 'goal.updated',
  'goal.abandoned', 'preference.updated'
))
```

The second is that the trail must outlive the actor it describes, since
deleting a user must not erase the security history of what that user
did, so the foreign key releases rather than cascades:

```sql
user_id BIGINT REFERENCES users(user_id) ON DELETE SET NULL
```

The table stores no unmasked PII or secrets. It records user_id and, for
failed logins where there is no user, a masked email in metadata,
following the masking strategy in ADR-0011.

Constraint hardening, documented in ADR-0025, is where migration 0010
adds the value-domain checks the audit surfaced. An achievement
threshold, for example, is NULL only for the goal_reached type and
otherwise must be positive, a rule the application assumed but the
database did not enforce:

```sql
CONSTRAINT achievements_threshold_positive
CHECK (threshold IS NULL OR threshold > 0)
```

Each new constraint lives in the SQLAlchemy model's \_\_table_args\_\_,
so the SQLite integration suite enforces it through create_all, and also
in the migration via op.create_check_constraint, which is the production
Postgres path, with a rejection test proving bad data raises
IntegrityError.

Migration discipline across the 0001 through 0010 chain is the next
piece. Every migration has a tested downgrade, and
upgrade-downgrade-upgrade round-trips plus a from-scratch apply against
a clean database run in CI through migration-ci.yml. A schema is only as
trustworthy as its ability to be rebuilt and rolled back, and this
proves it.

The final database-architecture document and the backup and restore
runbook close the set. The architecture document records every
constraint and index with its rationale and an ERD, and the runbook
gives a pg_dump and pg_restore procedure with verification steps and an
explicit security note that a dump is raw PII.

***How the artifact was improved***

The schema went from one that assumed its data was valid to one that
guarantees it. Contradictory rows can no longer be written, the security
history is durable and tamper-resistant by construction, and the
operational question of how you recover this database is now documented
and scripted. The backend carries 682 tests at 98 percent coverage, and
the new shell scripts are themselves covered by the backend's pytest
suite. None of this is visible on the screen, and that is the point of
the database category.

**3. Did you meet the course outcomes you planned to meet in Module
One?**

**Develop a security mindset that anticipates adversarial exploits and
ensures privacy of data. Met, and the lead outcome.** This milestone is
where the artifact's security story moves from the application into the
data layer. The audit trail records the authentication events a reviewer
would need after an incident, retains them past actor deletion, and
stores no secrets or unmasked PII. Constraint hardening removes a whole
class of bad-data-can-persist flaws, and the restore runbook treats a
dump as the sensitive artifact it is. Each is a deliberate response to
how the data could be misused rather than an afterthought.

**Demonstrate well-founded and innovative techniques, skills, and tools
for software engineering and database work. Met.** The work uses the
database's own integrity mechanisms, including CHECK constraints,
foreign-key actions, and partial unique indexes, rather than
re-implementing them in Python. The migrations use the established
constraint-in-model-plus-migration pattern so one definition serves both
the SQLite test engine and production Postgres, and the audit recorder
is wired at the composition root and kept import-isolated so no domain
depends on it.

**Design and evaluate computing solutions while managing trade-offs.
Met.** ADR-0024 and ADR-0025 each name the alternatives and justify the
choice against explicit trade-offs, including ON DELETE SET NULL versus
cascade for audit retention, a CHECK-constrained VARCHAR versus a
Postgres ENUM for the event taxonomy, fail-open auth-event auditing for
availability versus fail-closed data-mutation auditing for integrity,
and which invariants belong at the database versus the application.

**Design, develop, and deliver professional-quality communication.
Met.** Two new architecture decision records, 0024 and 0025, the final
database-architecture document, the backup and restore runbook, the
reconciled SRS, and this narrative communicate the milestone to
instructors, maintainers, and portfolio readers.

**Employ strategies for building collaborative environments. Partial.**
The capstone is solo, but the decision records, the per-slice reviewable
pull requests, the conventional-commit history, and the closeout's
deliberate reconciliation of drift all keep the work in a form that
supports collaboration.

I have no updates to the Module One outcome-coverage plan. The milestone
schedule, with Milestone Two covering software engineering, Milestone
Three covering algorithms and data structures, and Milestone Four
covering databases, maps cleanly to the program outcomes, and Milestone
Four is the database-integrity and security work the earlier narratives
pointed toward.

**4. Reflect on the process of enhancing and modifying the artifact.**

***What I learned***

The clearest lesson was that late-stage database work is mostly
verification and rigor rather than new construction, and that this is a
real skill rather than a consolation prize. Much of what the SRS listed
for this milestone, including CHECK constraints and composite indexes,
had been built incrementally during the earlier milestones, so the
milestone's job was to audit that work, harden the genuine gaps, and
prove the whole chain rolls back and rebuilds. Writing a constraint is
easy. Proving that adding it will not reject an existing row, that its
downgrade is correct, and that it is enforced identically on both the
SQLite test engine and production Postgres is the actual work. I came to
see documented rather than automated, which is how I handled the backup
procedure, as an honest engineering decision about scope rather than a
shortcut.

A second lesson was about the audit trail as a security primitive. The
interesting design questions were about what survives rather than what
columns to store. An audit row has to outlive the user it describes,
which is why it uses ON DELETE SET NULL. It must never become a new
place that leaks PII, which is why it stores user_id and a masked email
rather than the raw address. And it must not be able to turn a logging
hiccup into a denied login, which is why auth-event writes fail open
while data-mutation writes fail closed. Each of those is a small
decision with a security consequence and writing them into ADR-0024
forced me to defend them rather than assume them.

***Challenges I faced***

The most instructive challenge appeared during closeout rather than
implementation. The milestone's own architecture document, freshly
written and reviewed, labeled two migrations under the wrong milestone.
The fact was checkable, because the release tags fix the boundaries,
with v0.1.0 on May 23, 2026, and v0.2.0 on May 29, 2026, and the
migrations in question were authored between them, which makes them
Milestone Three. The lesson was to verify documentation claims against
ground truth, meaning the tags and the files on disk, rather than
against another document, because two documents can be confidently
consistent and both wrong. The reconciliation step exists to catch
exactly that and running it as a deliberate pass turned the docs are
probably right into the docs match the tags.

***How this work prepares me for the rest of the capstone***

Milestone Four closes the three enhancement categories of software
engineering, algorithms and data structures, and databases, and leaves
the artifact in a state I can defend section by section. Every table has
documented constraints, every migration is round-trip tested, the
security-sensitive surfaces are audited, and the operational gap is
documented. The final-project work is now polish and the ePortfolio
rather than new capability, and the per-slice review discipline, which
here caught a documentation error a reader would otherwise have trusted,
is the habit I most want to carry into that final pass.
