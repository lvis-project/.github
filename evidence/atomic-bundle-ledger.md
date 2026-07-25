# Atomic plugin bundles — fleet ledger

Evidence file for the atomic plugin bundle migration. This is the authority every
downstream gate asserts against, so every gate that reads a value here records this
file's commit SHA and blob digest at the moment it was read.

Every SHA recorded here is a full 40-character object SHA. Abbreviated SHAs satisfy no
gate.

- Recorded: 2026-07-25
- Method: GitHub Contents and Git Refs APIs at the exact refs named in each row. No local
  working checkout was consulted; three of the four canonical checkouts sit on non-PR-head
  branches and the `lvis-app` checkout does not contain the authoritative merged `main` SHA.

## Block 1 — frozen tag-derivable block

Recorded once and immutable thereafter. Changing any recorded value re-freezes the whole
block, retains both values, and invalidates dependent evidence.

| Source repository | Catalog slug | Released tag | Released source SHA | Frozen `main` SHA |
| --- | --- | --- | --- | --- |
| `lvis-project/lvis-plugin-git` | `lvis-plugin-git` | `v0.1.11` | `c49132a1d7b921ce510530254cab0fdedca104db` | `fc72912ec3971940e555bff8d97e682ce3f3a930` |
| `lvis-project/lvis-plugin-local-indexer` | `local-indexer` | `v0.5.28` | `e225b68bfc2fc30ef245845ee2d450b88687dd0b` | `eb2ca45a21ba4290baef3477892d5ef1446da70e` |
| `lvis-project/lvis-plugin-meeting` | `meeting` | `v0.5.36` | `175796e26415d7684a5ca83ca52972b61fe19ad9` | `4801a9553f70eb3787cd7f4a9791229c7978c127` |
| `lvis-project/lvis-plugin-ms-graph` | `ms-graph` | `v0.3.43` | `27a2588c5eb65d26106d1d9a25f6a9aa1fd16673` | `f6d18e00a7c295bf6ff4ea97d3efb28cf382b329` |
| `lvis-project/lvis-plugin-ep` | `ep-api` | `v0.17.32` | `305f394915fcb2d3648b204e1ddad08f63d84544` | `e3b149059ac961bbfe4d72393bf64f38c8de7501` |

The catalog slug is asserted equal to `plugin.json.id` at the recorded source SHA, which is
the same equality the publish pipeline enforces between its `slug` input and the manifest.

| Catalog slug | Released version | `keywords` at tag | `keywords` at frozen `main` | Released `minAppVersion` | `installPolicy` |
| --- | --- | --- | --- | --- | --- |
| `lvis-plugin-git` | 0.1.11 | 15 | 0 | 0.5.2 | `user` |
| `local-indexer` | 0.5.28 | 9 | 0 | 0.5.7 | `admin` |
| `meeting` | 0.5.36 | 5 | 0 | 0.5.7 | `admin` |
| `ms-graph` | 0.3.43 | 9 | 0 | 0.5.2 | `admin` |
| `ep-api` | 0.17.32 | 55 | 0 | 0.5.2 | `admin` |
| **total** | | **93** | **0** | | |

The released total is exactly 93 and the frozen-`main` total is exactly 0. A non-zero
`main` total is itself a drift failure. No row declares any 0.6.0-only contribution key
(`skills`, `hooks`, `mcpServers`) at either ref.

`installPolicy` decides the post-publish approval state: a publish that does not declare
`admin` is approved into the live public catalog immediately, while an `admin` publish lands
in `pending_review` and requires a separate admin approval. Four of five rows are `admin`;
only `lvis-plugin-git` is `user`.

