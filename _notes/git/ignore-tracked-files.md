---
title: Ignore Tracked Files
parent: Git
nav_order: 1
---

# Git Ignore Tracked File Changes

Sometimes certain files are tracked and committed to a repository for continuous deployments. However, sometimes slight adjustments will want to be made for local development operations that should **never** be committed back to `origin`.

Additionally, some files are automatically updated when performing operations to test a site locally, such as cache files, that should also **never** be committed back up to `origin`.

To ignore changes to certain tracked files, Git can be told to assume certain files are unchanged—which effectively means Git will ignore any changes to these files.

```bash
# git update-index --assume-unchanged <file>...
git update-index --assume-unchanged docker-compose.yml wp-config.php
```

To stop ignoring changes and begin tracking them again, prefix the option with `no`:

```bash
# git update-index --no-assume-unchanged <file>...
git update-index --no-assume-unchanged docker-compose.yml
```

*[Documentation](https://git-scm.com/docs/git-update-index#Documentation/git-update-index.txt---no-assume-unchanged) / [Source](https://stackoverflow.com/a/10755704)*

## List Files Marked as `--assume-unchaged`

```bash
git ls-files -v | grep '^[a-z]'
```

*[Documentation](http://git-scm.com/docs/git-ls-files#Documentation/git-ls-files.txt--v) / [Source](https://stackoverflow.com/a/2363495)*
