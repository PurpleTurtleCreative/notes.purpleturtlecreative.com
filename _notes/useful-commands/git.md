---
title: Git
parent: Useful Commands
nav_order: 1
---

# Useful Commands: Git

## Applying Patches
Get the patch file by viewing the Pull Request or commit in GitHub and adding `.patch` to the end:

```
https://github.com/PurpleTurtleCreative/grouped-content/pull/48.patch
```

Right-click the page of raw text and `Save As...` to your computer as a `.patch` file.

Apply the patch file ([source](https://stackoverflow.com/a/15375869)). File paths are relative to your current location, so you should be at the top-level of the repo:
```bash
cd ~/sites/wp-grouped-content
git apply --reject --whitespace=fix /path/to/mychanges.patch
```

Find reject files for review:
```bash
find . -name "*.rej"
```

Automatically open all reject files for review in Sublime Text, Mac OSX:
```bash
find . -name "*.rej" | xargs open -a "Sublime Text"
```

## Remove Commits from Pull Request
Sometimes you'll find your Pull Request on GitHub contains unrelated commits that you don't want polluting your branch's or PR's log. Use interactive rebase to remove the undesired commits and then force push the changes back up to the remote. ([source](https://stackoverflow.com/a/51400593))

```bash
git checkout my-pull-request-branch
git rebase -i HEAD~n
# where 'n' is the number of last commits you want to include in interactive rebase.
# Replace pick with drop for commits you want to discard.
# Save and exit.
git push --force
```

## Automatically Set Remote HEAD ref

If you change the HEAD branch of a remote, you'll likely encounter the error:

```
fatal: ref refs/remotes/origin/HEAD is not a symbolic ref
```

You can fix it automatically with this command:

```bash
git remote set-head origin -a
```

## List Tags with Commit SHAs

Tags are their own objects, which you can see by running:

```bash
git show-ref --tags
```

To know which commits each tag actually points to, dereference them and clean up the output:

```bash
git show-ref --tags -d | grep -F '^{}' | sed -e 's#refs/tags/##' | sed -e 's#\^{}##'
```

If your shell supports extended regex for the `sed` command, then this can be shortened to:

```bash
git show-ref --tags -d | grep -F '^{}' | sed -E 's#refs/tags/(.+)\^\{\}#\1#'
```

