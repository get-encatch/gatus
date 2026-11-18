# Encatch fork of Gatus

This is a modified version of [TwiN/gatus](https://github.com/TwiN/gatus), used for Encatch's
internal monitoring. It is licensed under Apache-2.0 like upstream (see `LICENSE`).

## What we changed

- `encatch/main` is upstream's `v5.38.0` plus upstream pull request
  [TwiN/gatus#1825](https://github.com/TwiN/gatus/pull/1825) (merged as-is, by Arkady Emelyanov):
  conditions can read response headers with `[HEADER].<name>` (case-insensitive), and an
  `age()` function. It changes `config/endpoint/` only.
- `.github/workflows/encatch-publish.yml` builds `ghcr.io/get-encatch/gatus:<tag>` (linux/amd64)
  when a `v*-encatch.*` tag is pushed. Upstream's own publish workflows are disabled in this fork.

## How we use it

Each Encatch API check has the condition `[HEADER].X-Encatch-Id != none` with
`ui.resolve-successful-conditions: true`. It always passes, and it puts the response's
`X-Encatch-Id` (the production trace id) into the alert's conditions list, e.g.
`[HEADER].X-Encatch-Id (d6be2842...) != none`. Avoid double quotes in conditions: Gatus's custom
alert provider inserts condition text into the JSON body without escaping. Avoid `pat(...)` for
the id: values compared with a pattern are cut to 25 characters.

## Updating

When upstream merges header support, drop this fork and return to `ghcr.io/twin/gatus`.
Until then, for a new upstream release: branch from the new tag, merge the PR again
(`git fetch upstream pull/1825/head`), run `TZ=UTC go test ./...` (some storage tests fail in
non-UTC time zones, also upstream), and tag `vX.Y.Z-encatch.N`.
