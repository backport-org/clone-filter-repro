# clone-filter-repro

Minimal reproduction for https://github.com/sorenlouv/backport/pull/614.

- `main`: `greeting.txt` was changed by PR #1 (squash-merged)
- `7.x`: branched off before PR #1, so it still has the old `greeting.txt`

Backporting PR #1 to `7.x` with `--clone-filter=blob:none` needs the old `greeting.txt`, which a blobless clone of `main` does not contain and has to fetch on demand.
