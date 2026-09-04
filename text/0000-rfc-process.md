# RFC-0000: The RFC process

- RFC: 0000
- Status: accepted
- Decided: 2026-09-03
- Pull request: none; this RFC bootstraps the process and was accepted by the founding maintainers (`public-software/core`)
- Repositories: every repository of `public-software`
- Supersedes: none
- Superseded by: none
- Implementation: this repository

## Agent summary

To propose a change that crosses repositories or changes an interface in Public Software: copy `0000-template.md` to `text/0000-<slug>.md`, fill every section (the Agent summary is one paragraph a coding agent can act on without reading the rest; How this is verified names the check that proves the RFC is implemented), open a pull request against `main` of `public-software/rfcs`, then rename the file to `text/<pull request number, four digits>-<slug>.md`, set `RFC:` in the decision record and push. The pull request is the review; the repository's Discussions are for open questions that are not yet a design. A maintainer of an affected repository moves the RFC into its final comment period with the label `rfc/final-comment-period` and a comment naming the end date, 10 calendar days later; at the end the pull request is merged (accepted), closed with the reason (rejected) or closed with the label `rfc/postponed`. An accepted RFC keeps its decision record at the top of the file and gets a line in the README's decision log. A change inside one repository is not an RFC: it is an ADR in that repository's `docs/adr/`.

## Summary

Design decisions that cross repositories or change an interface are written down as RFCs in this repository before they are built. This RFC defines how an RFC is numbered, where it lives, which states it passes through, who decides, and what the record of a decision looks like. It is the first RFC so that every other decision has a home.

## Motivation

Public Software is many repositories in one suite, built by people and by coding agents. Interfaces between repositories are contracts, and a contract changed in a pull request on one side breaks the other. The organization already says that such changes need an RFC (the org-wide CONTRIBUTING, the handbook, the ADR template of every repository), but until now nothing said what an RFC is. Several first ships of wave 1 are decisions rather than code: the AI contribution policy, the CRDT for the document model, the kernel architecture, the store's wire compatibility. They need the process first.

The process is agent-first as well as human-first: an RFC must be actionable by a coding agent that reads only its summary, and it must say how anyone, human or agent, can verify that it was implemented.

## Design

### Numbering and location

- The RFC number is the pull request number that proposes it, zero-padded to four digits. Numbers are therefore never assigned by hand and never collide.
- The text lives at `text/NNNN-slug.md` in this repository. While the pull request is being opened the file is `text/0000-slug.md`; the author renames it in the first push after the pull request exists.
- `0000` is this RFC. `RFC-0001` (the AI contribution policy) and `RFC-0002` (WIT package versioning and distribution) are the first decisions the organization made with this process; all three were accepted at bootstrap by the founding maintainers, before this repository had pull requests to number them.
- One RFC per pull request. A pull request that touches an accepted RFC's text is an amendment and follows the same lifecycle; a change of substance is a new RFC that supersedes the old one.

### Lifecycle

| State | Where it shows | Entered by | Leaves to |
|---|---|---|---|
| `draft` | an open pull request against `main` | opening the pull request | final comment period, rejected, postponed |
| `final comment period` | the label `rfc/final-comment-period` and a comment naming the end date | a maintainer of an affected repository proposes it once the open questions are settled; the maintainer team of every repository the RFC names, and `public-software/core` when more than one ring is affected, approves the pull request | accepted, rejected, postponed |
| `accepted` | merged into `text/`, decision record completed, a line in the README's decision log | merging the pull request at the end of the final comment period | superseded |
| `rejected` | the pull request closed, a closing comment stating the reason, a line in the decision log | closing the pull request; before the final comment period when the answer is clear, otherwise at its end | none; a new RFC may revisit the question |
| `postponed` | the pull request closed with the label `rfc/postponed` | closing the pull request when the question is right but the time is not | draft, by reopening or by a new pull request |
| `superseded` | the decision record of the old RFC says `Superseded by: RFC-NNNN`; the file stays | accepting the newer RFC, whose pull request also edits the old record | none |

The final comment period lasts 10 calendar days, so that it spans at least five working days. A substantive change to the text during the period restarts it.

### The decision record

Every RFC opens with the same record, before any prose:

```text
- RFC: NNNN
- Status: draft | final comment period | accepted | rejected | postponed | superseded
- Decided: YYYY-MM-DD (when accepted, rejected or postponed)
- Pull request: https://github.com/public-software/rfcs/pull/NNNN
- Repositories: the repositories the RFC changes
- Supersedes: RFC-NNNN or none
- Superseded by: RFC-NNNN or none
- Implementation: the tracking issue, once accepted
```

The record is the machine-readable part of the RFC: a tool, or an agent, can list every accepted RFC that names a repository without reading the prose. The README's decision log repeats the number, title, state, decision date and pull request of every RFC that reached a decision.

### Where discussion happens

- Open questions that are not yet a design go to this repository's Discussions. A discussion that converges becomes an RFC.
- Review happens on the pull request, in review threads on the text. The author keeps the text current with the threads; a reader of the merged RFC must not need the pull request to understand the decision.
- Nothing about an RFC is decided in a channel that is not public and linked from the pull request.

### What every RFC carries

The template, `0000-template.md`, has these sections, and all of them are filled before the final comment period:

1. The decision record.
2. **Agent summary**: one paragraph a coding agent can act on without reading the rest: what to build, where, what must hold, what proves it.
3. **Summary** for a human, in one paragraph.
4. **Motivation**: why now, what it unblocks, what happens if nothing is done.
5. **Design**: enough detail to implement, including interfaces (WIT, schemas, wire formats), behaviour, migration and what is out of scope.
6. **Alternatives**: what else was considered and why not; prior art cited permissively, in line with the `PROVENANCE.md` of the affected repositories.
7. **Unresolved questions**: what the final comment period must settle, and what is deferred to implementation.
8. **How this is verified**: the check that proves the RFC is implemented: a conformance file in `specs`, a test, a `pub check` rule, a CI job. An RFC that cannot say how it is verified is not ready for its final comment period.

Every commit on an RFC pull request is signed off (Developer Certificate of Origin), like every commit in the organization. An RFC written with a coding agent says so the way the AI contribution policy (RFC-0001) prescribes.

### Changing this process

By an RFC that supersedes this one.

## Alternatives

- **Issues instead of pull requests.** An issue cannot be reviewed line by line, and its text is not versioned. Rust, Ember and Yarn use pull requests for the same reason; the organization follows that shape.
- **Numbers assigned by maintainers.** Sequential numbers assigned by hand collide when two RFCs are opened the same week. The pull request number is unique for free.
- **ADRs only.** An ADR lives in one repository and is invisible from the others. Cross-repository decisions need one place, this repository.
- **A bot that tracks final-comment-period sign-off** (rfcbot in rust-lang). Deferred: labels and a dated comment carry the state at the organization's current volume; a bot can be added by an RFC when the volume warrants it.

## Unresolved questions

- Whether `pub check` should validate the file names and decision records under `text/`; expected to follow once the CLI gains repository-kind awareness.
- Whether accepted RFCs should also be published on the documentation site; the handbook links this repository until then.

## How this is verified

- `text/0000-rfc-process.md` and `0000-template.md` exist on `main` of `public-software/rfcs`, with the sections and states named here. The bootstrap kit that provisions the organization renders them and its offline tests assert every state and section.
- Every other file under `text/` is named `NNNN-slug.md` with `NNNN` the number of a merged pull request of this repository; checked in review, and by `pub check` once it learns this repository's layout.
- The labels `rfc/final-comment-period` and `rfc/postponed` exist in the organization's label set.
