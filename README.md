# spec.jinn.network

The public spec surface of the Jinn protocol: every schema, profile, protocol
and facts document that a record cites by identifier. This repository is
deployed as a static site at `https://spec.jinn.network`, with no build step.

**Nothing here is written by hand.** Every file is byte-copied from a
CI-attested, signed artifact of `Jinn-Network/mono`, assembled by the
repository's own `build-profile-host-bundle.mjs`. Edits belong in the owning
package in the mono; a change here would be overwritten and would break the
digests and signatures that verification depends on.

## What is served

| | |
| --- | --- |
| Documents | 777 |
| Built from | `Jinn-Network/mono` @ `275b4bb8474927b8aeaa9dcf44dffeadae49dbb3` |
| Release groups | `sealed-platform-v1` (498), `implementations-v1` (279) |
| Signed | yes — each group's `manifest.dsse.json`, key id `jinn-profile-manifest-2026-08` |

Each release group's inventory is served at `<group>/manifest.json`, with a
DSSE signature sidecar beside it, verifiable against the public key published
in the mono (digest-pinned in the repository variables). Nothing is served at
the origin root: the per-group namespace is what lets two groups share one
host (Jinn-Network/mono#3215), and the live-host gate probes that a root
`manifest.json` stays 404.

`vercel.json` sets the content type, entity tag and cache lifetime for every
document and disables URL guessing (`cleanUrls: false`, `trailingSlash:
false`). This is load-bearing: 51 documents have no file extension. Documents
are immutable by the identifier rules and cached as such; the group manifests
and sidecars revalidate.

There is no home page. A spec origin serves documents and 404s everything else.

## Deploying

Import the repository into Vercel in the account that owns `jinn.network`,
with **no build command** and the repository root as the output directory,
then attach the domain `spec.jinn.network`. Subsequent pushes deploy
automatically.

Verify a deployment with an ordinary document and an extensionless one:

    curl -sI https://spec.jinn.network/schemas/delivery.schema.json
    curl -sI https://spec.jinn.network/task-profiles/prediction-forecast/1.0

Both should return `HTTP/2 200` with `content-type: application/json`. Also
confirm the root inventory stays absent — this must be a 404:

    curl -sI https://spec.jinn.network/manifest.json

## Certification

Once the domain serves this content, `stable-live-host-verification` in the
mono (runs on every push to `next`) byte-compares the live host against the
same-run attested artifact, verifies each group's manifest signature against
the digest-pinned public key, and probes for host fallback behavior. Its
first green is the precondition for reconsidering the npm stable-publishing
hold.

Consequence to know: the host must serve what the verifying commit built, so
any mono change to the served spec surface requires refreshing this repository
before the gate passes again. Regeneration: download
`platform-verification-artifacts` from the publish run, then

    node .github/scripts/build-profile-host-bundle.mjs \
      --root <artifacts>/sealed-platform-v1/profile-root \
      --root <artifacts>/implementations-v1/profile-root \
      --out <fresh-dir>

and replace this repository's contents with the result. CI-owned refresh is
the intended end state.