| Catalog slug | Manifest blob digest (`plugin.json` @ tag) | Lockfile | Lockfile blob digest |
| --- | --- | --- | --- |
| `lvis-plugin-git` | `7569efc9218b2e588fcdabf8977bd3d2d2103bb2` | `bun.lock` | `f943caf6c412ec1be68008327a40922b4f6b6159` |
| `local-indexer` | `1f791edea59f898f98cd633123b10795374c34b3` | `bun.lock` | `b72e029352a5f20905e5ee327c6da0f1d0132ab7` |
| `meeting` | `aeffc2cf9ac71e6e63e00470607dc881cb3cf9e1` | `bun.lock` | `0c4141113d78dd9f5e85a27b531b9ee43c27f29b` |
| `ms-graph` | `bcce1ee6ee5d5cd6fee3f098d15de6e5978f7115` | `bun.lock` | `ee2db185a31e3755999c885a3f40bb4ec227aea5` |
| `ep-api` | `4e95b75e0eed2a4799dd19b1ac94a0d5b9bd0325` | `bun.lock` | `93f816852da4e5810bcda0257aa1547bc30e82c3` |

Blob digests are Git object IDs of the file at the recorded source SHA. The released
**archive** digest is deliberately absent from this block: it is not tag-derivable and
belongs to the catalog block below.

### Resolved SDK object SHA

Obtained from each row's own `bun.lock` at that row's recorded source SHA, expanded to 40
characters against the SDK repository. Resolving the pin *string* against the SDK repository
at freeze time does not satisfy this column.

Every pin in this fleet resolves to an **annotated tag object**, not directly to a commit,
so both the object SHA and its peeled commit SHA are recorded. A crossing comparison must
compare like with like.

| Catalog slug | Pin string in `package.json` | `bun.lock` resolution | Resolved SDK object SHA (annotated tag) | Peeled commit SHA |
| --- | --- | --- | --- | --- |
| `lvis-plugin-git` | `github:lvis-project/lvis-plugin-sdk#v8.0.0` | `#45ccbe3` | `45ccbe36eb982c27d6da87d3cf7199e590bbde20` | `d25cb18af4a4ea1cea2fb448536b2bb1432514f1` |
| `local-indexer` | `github:lvis-project/lvis-plugin-sdk#v11.1.0` | `#64e7271` | `64e7271db807bbe22ce5d0b534a66341f3cbefca` | `c5f0d1b22ec7f7fbe5a1c230cf2b8687181cd4b8` |
| `meeting` | `github:lvis-project/lvis-plugin-sdk#v11.1.0` | `#64e7271` | `64e7271db807bbe22ce5d0b534a66341f3cbefca` | `c5f0d1b22ec7f7fbe5a1c230cf2b8687181cd4b8` |
| `ms-graph` | `github:lvis-project/lvis-plugin-sdk#v11.0.0` | `#b6ae7ed` | `b6ae7ede764a57517d0e8fd2be15ef1f5553914e` | `a41c1fa0fd78c93c488116efc9855c3d8a53ab34` |
| `ep-api` | `github:lvis-project/lvis-plugin-sdk#v8.0.0` | `#45ccbe3` | `45ccbe36eb982c27d6da87d3cf7199e590bbde20` | `d25cb18af4a4ea1cea2fb448536b2bb1432514f1` |

Lockfile tarball integrity hashes, recorded as the second equality condition:

| Resolved SDK object | Integrity |
| --- | --- |
| `45ccbe36…` (v8.0.0) | `sha512-67lfqCIAH3gf4TCQqsouIeyo6rQ7wzIv4Ngr+NsrkVwbjRe9T0TGUAIIjwMAuYuuzhuLBkHH4qKeXN/+QB7w9Q==` |
| `64e7271d…` (v11.1.0) | `sha512-Bdc4TpdXA5+nSkk2OXRm1n6cVoZlx+bHqYEDvq81quZUHfNBfmzDfJddlq6vBH5DORAIixZ/Hq/Yz+w+8Fs4vQ==` |
| `b6ae7ede…` (v11.0.0) | `sha512-LiFta14xVj/SV93v95vVWQhgttU7MoYuW7P5uYIAWi+HMucbAkv2CM5yCnb/ocBreo/519eiKIkGH0nUuR6m2A==` |

