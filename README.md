<!--
Copyright (c) 2026 Robert August Vincent II <pillarsdotnet@gmail.com>
Co-author: Claude Code.
-->

# `node-braces` for `ppa:pillarsdotnet/ppa`

This repository holds no `braces` code. It holds the Debian packaging that puts
Ubuntu's own `node-braces`, plus the fix for CVE-2026-93687, into
[`ppa:pillarsdotnet/ppa`](https://launchpad.net/~pillarsdotnet/+archive/ubuntu/ppa).

The fix is micromatch/braces pull request 72 at its final head, with its
three follow-ups, as tagged `v3.0.4-pillarsdotnet.2` on the
`fix-deep-nesting-2` branch of
[`pillarsdotnet/braces`](https://github.com/pillarsdotnet/braces). Upstream
closed the request unmerged on 2026-10-05 and has released no fixed version.
Patterns nested more than 100 levels deep are rejected with a `SyntaxError`
while parsing, and with a `RangeError` when a caller hands `compile`,
`expand` or `stringify` such an AST, instead of overflowing the call stack.

## Layout

| Path            | What it is                                                        |
| --------------- | ----------------------------------------------------------------- |
| `3.0.3/debian/` | Ubuntu's packaging of 3.0.3 (resolute, stonking), plus the patch  |
| `3.0.2/debian/` | Ubuntu's packaging of 3.0.2 (noble), plus the patch rebased to it |
| `bases`         | Which series is built from which directory                        |
| `build-source`  | Builds the unsigned source packages                               |

Each `debian/` is Ubuntu's, unchanged except for a new changelog entry and
`debian/patches/`. The orig tarballs are not kept here: `build-source`
fetches Ubuntu's from Launchpad by version, so the PPA's copies match the
archive's byte for byte.

Noble gets its own directory because it ships 3.0.2, and 3.0.3 needs a
newer `node-fill-range` than noble has. The patch is the same fix rebased:
it keeps 3.0.2's 64 KiB input limit, since lowering that to 10000 was 3.0.3's
own fix (CVE-2024-4068), which noble's `node-braces` still lacks.

## Building

```sh
./build-source ../out             # every series in bases
./build-source ../out noble       # just one
```

Then sign each `_source.changes` with `debsign` and upload it with `dput`.

## Versions

`3.0.3+~3.0.5-1ubuntu0.1~ppa1~ubuntu26.04.1` sorts just above Ubuntu's
`3.0.3+~3.0.5-1`, so `apt` prefers it, and below any update Ubuntu makes
(`-1ubuntu1`, `-1ubuntu0.1`, `-2`), so an official fix replaces it without
anyone having to remove the PPA's copy. The catch is that any Ubuntu update
replaces it, fixed or not; when one appears, check it, and rebase the patch
onto it if it lacks the fix. A new PPA revision bumps `ppa1` in the committed
changelog.
