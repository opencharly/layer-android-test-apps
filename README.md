# layer-android-test-apps

Android test apps for a `kind: android` device.

The `android-test-apps` candy carries Android applications installed onto a
`kind: android` device through the `apk:` package format — the declarative
counterpart to the `charly check adb install-app` probe verb. It ships the
[F-Droid](https://f-droid.org) client (`org.fdroid.fdroid`) as a **committed APK**
(`tests/data/F-Droid.apk`, the canonical client from f-droid.org), pushed over the
goadb sync protocol. A `target: android` deploy applies this layer's `apk:` list
onto the running device; the candy installs nothing into the image itself.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `android-test-apps` |
| Package format | `apk:` (device-scoped, applied by a `target: android` deploy) |
| Committed APK | `tests/data/F-Droid.apk` (`org.fdroid.fdroid`) |
| Install path | goadb push + `pm install` on the device (no apkeep, no `source:`) |
| Image install | none — device-scoped only |
| Service / port | none |

The `apk:` list is applied **only** by a `target: android` deploy — it is skipped
at image build and on every other target. Its observable effect is on the running
device: after the deploy, `pm list packages` reports `org.fdroid.fdroid` and the
app launches.

## How to use it

Compose the layer in the box that backs a `kind: android` device:

```yaml
android-emulator:
  candy:
    base: cachyos
    candy:
      - '@github.com/opencharly/layer-android-sdk:v2026.250.1813'
      - '@github.com/opencharly/layer-android-test-apps:v2026.251.0811'
```

After a `target: android` deploy, verify on the device through charly's
declarative `adb:` check verb — no host `adb` binary required:

```yaml
plan:
  - check: F-Droid is installed on the device
    adb:
      method: shell
      arg: [pm, list, packages, org.fdroid.fdroid]
    context: [runtime]
  - check: the installed F-Droid launches
    adb:
      method: shell
      arg: [sh, -c, "monkey -p org.fdroid.fdroid -c android.intent.category.LAUNCHER 1"]
    context: [runtime]
```

These are exactly the two `context: [runtime]` checks the candy's own `plan:`
carries.

## Layout

- `charly.yml` — the candy manifest: the `apk:` list, an ordered `plan:` of
  runtime `check:` steps, and the embedded `skill:` entity (when present).
- `tests/data/F-Droid.apk` — the committed F-Droid client APK.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: none yet — routed to the skill-authoring batch
  [`opencharly/opencharly#291`](https://github.com/opencharly/opencharly/issues/291);
  meanwhile see `/charly-check:android` for the `apk:` format and
  `target: android` deploy model
- Sibling fixture layer: `layer-android-apidemos` (the same committed-APK path)
- Device interaction: `/charly-check:adb`
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