For reference, the SDK tags this migration also touches:

| SDK tag | Annotated tag object SHA | Peeled commit SHA |
| --- | --- | --- |
| `v8.0.0` | `45ccbe36eb982c27d6da87d3cf7199e590bbde20` | `d25cb18af4a4ea1cea2fb448536b2bb1432514f1` |
| `v11.0.0` | `b6ae7ede764a57517d0e8fd2be15ef1f5553914e` | `a41c1fa0fd78c93c488116efc9855c3d8a53ab34` |
| `v11.1.0` | `64e7271db807bbe22ce5d0b534a66341f3cbefca` | `c5f0d1b22ec7f7fbe5a1c230cf2b8687181cd4b8` |
| `v11.2.0` | `a714e737956b23879ec965e2c9f875435541127d` | `8f4c46236dd046abb4604b6e3c4b3e544a8f044d` |

## Block 2 — catalog block

**Not yet recorded.** This block is appended exactly once, at the moment Marketplace catalog
read access is exercised, and is immutable thereafter. It is a hard prerequisite of bridge
publication.

Columns: catalog-served version, served archive digest, served signature, catalog
`commit_hash`, and per-row `visibility`.

It asserts the catalog `commit_hash` equals the frozen released source SHA in Block 1, that
the served artifact is self-consistent (bytes match the catalog-declared digest and the
signature subject), and that the row set re-derived from the catalog maps one-to-one onto the
caller-derived repository set through the recorded slug column. Any divergence fails the gate.

Blocking prerequisite: `ep-api` is `visibility="internal"` in production. Internal-ness is
decided solely from the `CF-Connecting-IP` header matched against the configured corporate
CIDRs, and no credential bypasses it, so this block's `ep-api` row requires an authorized
internal-network requester. Every absence assertion for `ep-api` must be preceded by a
positive control from that same requester.

## Block 3 — pre-build bridge input block

**Not yet recorded.** Recorded before the resolver accepts any bridge tuple, and immutable
thereafter. The resolver accepts only the tuple recorded here.

Columns: bridge candidate source repository, full 40-character candidate commit SHA, the
frozen `main` SHA it descends from, the full SHAs of its permitted repair commits, lockfile
digest, intended SDK pin and its resolved object SHA, `installPolicy`, and the measured
HostApi-usage difference against the released predecessor.

## Block 4 — append-only bridge output block

**Not yet recorded.** Recorded at bridge publication.

Columns: bridge SDK pin and resolved object SHA, bridge version, exercised-floor Host version,
signed-ZIP digest, signature, admin approval record where `installPolicy` is `admin`, and
publication transaction ID. It asserts equality between the provenance-reported source SHA,
the pre-build recorded candidate SHA, and the reviewed diff's head.

## Derived facts

Recorded here because downstream gates cite them and they must not be re-derived by hand.

- **SDK-crossing set.** Derived by comparing Block 1's resolved SDK object SHA against Block
  4's, never by comparing pin strings. On this measurement `lvis-plugin-git` resolves to
  `45ccbe36…` at both its released tag and its frozen `main`, so it does not cross.
  `ep-api` resolves to `45ccbe36…` (v8.0.0) at its released tag while its frozen `main` pins
  `v11.2.0`, so it is the only crossing row in this fleet.
- **Bridge floors.** Four rows retain their released predecessor's floor — `lvis-plugin-git`
  0.5.2, `local-indexer` 0.5.7, `meeting` 0.5.7, `ms-graph` 0.5.2 — because their SDK pin and
  HostApi usage are unchanged from that predecessor. `ep-api` crosses, so it cannot retain
  0.5.2 on the strength of the schema alone and must declare the floor it actually exercises.

## Private operator record

Named authority holders, cutover machine and profile identities, canary-entry approvers, and
the internal requester's owner, host and network position are **not** recorded in this file,
which is public. They live in the private operator record and are referenced from here by
identifier only.

Referenced identifier: *(to be assigned when the private record is created.)*
