# Contributing to public-software/rfcs

This repository holds the design proposals that cross repositories or change an interface. The process is [RFC-0000](text/0000-rfc-process.md); this page is the short version. The org-wide [CONTRIBUTING](https://github.com/public-software/.github/blob/main/CONTRIBUTING.md) applies here too: every commit is signed off, `pub check` must pass.

## Open an RFC

1. Ask first when the idea is not yet a design: [Discussions](https://github.com/public-software/rfcs/discussions).
2. Copy [`0000-template.md`](0000-template.md) to `text/0000-<slug>.md` and fill every section. The Agent summary is one paragraph a coding agent can act on; How this is verified names the check that proves the RFC is implemented.
3. Open a pull request against `main`. Its number is the RFC number.
4. In the next push, rename the file to `text/<number>-<slug>.md` (four digits, zero-padded) and set `RFC:` and `Pull request:` in the decision record.
5. Request review from the maintainer team (`maint-<repo>`) of every repository the RFC names. Review happens on the pull request; keep the text current with the review threads.

## What happens next

A maintainer of an affected repository proposes the final comment period with the label `rfc/final-comment-period` and a comment naming its end, 10 calendar days later. At the end the pull request is merged (accepted), closed with the reason (rejected) or closed with `rfc/postponed`. An accepted RFC has its decision record completed in the merge and a line in the [decision log](README.md#decision-log).

## Not an RFC

A change inside one repository is an ADR in that repository's `docs/adr/`. A bug is an issue on the repository it affects. A question is a discussion.
