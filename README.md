# hubcom-tech / versioning

Per-app version manifests consumed by Hubcom mobile apps for the
in-app "update available / required" check.

## Layout

One folder per app, lowercase, matching the app's pubspec name. Each
folder contains an `app-version.json` with the schema below.

```
versioning/
├── README.md
└── <appname>/
    └── app-version.json
```

| App | Path |
| --- | --- |
| Solo POS | [`solopos/app-version.json`](./solopos/app-version.json) |

## Schema (v1)

| Field | Type | Purpose |
| --- | --- | --- |
| `schemaVersion` | int | Currently `1`. Apps reject unknown values to fail safe. |
| `android.latest` / `ios.latest` | semver string | Newest released version. `installed < latest` → soft banner. |
| `latestBuild` | int | Build-number tiebreaker (Flutter `+N` in `pubspec.yaml`). |
| `minimum` / `minimumBuild` | string / int | `installed < minimum` → hard blocking screen, no skip. |
| `storeUrl` | string | Play Store / App Store deep-link. Tap "Update" → `url_launcher` opens this. |

## Severity tiers

| Installed version vs config | Result |
| --- | --- |
| `>= latest` | Silent. |
| `>= minimum` but `< latest` | **Soft banner** — dismissible, re-shows after 1 hour. |
| `< minimum` | **Hard block** — full-screen, no skip, "Update now" is the only action. |

## Adding a new app

1. Create `<appname>/app-version.json` mirroring the schema above
   (lowercase, matches the app's pubspec name).
2. The app's build constant points to the raw URL of that file:
   `https://raw.githubusercontent.com/hubcom-tech/versioning/main/<appname>/app-version.json`.

## Workflow

Bump version → soft banner:

```jsonc
"android": {
  "latest": "1.1.0", "latestBuild": 8,
  "minimum": "1.0.0", "minimumBuild": 1,
  ...
}
```

Force-upgrade everyone (security fix, breaking change):

```jsonc
"android": {
  "latest": "1.1.3", "latestBuild": 11,
  "minimum": "1.1.3", "minimumBuild": 11,
  ...
}
```

No publish workflow — commit + push, every running app picks up the
new manifest on its next 1-hour check (or next launch).
