# RFC-0002: WIT package versioning and distribution

- RFC: 0002
- Status: accepted
- Decided: 2026-09-03
- Pull request: none; this RFC was accepted by the founding maintainers (`public-software/core`) at bootstrap, like RFC-0000
- Repositories: `public-software/interfaces` (the packages), `public-software/docs` (the registry file), every repository that implements or consumes a `public:*` package
- Supersedes: none
- Superseded by: none
- Implementation: `wit.yml` in `public-software/interfaces`; `/.well-known/wasm-pkg/registry.json` on `https://publicsoftware.dev`; the `pub-interfaces-*` crates

## Agent summary

A cross-repository interface of Public Software is a WIT package `public:<name>@<major>.<minor>.<patch>` under `wit/` of the `interfaces` repository (`wit/` holds `core`; a second package gets `wit/<name>/`), and every world, interface, type, function, import, export and include in it carries `@since(version = <the release that added it>)`; to change a package, add items only in a minor release and gate them with `@since`, mark an item you want gone with `@deprecated(version = <that minor>)` and remove it in the next major release, and reserve patch releases for documentation and comments, never for a change a binding generator would notice; to release, bump the version in every `package` line of the package, make the `@since` gates agree, push the tag `<name>-v<version>` (for `core@0.1.0`: `core-v0.1.0`), and the repository's `wit.yml` fetches the WASI packages with `wkg wit fetch`, parses with `wasm-tools component wit`, builds with `wkg wit build`, checks that the tag names the version the package declares, and publishes the artifact with `wkg publish` to `ghcr.io/public-software/public/<name>:<version>` with a build provenance attestation; to consume a package, run `wkg get public:core@0.1.0` (or add it to a `wit/deps` fetch), which resolves through `https://publicsoftware.dev/.well-known/wasm-pkg/registry.json` with no client configuration, and depend on the bindings crate `pub-interfaces-<name>` whose major.minor equals the package's major.minor; never edit a published version, never publish a package whose items lack a gate, and never hand-pick an OCI path, because the reference is derived by wkg from the package name.

## Summary

WIT packages are the organization's contracts between repositories, so they need a versioning rule a generator can enforce and a distribution channel a consumer can reach without configuration. This RFC fixes both: semantic versioning per package with `@since` and `@deprecated` gates on every item, a release tag per package, an OCI artifact at a reference wkg derives from the package name, a well-known registry file on the organization's domain, and a bindings crate that tracks the package's major.minor.

## Motivation

`public-software/interfaces` is the spine repository every platform component binds; its first package, `public:core@0.1.0` (health, logging, config, clock), ships with this RFC, and seven more are planned. Without a rule, each package would choose its own idea of a breaking change, its own tag name and its own place in a registry, and a consumer would need a configuration file per package. WASI 0.3.0 (released 2026-06-11) shows the shape that works: one package per proposal, `@since` on every item, a versioned OCI artifact per release at `ghcr.io/webassembly/wasi/<name>:<version>`, and a registry file at `wasi.dev/.well-known/wasm-pkg/registry.json` that lets `wkg` resolve `wasi:*` with no configuration. The organization adopts the same shape under its own namespace.

The rule is agent-first: a coding agent that changes an interface must be able to tell, from the diff alone, which version to bump and which gate to write, and a check must be able to refuse the change when it cannot.

## Design

### Package naming and layout

- A package is `public:<name>@<major>.<minor>.<patch>`. The namespace is `WIT_NAMESPACE` of the organization's configuration (`public`); `<name>` is lowercase, hyphenated, and one of the areas the catalog entry of `interfaces` names (`core`, `doc`, `ui`, `plugin`, `identity`, `store`, `media`, `net`, ...). A WIT version is full semver, with major, minor and patch, so `0.1` is not a version.
- The package lives in `public-software/interfaces` under `wit/` (the first package, `core`) or `wit/<name>/` (every package once there are two; `core` moves to `wit/core/` in the release that adds the second). Every `.wit` file of a package repeats the same `package` line.
- One package has one version at a time: several interfaces in one package move together. An interface that must move on its own is a package of its own.
- The WASI packages a world imports are fetched into `wit/deps/` by `wkg wit fetch` and pinned by the committed `wkg.lock`; `wit/deps/` itself is not committed.

