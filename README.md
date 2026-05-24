# privateprompt-rules

Public, versioned, signed recognition rules for [PrivatePrompt](https://github.com/dnc4104/privateprompt) —
a local pseudonymization app for legal documents.

This repo holds the **detection rules only**: regular expressions, place
lists, false-positive denylists, first-name dictionaries, statute citation
patterns, and Presidio recognizer metadata. The app's source code stays
private; this repo is what changes when a false positive is reported or
a new pattern is added.

## What's inside

```
data/
  manifest.json              sha256 hashes of every other file
  recognizers/
    swiss.yaml               AHV, UID, CH phone/passport/address, …
    french.yaml              NIR, SIREN, SIRET, TVA, phone, …
  filters/
    false_positives.yaml     legal-doc denylists (section headings, etc.)
    preserve.yaml            institutions, courts, departments, cantons
  lexicons/
    first_names.yaml         FR + DE first names with gender
    dates.yaml               month tables + date regex
  placeholders.yaml          placeholder type map + delimiters
```

## How updates reach the app

1. Someone PRs an edit (e.g. tighten a regex, add a place name).
2. After merge, maintainer tags `data-pack-vN` and pushes.
3. [`.github/workflows/release.yml`](.github/workflows/release.yml) signs the
   manifest with Ed25519, builds `pack.tar.gz`, publishes a GitHub Release.
4. Users open Settings → "Règles de reconnaissance" → **Vérifier les mises
   à jour** → **Installer v{N}** → restart app.

Updates are **opt-in**. The app never fetches in the background.

## Trust model

Updates ship **rules only, no executable code**. The signing key can
degrade detection quality (loosen regex, plant false positives) but
**cannot execute code** on user machines — validators (AHV checksum, NIR
checksum, Luhn, etc.) live in the app binary and are referenced by name
from the YAML. Adding a new validator requires a real app release.

If the bundled fallback's signature verifies, the user is always at least
as safe as the day they installed the app.

### Public key fingerprint

The current Ed25519 public key (hex):

```
1c1f06647b70b7857609a1f85f616a1fbbc8d436e529d37b09b3fbb22853a487
```

This must match the `PUBLIC_KEY` constant baked into the app binary at
[`src-tauri/src/data_pack_update.rs`](https://github.com/dnc4104/privateprompt/blob/main/src-tauri/src/data_pack_update.rs).
Mismatch → all updates from this repo are rejected (by design, to protect
against attacker-controlled keys).

## Contributing a rule change

1. Fork + branch.
2. Edit the relevant YAML under `data/`. **Don't** edit `manifest.json`
   directly — CI recomputes it on release. (Or run the regenerator script
   from the app repo: `uv run python scripts/dump_data_pack.py`.)
3. Open a PR. Include a short note: what false positive triggered the
   change, with a minimal example string.
4. Maintainer reviews, merges, tags.

## Schema versioning

Every YAML carries `schema_version: 1`. Apps reject packs whose
`schema_version` they don't recognize — so a v2 schema can ship
incompatible field shapes without breaking older app installs (they
silently keep using their bundled v1 pack).

## License

[Choose: MIT / Apache-2.0 / CC0]. Rule lists are themselves data; the
license should make clear they're freely reusable.
