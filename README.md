# clone-filter-repro

Minimal reproduction for https://github.com/sorenlouv/backport/pull/614: with `--clone-filter blob:none`, backporting fails because backport deletes the git remote that a partial clone fetches missing file contents from.

- `main`: PR #1 changed `greeting.txt` (squash-merged)
- `7.x`: branched off before PR #1, so it still has the old `greeting.txt`, which a blobless clone of `main` does not contain

## Reproduce

From a checkout of https://github.com/sorenlouv/backport:

```sh
git fetch origin pull/614/head
npm ci && npm run codegen

# Without the fix: "✖ Cherry-picking: Update greeting (#1)"
git checkout 9b3c9d1d046c9852232958efb12366b4228d087c
npm start -- --repo backport-org/clone-filter-repro --pr 1 --branch 7.x --clone-filter blob:none --dry-run --dir "$(mktemp -d)"

# With the fix: "✔ Cherry-picking: Update greeting (#1)"
git checkout 3b6c2b58cfec385f6ea3e99fa135f113477a9fa9
npm start -- --repo backport-org/clone-filter-repro --pr 1 --branch 7.x --clone-filter blob:none --dry-run --dir "$(mktemp -d)"
```

Notes:

- `--dry-run` clones and cherry-picks, but skips pushing and creating the pull request, so no write access to this repo is needed.
- `--dir "$(mktemp -d)"` forces a fresh clone on every run. Without it, backport reuses its cached clone in `~/.backport/repositories`, and a clone damaged by the first command stays broken.
- A GitHub token is needed: set `githubToken` in `~/.backport/config.json`, or append `--github-token "$(gh auth token)"`.
- Append `--json` to see the underlying git error: `fatal: unable to read e965047…` (the old `greeting.txt`).
