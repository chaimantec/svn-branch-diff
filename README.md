# svn-branch-diff

Show only the work done on an SVN branch, excluding code that was merged in
from its parent (e.g. trunk "catch-up" merges).

A branch that is kept up to date with sync merges is hard to review with plain
`svn diff`:

- diff vs. the branch point → also shows every trunk change merged in
- diff vs. trunk HEAD → also shows newer trunk changes, reversed

`svn-branch-diff` diffs the branch against its parent *at the revision the
branch was last synced to*. Code that came in through those merges is identical
on both sides and cancels out, leaving only the branch's own changes (plus any
merge conflict resolutions, which are branch work too).

## Requirements

- Python 3.7+
- The `svn` command-line client on `PATH`

## Install

```sh
curl -o ~/bin/svn-branch-diff https://raw.githubusercontent.com/chaimantec/svn-branch-diff/main/svn-branch-diff
chmod +x ~/bin/svn-branch-diff
```

Or clone the repo and put `svn-branch-diff` somewhere on your `PATH`.

## Usage

```sh
svn-branch-diff                                  # working copy in the current dir
svn-branch-diff ^/branches/feature-x
svn-branch-diff https://svn.example.com/repo/branches/feature-x --summarize
svn-branch-diff --log                            # list the branch's own commits
```

Anything after `--` is passed to `svn diff`:

```sh
svn-branch-diff -- -x -w          # ignore whitespace
svn-branch-diff -- --git          # git-style diff
svn-branch-diff -- src/main.c     # limit to a path inside the branch
```

### Options

| Option | Description |
| --- | --- |
| `-r, --revision REV` | Branch revision to inspect (default: HEAD) |
| `--parent URL` | Parent to compare against, as a URL or `^/path` (default: where the branch was copied from) |
| `--base-rev REV` | Parent revision to compare against (default: the last synced revision) |
| `-s, --summarize` | List changed files instead of the full diff |
| `--log` | List the branch's own commits, skipping merge commits |
| `--show-mergeinfo` | Keep `svn:mergeinfo` property changes (hidden by default) |
| `--color {auto,always,never}` | Colorize the diff (default: auto) |
| `-q, --quiet` | Don't print the summary header (on stderr) |

## How the sync point is found

`svn mergeinfo --show-revs eligible PARENT BRANCH` lists the parent revisions
not yet merged into the branch. The sync point is the parent revision just
before the oldest of those, or the parent's latest revision when everything has
been merged. Revisions cherry-picked from the parent past the sync point are
reported, since their changes will appear in the diff.
