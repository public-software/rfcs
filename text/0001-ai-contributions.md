# RFC-0001: AI contributions

- RFC: 0001
- Status: accepted
- Decided: 2026-09-03
- Pull request: none; this RFC was accepted by the founding maintainers (`public-software/core`) at bootstrap, like RFC-0000
- Repositories: every repository of `public-software`
- Supersedes: none
- Superseded by: none
- Implementation: `pub check` (the `ai-trailer` rule), the org-wide CONTRIBUTING and pull request template, `PROVENANCE.md` in every repository

## Agent summary

You are welcome to write code for Public Software, alone or with a human; the rules are these. Every commit you helped write carries one trailer per tool, on its own line at the end of the commit message, in the form `Assisted-by: <tool>:<model>` (for example `Assisted-by: claude-code:claude-fable-5-1`), where tool and model are single tokens of letters, digits, dots, hyphens and underscores; never write `Signed-off-by:` yourself, because only the human you work for may certify the Developer Certificate of Origin, and never write `Co-authored-by:` for yourself. Before you open the pull request make sure it carries: a test that fails without the change and passes with it; a `PROVENANCE.md` entry for every specification, conformance suite or permissively licensed reference you consulted (never copyleft source of the module you touched); the trailer on every assisted commit; and a body with the sections What, Why and How this is verified that another agent can act on without reading the diff. Then run `pub check` and the repository's tests locally, open the pull request from the human's account with a conventional-commit title, and answer review threads on the change itself; the merge gates (`pub check`, CI, the agent review's `suite / policy` check, the mutation and property checks) decide whether it merges, not who or what wrote it.

## Summary

Public Software is built by people and by coding agents, on purpose. Agent-assisted and agent-authored pull requests are welcome in every repository. The policy has four parts: disclosure (every assisted commit carries an `Assisted-by:` trailer naming the tool and the model), responsibility (a human owns the Developer Certificate of Origin and answers for the change; an agent never signs off), completeness (a pull request carries tests that fail without the change, provenance, the trailer and an agent-readable summary), and the bar (the merge gates decide what merges, not the reviewer's patience with how the change was made).

## Motivation

In 2026 open source splits into two camps on AI contributions: projects that ban them and projects that tolerate them behind a human who can explain the code. The Linux kernel's policy ([docs.kernel.org/process/coding-assistants](https://docs.kernel.org/process/coding-assistants.html), merged January 2026) allows assistants, requires an `Assisted-by:` trailer and forbids agents from adding `Signed-off-by:`; LLVM's ([llvm.org/docs/AIToolPolicy](https://llvm.org/docs/AIToolPolicy.html)) requires a human in the loop who has read the code and can answer for it, and labels unreviewed output as extractive. Both are policies of tolerance written for projects whose contributors were already there.

Public Software starts from the other end: 57 repositories, a spec-first cleanroom method, a handful of maintainers and a review gate that is itself an agent. The intended contributors are vibe coders, AI-native engineers and autonomous agents run by a human, and the organization cannot staff a review culture that judges each change by its author. What it can do is state the rules once, make them machine-checkable and let the gates decide. Without this RFC every repository would invent its own stance, the `Assisted-by:` trailer that RFC-0000 already refers to would have no format, and the AI-first governance documents, the AGENTS.md of every repository and the trailer check in `pub check` would have nothing to cite.

## Design

### Who may contribute

- A human, alone.
- A human working with a coding agent, in any proportion, from autocomplete to "write the whole thing".
- An agent run by a human: the human's account opens the pull request, or a bot account the human operates and is named on, and the human is the submitter in every rule below.

A pull request that no human stands behind (an autonomous account with no operator named, no `Signed-off-by:` from a person) is closed, not reviewed. Everything else is reviewed like any other change.

### The trailer

Every commit that a coding assistant helped to write carries, in its trailer block at the end of the commit message, one line per tool:

```text
Assisted-by: <tool>:<model>
```

- `tool` names the product (`claude-code`, `codex`, `cursor`, `copilot`, `aider`, ...); `model` names the model it ran (`claude-fable-5-1`, `gpt-5.2`, `gemini-2.5-pro`, ...). Both are single tokens: letters, digits, dots, hyphens and underscores, no spaces.
- Several tools, several lines. A model used through an editor and again through a CLI is two lines.
- A commit with no trailer asserts that no assistant touched it. The pull request template asks the author to confirm that, so the omission is a statement, not an oversight.
- The regex, which `pub check` implements verbatim on every commit of a pull request range and which this file is tested against:

```text
^Assisted-by: [A-Za-z0-9][A-Za-z0-9._-]*:[A-Za-z0-9][A-Za-z0-9._-]*$
```

Examples that pass:

```text
Assisted-by: claude-code:claude-fable-5-1
Assisted-by: cursor:gpt-5.2
Assisted-by: aider:qwen3-coder-480b
```

The kernel's bare form (`Assisted-by: LLM`) does not pass: the organization records which tools and models produce mergeable changes, and a trailer that says only "an LLM" records nothing. Basic tools (git, cargo, an editor without a model) are not listed.

### The Developer Certificate of Origin

- `Signed-off-by:` is a legal statement under the [Developer Certificate of Origin](https://developercertificate.org). Only a human can make it. **An agent must not add `Signed-off-by:`**, and an agent that is configured to sign off on a human's behalf is misconfigured: the trailer is added by the human running `git commit -s`, or by the human's tooling under the human's name at the human's instruction.
- The human who signs off has read the change, can explain it in review, has checked the licensing of anything the agent brought in, and takes responsibility for it. "The agent wrote that part" is not an answer in a review thread.
- An agent is not a co-author. `Co-authored-by:` is for people; the agent's part is what `Assisted-by:` records.
- Commits are also signed (SSH or GPG), as the org-wide CONTRIBUTING already requires. The signature belongs to the human's key.

### What a pull request carries

Every pull request, agent-authored or not, carries:

1. **A test that fails without the change and passes with it.** For a bug, the regression test; for a feature, its behaviour; for a refactor, the existing tests, unchanged and green. The mutation check in CI holds the test to that standard.
2. **Provenance.** Every specification, conformance suite, documentation page or permissively licensed reference the human or the agent consulted is listed in the repository's `PROVENANCE.md`; the prompts that produced the change pointed at those, never at copyleft source of the module touched (the two-team rule of the Charter §09 applies to agents as it applies to people).
3. **The trailer** on every assisted commit, as above.
4. **An agent-readable summary** in the pull request body: What (the change, in one paragraph), Why (the issue, ADR or RFC it implements), How this is verified (the test, the check, the command). Another agent, or a maintainer six months later, acts on the body without reading the diff.
5. **A conventional-commit title**, since squash merges use it.

### The bar

The merge gates are the bar, not the reviewer:

- `pub check` (conventions, layout, the `ai-trailer` rule).
- CI as the reusable workflows define it: build, tests, lint, audit, typos, and the mutation, property and miri checks where they apply.
- The agent review as a required check (`suite / policy`), which holds the change to the repository's AGENTS.md and to this RFC.
- Review by the maintainer team of the repository (`CODEOWNERS`), on the change: its design, its tests, its provenance.

What is not a review objection: that a change was written by an agent, that it was written quickly, that the author is new, or that the author is an agent's operator rather than its typist. What is: a gate that fails, a test that does not fail without the change, provenance that is missing, a body a reader cannot act on, a change nobody can explain. A reviewer who wants a change made says which of these it is.

There is no "good first issue" carve-out: an issue labelled for newcomers is open to a newcomer with an agent as much as to one without.

### Out of scope

- The organization's own agentic workflows (review, triage, maintenance) run under a maintainer's name and follow this RFC as any contributor does; their configuration is an ADR in `public-software/.github`, not this RFC.
- Which models and tools the organization recommends: the AGENTS.md of each repository, rendered from the catalog, says what works there.
- Content licensed CC-BY-4.0 (specs, datasets, prose) follows the same disclosure rule; the DCO rule applies to code and content alike.

## Alternatives

- **Ban AI contributions** (NetBSD, Gentoo, QEMU, and others in the [RedMonk survey](https://redmonk.com/kholterhoff/2026/02/26/generative-ai-policy-landscape-in-open-source/)). Rejected: the organization is built this way and would be banning its own method. The concern behind the bans, unreviewable slop that extracts maintainer time, is met by gates, not by exclusion.
- **The kernel's `Assisted-by: LLM [tools]` form.** Read as a policy, not copied as text; rejected as the format because it discards the information the organization wants: which tool, which model. The word `LLM` is not a tool. The rule that agents never sign off is taken from it unchanged.
- **LLVM's "substantial" threshold** (label only contributions with substantial tool-generated content). Rejected: "substantial" is not machine-checkable and invites arguments about degree. Any assistance is disclosed; a one-line autocomplete carries the same trailer as a whole feature, and the cost of the line is nothing.
- **LLVM's good-first-issue ban.** Rejected: the organization has no newcomer track that an agent would crowd out; the gates hold every change to the same bar.
- **Disclosure in the pull request body only** (a checkbox, no trailer). Rejected: the body is not in the git history; the trailer travels with the commit through squash, rebase and mirror.
- **A `Co-authored-by:` line for the agent** (the GitHub Copilot convention). Rejected: co-authorship names a person who could have signed off; an agent cannot.

## Unresolved questions

- Whether the trailer format should also carry a version or a date of the model (`claude-fable-5-1@2026-08`): deferred until the catalog of tools makes it useful; the regex admits it without change.
- Whether a pull request should state "no assistance" explicitly in a commit trailer rather than by omission: deferred to the pull request template, where the author checks it.
- How the organization's own agentic workflows identify themselves in trailers when they open pull requests: settled by the ADR that introduces each workflow.

## How this is verified

- `text/0001-ai-contributions.md` exists on `main` of `public-software/rfcs` with the trailer format, the regex, the DCO rule and the list of what a pull request carries. The bootstrap kit renders it and its offline tests assert every rule, including that each example trailer in this file matches the regex above.
- `pub check` runs the `ai-trailer` rule on the commits of a pull request range: every `Assisted-by:` line matches the regex, no commit carries a `Signed-off-by:` naming a bot, and a commit that names an assistant anywhere in its message body carries the trailer.
- The org-wide CONTRIBUTING, the pull request template and every repository's `PROVENANCE.md` cite RFC-0001; the agent review's rubric holds a change to it.
