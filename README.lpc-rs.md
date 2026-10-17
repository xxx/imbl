# imbl owned-branch disposal extension

This fork starts from upstream imbl v7.0.2 at
834c67f1e08c63a4d662a4132425026a4cea7b25. Its Rust source and original
manifest match the published 7.0.2 crate. A separate commit corrects the
upstream lockfile's stale self-version from 7.1.0 to 7.0.2, matching that crate.

The feature commit adds GenericHashMap::into_drop_chunks and its opaque
DropChunk result in src/hash/map.rs and src/nodes/hamt.rs. A uniquely owned root
transfers its entries without walking descendants or cloning keys/values.
A shared root remains one chunk. Dropping chunks releases only their ownership;
other snapshots keep shared nodes alive. Existing map layout and operations are
unchanged. The extension adds no unsafe code, runtime dependencies or Rayon policy.
Original authorship, source headers and MPL licensing are retained.

Ownership tests in tests/drop_chunks.rs cover empty and unique roots, shared
roots and descendants, collisions, exact destruction on two threads and local
values that cannot be sent to another thread. Run:

```sh
cargo test --locked --test drop_chunks
```

lpc-rs depends directly on this fork using an exact Git revision and commits its
lockfile. Its workspace contract tests also run these ownership checks when the
dependency is updated. Native worker integration is maintained in lpc-rs.

To update against upstream:

1. Fetch the intended upstream release and create a new branch from its reviewed
   tag or commit. Keep previous published commits available for pinned consumers.
2. Cherry-pick the feature commit onto the new base. Review representation and
   ownership changes before resolving conflicts; do not assume a clean textual
   application proves compatibility. Apply any release lockfile correction as
   a separate commit only if still needed.
3. Run the ownership tests and upstream checks, then run lpc-rs workspace tests,
   formatting, Clippy and documentation checks with the proposed revision.
4. Advance the exact lpc-rs revision only after those checks pass.
5. If upstream provides equivalent ownership-preserving branch disposal, switch
   lpc-rs back to a reviewed registry release and remove the custom dependency.

Upstream: https://github.com/jneem/imbl
