# AGENTS.md — layer-android-test-apps

Standalone candy repo for the `android-test-apps` layer. The candy lives in
`charly.yml` at the repo root: the `apk:` list (device-scoped, applied by a
`target: android` deploy), an ordered `plan:` of runtime `check:` steps, and the
embedded `skill:` entity (when present). It carries the F-Droid client as a
committed APK under `tests/data/`; it installs nothing into the image and has no
service of its own.

Canonical files:

- `charly.yml` — the `android-test-apps:` candy entity (and the
  `android-test-apps-skill:` skill entity, when present).
- `tests/data/F-Droid.apk` — the committed F-Droid client APK.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-check:android` — the closest owning skill: the `apk:` layer package
  format, the `kind: android` device substrate, and the `target: android` deploy
  that applies this candy. Load before editing or troubleshooting the candy.
- `/charly-check:adb` — the `adb:` verb whose shared goadb installer this
  candy's `apk:` list rides. Load when changing the install path.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, the `apk:` field). Load before editing any
  entity field or plan step.

There is no dedicated `/charly-*:android-test-apps` owning skill yet — this repo's
candy carries no `skill:` entity. The gap is routed to the named skill-authoring
batch `opencharly/opencharly#291`; when that lands, add the owning skill here.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has no
  per-repo candy gate.
- The `plan:` `check:` steps are `context: [runtime]` device probes (the
  `org.fdroid.fdroid` package is installed and the app launches) — they execute
  after a `target: android` deploy, not at image build.

## Modify this repo

- Edit the `android-test-apps:` candy entity (and its `skill:` entity together,
  when one exists). The skill is the projected usage source, so a change that is
  not mirrored in the skill leaves the corpus stale.
- The app list is the `apk:` field: a committed file (`apk: tests/data/<name>.apk`)
  or a `package:` download by id. Prefer a committed APK from the app's canonical
  source; the install must be venue-agnostic (in-pod image device and remote
  adb endpoint alike). New behaviour claims go in `plan:` as runtime `check:` steps.
- Keep `version:` at the schema stamp the pinned CI charly supports.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
