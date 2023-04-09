---
title: Shell Scripting
parent: Useful Commands
nav_order: 1
---

# Useful Commands: Shell Scripting

## Find directories that have a particular plugin
Use in your project directory containing WordPress sites. Writes results to the specified file.
```bash
find . -path "*/wp-content/plugins/wordpress-seo" -print | sed -E -e "s#\.\/([^\/]*)\/.*#\1#" > sites_with_yoastseo.txt
```

## Find all site repositories
Change the `-depth` param to match your project structure.
My hierarchy is `sites/sitename/SiteRepo/.git`, so it outputs all "SiteRepo" dirs when in my "sites" directory.

```bash
find . -depth 3 -name '.git' -execdir pwd \;
```

## Update all site `git` repositories
See [Find all site repositories](#find-all-site-repositories). Define `git` aliases for better, custom results.
Just pull the current branch.

```bash
find . -depth 3 -name '.git' -execdir git pull \;
```
Switch to master, fetch pruned branches, and pull. **Note that this will not work for repositories without a 'master' branch, ie. repos using 'main' as the default.**
```bash
find . -depth 3 -name '.git' -execdir git checkout master \; -execdir git fetch --prune \; -execdir git pull \;
```

## Refactoring Code
Useful when copying code from another project. Using `grep` is _much_ more efficient than using `find`.
```bash
grep -FRl --include='*.php' --exclude='*/node_modules/*' 'Purple_Turtle_Creative' . | xargs sed -i '' -e 's#Purple_Turtle_Creative#PTC_Theme#g'
```

### Refactoring a PHP namespace
Be _super_ careful with your search string by replacing the entire namespace line like `namespace PTC_Theme\Something;` rather than a prefix like `namespace PTC`.

### Refactoring SCSS namespace to function format
Refactor SCSS namespace format `colors.$grey-darker` to function format `color(grey-darker)`.
```bash
grep -FRl --include='*.scss' --exclude='*/node_modules/*' 'colors.$' /path/to/assets/styles/scss | xargs sed -i '' -E 's#colors.\$([a-z\-]*)#color\(\1\)#g'
```

### Refactoring version placeholders before a release

Refactor version placeholder tags in docblocks to prepare for a release.

```bash
grep -FRl --exclude='*/node_modules/*' --exclude='*/vendor/*' '[unreleased]' . | xargs sed -i '' -e 's#\[unreleased\]#3\.4\.0#g'
```

## Removing Lines

Useful when deleting files and needing to remove lines referencing them for inclusion or import. _([source](https://stackoverflow.com/a/5410784))_
```bash
grep -FRl --include='*.scss' --exclude='*/node_modules/*' "@import '../../_tools/scss/mixins';" . | xargs sed -i '' -e "/@import '..\/..\/_tools\/scss\/mixins';/d"
```

## GET HTTP Response Code
Useful for detecting the state of a webpage. For example, a site protected by HTTP Authentication should return `401` (Unauthorized). ([source](https://superuser.com/questions/272265/getting-curl-to-output-http-status-code))

`-L` follow redirects (like http to https, for example)

`-s` silence progress and errors

`-o` direct output

`-w` write to stdout

`--url` url to request

```bash
curl -L -s -o /dev/null -w "%{http_code}" --url https://test.purpleturtlecreative.com/
```