### Versioning

| Change | Release | Gate |
|---|---|---|
| A new interface, type, function, import, export or world | minor | `@since(version = <the new minor>)` on the item |
| A new field of a record, case of a variant or member of an enum | major (the shape of an existing type changes) | `@since` on the new item, and the type's users in a major |
| Removing or renaming an item, changing a signature or a type | major | `@deprecated(version = <the last minor>)` first, on the old item, for at least one minor release; removal in the next major |
| A change to a comment, documentation or the manifest | patch | none |
| An item not yet stable | any | `@unstable(feature = <name>)`; unstable items are not part of the contract and may change without a bump |

- Every item carries a gate; a package in which any item lacks one is not releasable. The gate names the release that introduced the item, and a `@since` version is never later than the package version.
- Before `1.0.0` the table applies unchanged, except that a breaking change bumps the minor (there is no major to bump) and says so in the release notes, and the deprecation step may be skipped while the item has no consumer outside `interfaces`. `1.0.0` is declared by an amendment to this RFC once the platform's reference service has run on the package for one release train.
- A published version is immutable: the tag, the artifact and the text at that tag never change. A mistake is fixed in the next version.

### Releases and the OCI reference

- A release is the tag `<name>-v<version>` on `public-software/interfaces`, for example `core-v0.1.0`; one repository holds several packages, so the tag carries the package name. The tag is pushed by a maintainer of `interfaces` (`public-software/maint-interfaces`) after the change merged; release tags are immutable under the organization's rulesets.
- `wit.yml` in `interfaces` runs on every push, pull request and merge-queue entry: `wkg wit fetch`, `wasm-tools component wit wit/`, `wkg wit build`, and a check that the built package is `public:<name>@<version>`. On a release tag its `publish` job checks that the tag names the version the package declares, logs into ghcr.io with the workflow token, runs `wkg publish` with the repository's `.wkg/config.toml`, and attaches a build provenance attestation to the artifact.
- The OCI reference is wkg's own derivation, `{registry}/{namespacePrefix}{namespace}/{name}:{version}`, with the registry `ghcr.io` and the prefix `public-software/`: `ghcr.io/public-software/public/<name>:<version>`, so `public:core@0.1.0` is `ghcr.io/public-software/public/core:0.1.0`. Nobody chooses a path by hand: a consumer's `wkg get` computes the same reference from the package name, and a path a tool cannot compute is a path every consumer must configure.
- The artifact's annotations (`org.opencontainers.image.description`, `licenses`, `url`, `source`, `version`) come from `wkg.toml` in `interfaces`, so the package on the registry says what it is and where it comes from.

### Resolution: the registry file

`https://publicsoftware.dev/.well-known/wasm-pkg/registry.json`, served by the documentation site, is:

```json
{
  "ociRegistry": "ghcr.io",
  "ociNamespacePrefix": "public-software/"
}
```

`wkg` maps a namespace to the domain that serves this file, so a consumer who runs `wkg get public:core@0.1.0` on a clean machine gets the package with no configuration once the organization registers the `public` namespace with its domain in wkg's defaults; until then, and for tools that do not read the file, `.wkg/config.toml` in `interfaces` shows the one-line `[namespace_registries]` entry that says the same. Both the file and the entry are generated from the organization's configuration by the bootstrap kit, so they cannot disagree.

### Bindings crates

- The Rust bindings of a package are the crate `pub-interfaces-<name>`, generated by `wit-bindgen` in `interfaces` and published through the organization's release path (crates.io Trusted Publishing).
- The crate's major.minor equals the package's major.minor; the crate's patch is free, for regenerated bindings, generator upgrades and documentation. A consumer who depends on `pub-interfaces-core = "0.1"` binds `public:core@0.1.x`.
- A bindings crate never adds an item the package does not have; helpers live in other crates.

