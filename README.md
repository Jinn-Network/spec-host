# spec.jinn.network

The public spec surface of the Jinn protocol: every schema, profile, protocol
and facts document that a record cites by identifier. This repository is
deployed as a static site at `https://spec.jinn.network`, with no build step.

**Nothing here is written by hand.** Every document is generated output,
byte-copied from `Jinn-Network/mono`. Edits belong in the owning package in the
mono; a change here would be overwritten and would break the digests that
verification depends on.

## What is served

| | |
| --- | --- |
| Documents | 762 |
| Built from | `Jinn-Network/mono` @ `9e8a1ac7bdba1ac5c87cea9f0d54c7eddeda1591` |
| Release groups | `sealed-platform-v1` (498), `implementations-v1` (262) |

Each release group's inventory is served at `<group>/manifest.json`, binding
every path in that group to a SHA-256 and a media type.

`vercel.json` sets the content type, entity tag and cache lifetime for each
document, and disables URL guessing (`cleanUrls: false`, `trailingSlash: false`).
This is load-bearing, not cosmetic: 34 documents have no file extension, so a
host left to infer their type would serve them wrongly. Documents are immutable
by the identifier rules and are cached as such; the manifests revalidate.

There is no home page. A spec origin serves documents and 404s everything else.

## Deploying

Import the repository into Vercel in the account that owns `jinn.network`, with
**no build command** and the repository root as the output directory, then
attach the domain `spec.jinn.network`. Subsequent pushes deploy automatically.

Verify a deployment with an ordinary document and an extensionless one, which is
where a misconfigured host reveals itself:

    curl -sI https://spec.jinn.network/schemas/delivery.schema.json
    curl -sI https://spec.jinn.network/task-profiles/prediction-forecast/1.0

Both should return `HTTP/2 200` with `content-type: application/json`.

## Regenerating

From a checkout of the mono, build one profile root per stack-published release
group and merge them into a deploy directory:

    node .github/scripts/build-profile-root.mjs \
      --out <roots>/<group> --commit <sha> --release-group <group> --lane stable

Documents across groups are disjoint; only each group's `manifest.json` needs
its own path, which is why they are namespaced by group here.

## Certification, pending

The signing key is provisioned and its public key is published and pinned, but
the fail-closed live-host gate cannot yet pass for both release groups at once —
see `Jinn-Network/mono#3215`. This deployment serves the correct bytes; it is
simply not yet certified as doing so. Certification is CI's job, not a manual
step.
