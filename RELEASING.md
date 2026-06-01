# Releasing

This repository publishes two composite actions (`cpp` and `rust`) under a single
shared version. A release is a git tag of the form `vMAJOR.MINOR.PATCH` that
applies to both actions in lockstep.

## Versioning scheme

We follow [semver](https://semver.org/):

- **MAJOR** — backwards-incompatible change to either action's inputs, outputs,
  or runtime behavior (e.g. renaming/removing an input, changing a default in a
  way that breaks existing callers, requiring a new permission).
- **MINOR** — new inputs/outputs or new capabilities, with existing usage still
  working unchanged.
- **PATCH** — bug fixes and internal changes only.

Consumers are expected to pin to the moving major-version tag (`@v1`). That tag
is force-updated on every release within the major line, so users on `@v1`
automatically pick up minor and patch releases.

## Cutting a release

1. Make sure `main` is green and you're at the commit you want to release.
2. Decide the next version per the scheme above. If unsure, default to a patch
   bump.
3. Create and push the tag:

   ```sh
   git tag -a v1.2.3 -m "v1.2.3"
   git push origin v1.2.3
   ```

   The `Release` workflow (`.github/workflows/release.yml`) runs on tag push and
   will:
   - force-update the matching major tag (`v1`) to point at `v1.2.3`,
   - create a GitHub Release with auto-generated notes.

4. Verify on the [Releases page](../../releases) that the new release exists and
   that `v1` now points at the new commit.

## Re-running for an existing tag

If the workflow fails partway through (e.g. release created but major tag not
moved), trigger it manually from the Actions tab → **Release** → **Run workflow**,
and supply the existing tag (e.g. `v1.2.3`). It's safe to re-run.

## Breaking changes (new major)

When introducing a breaking change:

1. Tag `v2.0.0` as usual. The workflow will create the `v2` major tag the first
   time it sees a `v2.x.y` release.
2. The `v1` tag is left frozen at the last `v1.x.y` release — existing consumers
   on `@v1` stay on the old major until they explicitly move to `@v2`.
3. Update the README usage examples to reference the new major.

## Yanking a release

GitHub doesn't have a true "yank," but you can:

1. Delete the bad release from the Releases page.
2. Move the major tag back to the previous good release:

   ```sh
   git tag -f v1 v1.2.2
   git push origin v1 --force
   ```

3. Optionally delete the bad tag (`git push origin :refs/tags/v1.2.3`). Note
   that anyone who already pinned to `v1.2.3` exactly will break — prefer
   superseding with a new patch release over deleting.
