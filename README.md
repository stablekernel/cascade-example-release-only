# cascade-example-release-only

A minimal, runnable example of [cascade](https://github.com/stablekernel/cascade)
driving a release-only pipeline with zero deploy environments.

## What this demonstrates

This repository exercises the smallest end of cascade: a library-style project
that has no environments to promote through, only a release lifecycle. cascade
watches trunk, and on every qualifying change it:

- runs a build callback (`build-lib`),
- generates a changelog with a Contributors section, and
- mints a release-candidate draft, then publishes it as a final release.

The feature slice shown here:

- **Zero environments.** State lives under an implicit `prerelease` key rather
  than per-environment keys.
- **`tag_prefix: rel-`.** Tags are `rel-<version>` (for example `rel-1.0.0-rc.0`
  while a candidate, `rel-1.0.0` once published) instead of the default `v` prefix.
- **Contributor changelog.** Release notes include a Contributors section.
- **State loop.** cascade records the released version and committer back into
  the manifest under `ci.state.prerelease`.

## Layout

| Path | Role |
|------|------|
| `.github/manifest.yaml` | The cascade manifest. Edit this, then regenerate. |
| `.github/workflows/build-lib.yaml` | Stub build callback (echo and sleep). |
| `.github/workflows/orchestrate.yaml` | Generated. Runs on trunk changes; mints the RC draft. |
| `.github/workflows/promote.yaml` | Generated. Publishes the prerelease into a final release. |
| `.github/actions/manage-release/` | Generated composite action used by the workflows. |
| `.github/workflows/scenario-suite.yaml` | Hand-written end-to-end check of the full lifecycle. |

## Regenerating the workflows

The orchestrate and promote workflows and the `manage-release` action are
produced by cascade from the manifest. After editing `.github/manifest.yaml`,
regenerate them:

```sh
cascade generate-workflow --config .github/manifest.yaml --force
```

The cascade CLI version is pinned through the `cli_version` field in the
manifest, which the generated workflows reference when they install the CLI.

## The scenario suite

`scenario-suite.yaml` is a hand-written driver that runs the whole release
lifecycle against this repository and asserts the outcome at each stage. It
resets to a clean slate, commits a change under `src/` to start orchestrate,
asserts that a `rel-*-rc.*` draft and tag appear with populated prerelease
state, dispatches promote, then asserts a published `rel-<semver>` release that
is marked latest, with the candidate tags cleaned up and a Contributors section
in the body.

It runs on a daily schedule and on manual dispatch. It needs a `CASCADE_STATE_TOKEN`
repository secret with permission to push to trunk and manage releases.