### The first package

`public:core@0.1.0`, under `wit/` of `interfaces`, on WASI 0.3.0:

- `health`, exported by a component: `enum state { ok, degraded, failing }`, `record report { state, detail }`, `check: func() -> report`.
- `logging`, imported from the host: `enum level { trace, debug, info, warn, error }`, `enabled: func(level, target) -> bool`, `log: func(level, target, message)`.
- `config`, imported from the host: `variant error { upstream(string), invalid(string) }`, `get: func(key) -> result<option<string>, error>`, `get-all: func() -> result<list<tuple<string, string>>, error>`.
- `clock`, imported from the host, on the WASI clock types (`wasi:clocks/types.duration`, `system-clock.instant`, `monotonic-clock.mark`): `now: func() -> instant`, `elapsed: func() -> mark`, `sleep: async func(how-long: duration)`. The host backs it with the WASI clocks in production and with a clock a conformance test sets by hand.
- The worlds `imports` (logging, config, clock, `wasi:cli/environment@0.3.0`) and `component` (`include imports`, `export health`).

### Out of scope

- Wire schemas that are not WIT (JSON Schema, protocol descriptions): they live in `interfaces` too, but their versioning follows the standard they are written in and is decided by the RFC that introduces each.
- A registry other than ghcr.io, or a `warg` registry: deferred; the namespace file is the seam where it would change.

## Alternatives

- **One package `public:platform` for everything.** Rejected: every interface would move on the schedule of the most volatile one, and a host implementing `logging` would have to bump for a change in `store`.
- **The repository's tag (`v0.1.0`) as the release.** Rejected: one repository, several packages; the tag must say which one.
- **Git dependencies (`wit/deps` committed from a git URL) instead of OCI artifacts.** Rejected: a consumer would need the repository, not the package, and the WASI ecosystem distributes WIT as OCI artifacts (`ghcr.io/webassembly/wasi/*`), which every host and generator already fetches.
- **A hand-picked OCI path such as `ghcr.io/public-software/interfaces/<name>`.** Rejected: wkg derives `{prefix}{namespace}/{name}` and cannot be told otherwise per package; a path outside that derivation would make every consumer configure the registry by hand for every package.
- **`@since` only on new items, nothing on the initial release.** Rejected: the WASI packages gate every item from their first release, and a check that requires a gate on every item is simpler than one that must know which items are original.
- **Publishing from a maintainer's machine.** Rejected by RFC-0001's spirit and by the organization's release policy: releases come from workflows without long-lived secrets, with provenance.

## Unresolved questions

- When `1.0.0` is declared: by an amendment, after the platform's reference service has run on `core` for one release train.
- Whether `pub check` should verify the gates (every item gated, no gate later than the package version, a removed item deprecated in the previous minor) by diffing `wasm-tools component wit` output between the last release and the working tree: expected to follow with the conformance test of `core`.
- Registering the `public` namespace in wkg's built-in defaults, so that consumers need neither the registry file nor a config: to be proposed upstream once the first package is public.

## How this is verified

- `wit.yml` on `public-software/interfaces` is green on `main` (fetch, parse, build, package-name check) and, on the tag `core-v0.1.0`, publishes `ghcr.io/public-software/public/core:0.1.0` with an attestation. The bootstrap kit renders the repository's files and its offline tests assert the package line, the four interfaces, the worlds, a gate on every item, the wkg files, the workflow's pins and jobs, and parse the rendered package with `wasm-tools` against the WASI 0.3.0 release when the tool is available.
- `GET https://publicsoftware.dev/.well-known/wasm-pkg/registry.json` returns the file above; the kit generates it into the documentation site and its tests assert the two fields.
- `wkg get public:core@0.1.0` on a machine with no wkg configuration returns the package, once the namespace resolves through the file.
- The handbook (How we work) states the package form, the tag and the reference, and links this RFC.
