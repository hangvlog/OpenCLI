# Local 1.8.7 owned-tab cleanup backport

This branch preserves the 1.8.7 CLI and adapter contracts while backporting the
Page.closeWindow portion of upstream commit
221d1d3d5ad4e3e70abe1ff61c6e954a6b595f47 (PR #2637).

Cleanup defaults to `close-owned-tab`. Set `OPENCLI_CLOSE_OWNED_TAB=0` only when
reusable blank placeholders are desired. Browser Bridge 1.0.26 acknowledges
`closed-owned-tab` or `no-owned-tab`; older extensions reject the action without
releasing the lease. Failed cleanup retains the page identity and must be
retried separately, never by repeating a site write.

This changes explicit browser close and adapter closeWindow cleanup. It does
not remove every about:blank tab, alter borrowed user tabs, or change the
extension's independent idle-expiration behavior.

## Build without dependency installation

The existing runtime dependency versions are recorded in local-runtime-lock.json.
Reuse that exact dependency tree; do not run an unpinned install. The local
build uses Node 22.22.0, TypeScript 5.9.3 and tsx 4.21.0:

1. `node scripts/clean-dist.cjs`
2. `node scripts/copy-yaml.cjs`
3. `tsc --build --noCheck`
4. `tsx src/build-manifest.ts`

`--noCheck` performs compilation only; validation is the targeted cleanup
behavior checks plus real browser open/close and borrowed-tab checks. This is
not a full typecheck or all-site regression claim.

Deploy the generated package from the committed source as a separate immutable
local runtime and retain the previous CLI path for rollback. Never modify the
installed package's dist files in place.
