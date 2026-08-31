# spec.jinn.network

The public spec surface of the Jinn protocol: every schema, profile, protocol
and facts document that a record cites by identifier. Deployed as a static
site at `https://spec.jinn.network`, with no build step.

**Nothing here is written by hand.** Every file is generated output from
`Jinn-Network/mono`, assembled by that repository's own
`build-profile-host-bundle.mjs`. Edits here would be overwritten and would
break the digests verification depends on. The mono is the only place the
spec is written.

## What is served

| | |
| --- | --- |
| Documents | 797 |
| Built from | `Jinn-Network/mono` @ `dc7ec72c7` |
| Release groups | `sealed-platform-v1` (518), `implementations-v1` (279) |
| Signed | not yet — see below |

Each release group's inventory is served at `<group>/manifest.json`. Nothing
is served at the origin root: per-group namespacing is what lets two groups
share one host (Jinn-Network/mono#3215), and the live-host gate requires a
root `manifest.json` to stay 404.

`vercel.json` sets the content type, entity tag and cache lifetime for every
document and disables URL guessing (`cleanUrls: false`, `trailingSlash:
false`). This is load-bearing: 51 documents have no file extension.

There is no home page. A spec origin serves documents and 404s everything else.

## Deploying

Import into Vercel in the account that owns `jinn.network`, with **no build
command** and the repository root as the output directory, then attach the
domain. Subsequent pushes deploy automatically.

    curl -sI https://spec.jinn.network/schemas/delivery.schema.json
    curl -sI https://spec.jinn.network/task-profiles/prediction-forecast/1.0

Both: `HTTP/2 200`, `content-type: application/json`. And the root inventory
must be absent:

    curl -sI https://spec.jinn.network/manifest.json

## On signing and certification

Manifests here carry no `manifest.dsse.json` sidecar, so
`stable-live-host-verification` will not pass against this content. That is
expected, and it is not a step someone can perform by hand.

The gate compares the live host against the attested artifacts **of its own
run**, and those artifacts expire within a day. A human refresh can therefore
satisfy the gate only by landing inside the window between one run's build and
its verification. Certification is only reachable when CI publishes this
content itself — Jinn-Network/mono#3308. Until that lands, this repository is
refreshed by hand, serves correct bytes, and stays uncertified.

## Regenerating by hand (until #3308)

From a checkout of the mono at the commit to publish:

    node .github/scripts/build-profile-root.mjs \
      --out <roots>/<group> --commit <sha> --release-group <group> --lane stable

for each stack-published group, then

    node .github/scripts/build-profile-host-bundle.mjs \
      --root <roots>/sealed-platform-v1 \
      --root <roots>/implementations-v1 \
      --out <fresh-dir>

and replace this repository's contents with the result.
