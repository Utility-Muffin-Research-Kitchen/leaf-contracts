# Pak Rat Content Paks — Normative Contracts

Date: 2026-08-28

This is the **normative text** for the three interfaces that let a `.pak`
contribute a system and the emulator that runs it:

| Contract | Scope |
| --- | --- |
| **CONTENT-1** | the `provides` block a content pak declares in its `pak.json` |
| **CAT-1** | the effective catalog: generation directories, the selector, and the stamp |
| **STORE-CONTENT-1** | the storefront `content[]` lane |

This public document is complete enough to implement against: it states the
contracts field by field and gives the exact rejection reason for every rule.
Private design notes may explain *why*, but are never required to use these
interfaces.

The machine-checkable half — schemas, fixtures, and a validator that asserts
every invalid fixture fails for the reason named here — is
[`../contracts/leaf-content/`](../contracts/leaf-content/). If a schema
or fixture there disagrees with this file, **this file wins**; fix the
schema or fixture, unless the disagreement reveals that this text needs a
change, in which case follow [Change procedure](#change-procedure).

This contract copies the SVC-1 shape wholesale, including its
sibling-checkout-vs-pinned-SHA rule — see
[`../contracts/leaf-services/README.md`](../contracts/leaf-services/README.md).

---

## CONTENT-1 — the `provides` manifest

### Marker

A content pak is an ordinary pak whose `pak.json` carries a top-level
`provides` object. Nothing else marks it. Whether it is *listed in Apps* is a
separate question answered by the installed pak's `launch.sh`, never by
`provides` and never by the storefront lane:

| Installed pak | Listed in Apps | Contributes catalog content |
| --- | --- | --- |
| `launch.sh` executable, no `provides` | yes | no |
| `launch.sh` executable, `provides` present | yes (hybrid) | yes |
| no `launch.sh`, `provides` present | **no** | yes |
| neither | no | no |

### Location

`provides` is honored only for a pak installed under `Apps/<platform>/`.

A pak under `Apps/shared/` is still **classified** — its `provides` block is
read and refused with `shared-content-unsupported` — so it can never slip
through as an unrecognized pure app. A shared pak with `provides` and no
`launch.sh` stays hidden from Apps; a shared hybrid remains an ordinary
launchable app and contributes nothing (D16).

V1 also honors `provides` only on the **primary** storage source. A pak on a
secondary card is refused with `secondary-source-unsupported`: a system whose
emulator disappears when a card is pulled is a bad first experience, and the
stamp would need mount-state tracking to be correct. `source_id` is already
in the stamp so this can be relaxed later without a format change.

### Shape

```json
{
  "name": "ScummVM",
  "platform": "mlp1",
  "pak_version": "1.0.0",
  "min_leaf_version": "0.11.0",

  "provides": {
    "schema": 1,
    "systems": [
      {
        "id": "SCUMMVM",
        "name": "ScummVM",
        "patterns": ["SCUMMVM"],
        "extensions": ["scummvm", "svm"],
        "archive_extensions": [],
        "archive_mode": "pass_through",
        "playlist_extensions": [],
        "m3u_generation": "none",
        "rom_root": "Roms/SCUMMVM",
        "image_root": "Images/SCUMMVM",
        "default_core": "scummvm",
        "icon_flat": "art/SCUMMVM.png",
        "icon_photographic": null,
        "screenscraper_platform_ids": [123],
        "group": "Computer",
        "bios_directory": null
      }
    ],
    "system_extensions": [],
    "cores": [
      {
        "id": "scummvm",
        "display_name": "ScummVM",
        "type": "retroarch",
        "libretro_name": "scummvm",
        "file_name": "cores/scummvm_libretro.so",
        "info_name": "info/scummvm_libretro.info",
        "config_folder": "ScummVM",
        "supports_menu": true,
        "supports_savestate": false,
        "supports_disk_control": false
      }
    ]
  }
}
```

### `provides` — top level

| Field | Required | Rule | Reason on violation |
| --- | --- | --- | --- |
| `schema` | yes | Exactly the integer `1`. A newer schema is **refused, never guessed at**. | `unknown-schema` |
| `systems` | no | Array, ≤ 32 entries. | `too-many-systems` |
| `system_extensions` | no | Array, ≤ 32 entries. | `too-many-system-extensions` |
| `cores` | no | Array, ≤ 32 entries. | `too-many-cores` |
| — | — | At least one of the three arrays is non-empty. | `empty-provides` |
| — | — | No property outside this table. | `unknown-field` |

### `provides.systems[]`

Every field maps onto the release `systems.json` schema-2 shape, except the
five new ones marked **new**.

| Field | Required | Rule | Reason |
| --- | --- | --- | --- |
| `id` | yes | `^[A-Z0-9_]{2,32}$` | `malformed-system-id` |
| `name` | yes | 1–64 characters | `malformed-system-name` |
| `patterns` | yes | ≥ 1 entry, ≤ 32, each 1–64 chars | `malformed-patterns` |
| `extensions` | yes | ≥ 1 entry, ≤ 64, each `^[a-z0-9_]{1,16}$` | `malformed-extensions` |
| `archive_extensions` | no | ≤ 8, each `^[a-z0-9]{1,8}$`; defaults `[]` | `malformed-archive-extensions` |
| `archive_inner_extensions` | no | ≤ 64, same item rule as `extensions`; defaults to `extensions` | `malformed-archive-inner-extensions` |
| `archive_mode` | no | `pass_through` \| `extract` \| `none`; defaults `pass_through` | `unknown-archive-mode` |
| `file_names` | no | ≤ 32 strings; defaults `[]` | `malformed-file-names` |
| `ignore_file_names` | no | ≤ 32 strings; defaults `[]` | `malformed-ignore-file-names` |
| `playlist_extensions` | no | ≤ 8, each `^[a-z0-9]{1,8}$`; defaults `[]` | `malformed-playlist-extensions` |
| `m3u_generation` | no | `none` \| `auto` \| `manual`; defaults `none` | `unknown-m3u-generation` |
| `default_core` | yes | Core-id shape; must resolve after merge | `malformed-core-id`, `unknown-default-core` |
| `alternate_cores` | no | ≤ 16 core ids; defaults `[]` | `malformed-core-id` |
| `rom_root` | yes | `^Roms/[A-Za-z0-9_-]{1,32}$` | `malformed-rom-root` |
| `image_root` | yes | `^Images/[A-Za-z0-9_-]{1,32}$` | `malformed-image-root` |
| `bios_notes` | no | ≤ 8 strings; defaults `[]` | `malformed-bios-notes` |
| `icon_flat` | **new**, yes | Pak-relative path to an existing regular `.png` | path reasons below |
| `icon_photographic` | **new**, no | Same, or `null` | path reasons below |
| `screenscraper_platform_ids` | **new**, no | ≤ 8 integers, each 1–99999 | `malformed-screenscraper-platform-ids` |
| `group` | **new**, no | 1–32 characters, or `null` | `malformed-group` |
| `bios_directory` | **new**, no | `^[A-Za-z0-9_-]{1,32}$`, or `null` | `malformed-bios-directory` |

`rom_root` must respect the canonical public-folder policy — one public
`Roms/` folder per console, per
[`../canonical-user-system-folders.md`](../canonical-user-system-folders.md).

### `provides.cores[]`

| Field | Required | Rule | Reason |
| --- | --- | --- | --- |
| `id` | yes | `^[a-z0-9_]{2,64}$` | `malformed-core-id` |
| `display_name` | yes | 1–64 characters | `malformed-core-display-name` |
| `type` | yes | `retroarch` \| `path` | `unknown-core-type` |
| `libretro_name` | `retroarch` only | `^[a-z0-9_]{2,64}$` | `malformed-libretro-name` |
| `file_name` | `retroarch` only | Pak-relative path to an existing regular file | `missing-core-file-name`, path reasons |
| `info_name` | `retroarch` only | Pak-relative path to an existing regular `.info` file | `missing-core-info-name`, path reasons |
| `config_folder` | `retroarch` only | Must pass `jw_ra_core_folder_is_safe` | `missing-config-folder`, `unsafe-config-folder` |
| `path` | `path` only | Pak-relative path to an existing regular **executable** | `missing-core-path`, path reasons |
| `supports_menu` | no | boolean; defaults `false` | `malformed-core-flag` |
| `supports_savestate` | no | boolean; defaults `false` | `malformed-core-flag` |
| `supports_disk_control` | no | boolean; defaults `false` | `malformed-core-flag` |
| `needs_swap` | no | boolean; defaults `false` | `malformed-core-flag` |

A `retroarch` core may not carry `path`, and a `path` core may not carry
`libretro_name`, `file_name`, `info_name`, or `config_folder`
(`core-type-field-mismatch`).

`config_folder` is used by RetroArch **verbatim as a FAT32 directory
component** for `Saves/` and `States/`. An unsafe value corrupts the save
layout for the whole device, not just the pak, which is why it is checked
against the same function the release catalog is checked against, and not
merely against a regex.

### `provides.system_extensions[]`

The **only** way a pak may touch an existing system. It has exactly two
fields, so the validator never has to infer intent from which fields happen
to be present:

```json
{ "system_id": "SNES", "add_alternate_cores": ["my_snes_core"] }
```

| Field | Required | Rule | Reason |
| --- | --- | --- | --- |
| `system_id` | yes | System-id shape | `malformed-system-id` |
| `add_alternate_cores` | yes | 1–16 core ids | `malformed-core-id`, `empty-add-alternate-cores` |
| — | — | No other property | `unknown-extension-field` |

There is **no** shape in which a pak restates an existing system's other
properties. A `systems[]` entry whose `id` already exists is a collision, not
an override (D9).

### Path rules

Every path field in `provides` — `icon_flat`, `icon_photographic`,
`file_name`, `info_name`, `path` — is **relative to the pak root**. This is
SVC-1's `run.path` rule verbatim, including its reason slugs:

| Rule | Reason |
| --- | --- |
| Must not be absolute | `absolute-path` |
| Must not contain a `..` component | `path-traversal` |
| Must resolve inside the pak after resolving **every** path component, not just the leaf | `escaping-symlink` |
| Must exist | `missing-file` |
| Must be a regular file | `non-regular-file` |
| A `type: "path"` target must be executable | `not-executable` |

**Absolute paths are not merely inelegant here — they are broken.** MLP1's SD
card mounts at `/mnt/sdcard` or `/media/sdcard1` and the two swap across
reboots. An absolute root baked into a compiled catalog on that card resolves
to a path that may not exist on the next boot. Every runtime path in the
compiled catalog is therefore stored **pak-relative** and resolved against
the contributing pak's live install path at load time.

### Forbidden fields (D15)

These may not be authored anywhere in `provides`. The validator rejects each
with its own reason, and the compiler force-clears them as defense in depth:

| Field | Reason | Why |
| --- | --- | --- |
| `requires_direct_drm` | `forbidden-requires-direct-drm` | `jw_standalone_policy_requires_direct_drm` honors metadata alone — there is no allowlist gate on that path. A wrong claim can wedge the display. |
| `legacy_flat_core` | `forbidden-legacy-flat-core` | Claims ownership of save/state files from RetroArch's historical flat layout. |
| `name_map` | `forbidden-name-map` | Release-owned arcade-name behavior. |
| `status` | `forbidden-status` | The compiler sets `status` from verified on-disk reality, never from a manifest. |

### Warnings

A warning does not refuse anything; it is written to `diagnostics.json`.

| Condition | Warning |
| --- | --- |
| `patterns` contains redundant case variants (`["SCUMMVM", "scummvm"]`) — matching is already case-insensitive | `redundant-case-variant` |

The generated first-party catalog contains such pairs for historical reasons.
That is data, not a rule to imitate.

---

## Merge policy (D9)

The rule is **unique ownership across contributions**, not literal uniqueness
within one entry. A system listing a value twice is harmless; two systems
claiming the same value is the collision. Matching is case-insensitive at
runtime (`jw_ra_string_list_contains_casefold`), so ownership is resolved
**case-folded** over this namespace set:

- system `id`, every `pattern`, `rom_root`, `image_root`
- core `id`, `config_folder`
- materialized `.info` filenames

| Case | Outcome | Reason recorded |
| --- | --- | --- |
| New system, no namespace collision | Accepted | — |
| `system_extensions[]` appending to an existing system | Accepted, appended | — |
| Any namespace collision with the release base | Contribution refused and dropped, offending value logged | `base-collision` |
| Two paks collide with each other on any namespace value | **Both refused.** Never resolved by `readdir` order — a contract that depends on directory iteration is not a contract | `contributor-collision` |
| A system's `default_core` was refused or dropped during merge | **The system is refused too** — a tile that cannot launch is worse than no tile | `default-core-unavailable` |
| A system's `default_core` names a core that never existed | System refused | `unknown-default-core` |
| `system_extensions[].system_id` names no system after merge | Extension dropped, dangling id logged | `dangling-extension-system` |
| `add_alternate_cores` names a core absent after merge | Extension **accepted**, that one name dropped, logged | `dangling-alternate-core` |

A refusal never fails the whole compile. The contribution is dropped,
everything else merges, and the reason is written to the rescan log **and**
to `diagnostics.json`, so the Pak Rat UI and a developer can both see why a
pak did nothing.

Dependency cleanup is part of the merge, not an afterthought, and it is
**iterated to a fixed point**: refusing a core can refuse a system, which can
in turn strand an extension that named it.

---

## CAT-1 — the effective catalog

### Layout

```text
$UMRK_INTERNAL_DATA_PATH/catalog/          # = $SDCARD_PATH/.umrk/<platform>/catalog/
  current                                  one line: "gen-<digest>\n"  ← atomically replaced
  diagnostics.json                         OUTSIDE any generation
  tmp-<random>/                            in-flight compile; never named by `current`
  gen-<digest>/
    systems.json      merged, schema 2, same `platform` as the base
    cores.json        merged
    info/             merged libretro .info files (release + pak)
    stamp.json        provenance
```

A generation directory is written once and **never mutated**. `current` is a
single small file replaced by `rename()` — that *is* atomic, including on
vfat. Replacing a non-empty *directory* by `rename()` is not; it fails
`ENOTEMPTY`, which is why the naive temp-dir swap does not work.

### Selector grammar

`current` contains exactly `gen-` followed by **64 lowercase hexadecimal
digits** and one trailing `\n`. Nothing else. A reader that cannot parse this
exact grammar falls back to release defaults with `selector-malformed`.

### Canonical bytes and digests

Three primitives, fixed here so producer and readers agree byte for byte:

1. **`canonical_bytes(obj)`** — `obj` serialized as JSON with keys sorted by
   Unicode code point, separators `,` and `:` (no spaces), UTF-8, no BOM, no
   escaping of non-ASCII, followed by exactly one `\n`.

2. **`tree_sha256(dir)`** — for each regular file under `dir`, in ascending
   byte order of its `/`-joined relative path, feed:

   ```text
   rel_path_utf8 || 0x00 || uint64_be(len(file_bytes)) || file_bytes
   ```

   The length prefix is what stops `a` + `bc` from hashing the same as `ab` +
   `c`. An empty directory hashes to SHA-256 of the empty string.

3. **`digest`** — `SHA-256(canonical_bytes(stamp))`, hex, lowercase. Because
   the stamp already contains every input hash **and** every output hash,
   this digest identifies the complete generation. The generation directory
   is named `gen-<digest>`.

Ordering inside the stamp is normative: `contributors[]` ascending by
`provider`, and each contributor's `files[]` ascending by `rel`, both in
UTF-8 byte order.

### `stamp.json`

```json
{
  "schema": 1,
  "platform": "mlp1",
  "release_id": "2026-08-28-gabc1234",
  "base": {
    "systems_sha256": "…",
    "cores_sha256": "…",
    "info_sha256": "…"
  },
  "contributors": [
    {
      "provider": "mlp1/ScummVM.pak",
      "source_id": "primary",
      "pak_version": "1.0.0",
      "provides_sha256": "…",
      "files": [
        { "rel": "art/SCUMMVM.png", "sha256": "…" },
        { "rel": "cores/scummvm_libretro.so", "sha256": "…" },
        { "rel": "info/scummvm_libretro.info", "sha256": "…" }
      ]
    }
  ],
  "output": {
    "systems_sha256": "…",
    "cores_sha256": "…",
    "info_sha256": "…"
  }
}
```

- `provider` is the **install path** — the same identity
  `pakrat_installs.install_path` and `apps.pak_dir` key on. It is never a
  resolved absolute root; the whole point is that no absolute path survives a
  reboot.
- `source_id` records which card the pak came from. It **cannot** identify a
  pak: `jw_storage_source.id` is `"primary"` or an index, so every pak on one
  card shares it.
- `provides_sha256` is `SHA-256(canonical_bytes(provides))`.
- `files[]` fingerprints **every declared file**. This is what makes
  swapping or deleting a `.so` without editing `provides` invalidate the
  stamp — the compiler's `status` output asserted that file was present and
  usable.
- `base.info_sha256` covers the release `.info` directory. Those files are
  *materialized* into the generation, so they are compile inputs like any
  other; leaning on `release_id` alone would miss a partial manual copy that
  replaced info files without changing the release identity.
- `release_id` comes from `$UMRK_INTERNAL_DATA_PATH/release.json` via
  `jw_installed_release_read(...).release_id`. A missing, empty, or invalid
  installed release identity prevents activation of an effective generation:
  use valid release defaults and record `release-identity-unavailable`; if
  those defaults are invalid too, return an explicit catalog error. Native
  tests supply an explicit `release.json` fixture or injected identity — the
  compiler never guesses one.

### Two validation levels

| Level | Who | What | When |
| --- | --- | --- | --- |
| **Full provenance** | `jawakad` only | Rehash the release inputs **and every declared contributor file**; compare against `stamp.json` | At startup, and during the contributor-enumeration pass before a ROM scan |
| **Structural** | Jawaka catalog readers, CS | Parse a safe selector name; require that generation and its `stamp.json`; validate stamp schema, `platform`, and installed `release_id`; load the immutable output | Per catalog load |

Readers **never** rehash contributor binaries on a request path. Rehashing
multi-megabyte cores on FAT32 would make a metadata request do compiler work.

CS may cache the structural result keyed by the exact
(`current` value, installed `release_id`) pair, and must revalidate when
either changes.

`mtime` + size may be recorded or used as a future optimization *hint*. It is
**never** proof of equality: FAT timestamp granularity and same-sized
replacement files can otherwise preserve a stale catalog.

### Publication protocol

1. The compiler writes a `tmp-<random>` directory, fsyncs its files, and
   computes the three output hashes.
2. It writes `stamp.json` in canonical form and computes
   `digest = SHA-256(canonical_bytes(stamp))`.
3. After the temp directory and its parent are fsynced, it is renamed to
   `gen-<digest>`.
4. **If `gen-<digest>` already exists**, the compiler compares the existing
   directory's canonical stamp bytes and output hashes against the temp
   directory's.
   - Exact match → **reuse** the existing generation, discard the temp
     directory. This is what makes a no-op recompile free.
   - Mismatch → corruption. Compilation **fails closed**, records
     `generation-digest-conflict` in `diagnostics.json`, and does **not**
     overwrite the existing directory.
5. `current` is replaced by `rename()` of a temp file in the same directory.

**Invalidation ordering.** On a full-provenance mismatch, `jawakad` first
`unlink`s `current` and fsyncs the catalog directory — making every *new*
reader fall back to release defaults — and only then recompiles. A successful
compile atomically publishes a new selector; a crash or failed compile safely
leaves it absent. A reader that already opened an immutable generation may
finish its existing snapshot.

### Fail-closed rules (D2)

| Situation | Behavior | Reason |
| --- | --- | --- |
| `current` absent | Release defaults | `selector-missing` |
| `current` not the exact selector grammar | Release defaults | `selector-malformed` |
| Named generation directory absent | Release defaults | `generation-missing` |
| Generation present but missing one of `systems.json`, `cores.json`, `info/` | Release defaults — a partially written generation is never served | `generation-missing` |
| `stamp.json` absent or unparseable | Release defaults | `stamp-missing` |
| `stamp.schema` not `1` | Release defaults | `stamp-schema-unsupported` |
| `stamp.platform` ≠ running platform | Release defaults | `stamp-platform-mismatch` |
| `stamp.release_id` ≠ installed `release_id` | Release defaults; producer recompiles | `stamp-release-mismatch` |
| Base hash mismatch (producer only) | Invalidate selector, recompile | `stamp-base-mismatch` |
| Contributor hash mismatch (producer only) | Invalidate selector, recompile | `stamp-contributor-mismatch` |
| Installed release identity unreadable | Release defaults | `release-identity-unavailable` |
| Compilation failed | Release defaults, logged loudly | `compile-failed` |
| Release defaults **also** invalid | **Explicit catalog error** | `release-defaults-invalid` |

A cached stale generation is **never** served after `current` changes or
after failed validation. A stale catalog that silently pins a user to a
pre-OTA system list is worse than not shipping the feature, so there is no
stale-data fallback.

`diagnostics.json` lives **outside** every generation, so a compile that
fails entirely still leaves a readable explanation.

### Lifetime

**No finalized-generation garbage collection in v1.** A CS process may
survive a `jawakad` crash, so even daemon startup is not a reader-free
window. Only abandoned `tmp-*` directories — which could never have been
named by `current` — are cleaned. Content addressing already makes no-op
recompiles reuse the same directory. A later GC requires an explicit
reader-coordination design honored by CS and RetroArch, and is added only if
measured storage growth justifies it.

### Compile timing (P0-2)

Contributor enumeration across the platform and shared app lanes, and the
compile itself, run **before `jw__scan_roms`**. `jw_scan_library` takes
**one** immutable catalog snapshot at the top and threads it through both
`jw__scan_roms` and `jw__scan_apps`. Without this, installing a content pak
adds a system the same scan never indexes, and uninstalling leaves a removed
system's games in the DB until some later scan.

`INFO_PATH` is resolved **per game launch**, from the same generation
snapshot that selected the core — not `setenv`'d once at daemon startup. The
process-wide default is inherited by every child, so after a generation swap
every launch would otherwise point at the previous generation.

---

## STORE-CONTENT-1 — the storefront `content[]` lane

```json
{
  "schema": 1,
  "product": "pak-rat",
  "apps": [ "… unchanged …" ],
  "content": [ { "id": "org.umrk.scummvm", "…": "same package shape" } ]
}
```

| Rule | Reason |
| --- | --- |
| `schema` stays `1`. A gate-unaware client parses `apps[]`, ignores the unknown `content` key, and **never sees a content pak at all** | — |
| The **safe-floor rule does not apply within `content[]`**; instead, its legacy mirror and every `versions[]` entry declare `min_leaf_version` | `ungated-content-version` |
| Immutability and append-only history apply unchanged | `immutable-history-violated` |
| Every `content[]` artifact must actually declare `provides` | `content-artifact-without-provides` |
| An id may appear in **exactly one** lane | `id-in-both-lanes` |
| An `apps[]` artifact that declares `provides` is in the wrong lane | `provides-artifact-in-apps-lane` |
| The `apps[]` safe-floor rule is **unchanged** — the exemption above must not leak into it | `missing-safe-floor` |
| `content[].packages[].platform` names a concrete supported platform; `"shared"` is refused | `shared-platform-in-content` |
| `content[].packages[].runtime` is `"leaf"` | `unknown-runtime` |

Everything else about a package — version grammar, `versions[]` ordering and
limits, `min_leaf_version` grammar, artifact completeness, HTTPS rules — is
shared with the ordinary Pak Rat package rules and enforced by the generator
and publication validator.

### Why a new lane and not `kind` on `apps[]`

Every package in `apps[]` needs an ungated **safe floor**, and the generator
rejects an all-gated history. Every content pak must gate on the release that
ships this contract, so no content pak can satisfy that rule. Worse, on an
old client a content-only pak would install and then show a broken tile,
failing at launch with "app launch.sh missing or not executable". The
safe-floor rule exists precisely to prevent that class of outcome; the
`content[]` lane honors its intent instead of inventing a fake ungated
version to route around it.

### Hybrid packages

PortMaster is both an app and a content provider, so the lanes need an
explicit rule:

- **An id lives in exactly one lane.** Both lanes resolve to the same
  `install_path` (`mlp1/PortMaster.pak`), and two catalog entries pointing at
  one install path is a duplicate-identity bug, not a feature.
- **A package that declares `provides` belongs in `content[]`**, whether or
  not it also has a `launch.sh`. It is gated on this contract by
  construction, so `apps[]` — the gate-unaware lane — is the wrong home.
- **"Open" is derived from the installed pak, never from the lane.** Pak Rat
  offers Open iff the installed `.pak` has an executable `launch.sh`. A
  hybrid gets Open; a pure content pak does not; and the rule stays correct
  for a sideloaded pak that is in no lane at all.
- **Lane migration is deliberate.** When PortMaster moves lanes, gate-unaware
  clients stop seeing it in `apps[]`. That is acceptable: they keep their
  working install, and they could not run the new gated versions anyway. The
  validator flags an id present in both lanes as an error. This migration is
  recorded here so it is not rediscovered during the PortMaster phase.

---

## Change procedure

`content-paks-v1`, `effective-catalog-v1`, and `storefront-content-v1` are
frozen once the first content pak publishes. Until then:

1. Change this file first. It is the normative text.
2. Update the schema and the fixture tree in
   [`../contracts/leaf-content/`](../contracts/leaf-content/) to match,
   and re-run `scripts/validate_fixtures.py`.
3. Every new rejection rule gets **exactly one** invalid fixture whose
   `expect.json` names its reason slug.
4. A breaking change after freeze is a **new** version identifier
   (`content-paks-v2`), never a mutation of v1. `provides.schema` is refused
   rather than guessed at precisely so a v2 pak fails loudly on a v1 device.

## Related

- [`../contracts/leaf-content/`](../contracts/leaf-content/) — schemas, fixtures, and reference models
- [`../contracts/leaf-services/README.md`](../contracts/leaf-services/README.md) — the public SVC-1-family contract guide
