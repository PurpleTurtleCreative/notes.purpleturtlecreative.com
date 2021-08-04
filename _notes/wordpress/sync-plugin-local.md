---
title: sync-plugin-local.sh
parent: WordPress
nav_order: 1
---

# Sync a Plugin Repository to Local WordPress

Sync a version-controlled plugin into a local site repo's plugins directory.

```bash
#!/bin/bash

SRC_PLUGIN="/Users/michelleb/Documents/plugins/$1"
DEST=$2

if [[ ! -d "$SRC_PLUGIN" ]]; then
	echo 'Invalid plugin: ' "$SRC_PLUGIN"
	exit 1;
elif [[ "$SRC_PLUGIN" == */ ]]; then
	echo 'Plugin path should not end in slash for rsync: ' "$SRC_PLUGIN"
	exit 1;
fi

if [[ ! -d "$DEST" ]]; then
	echo 'Invalid destination: ' "$DEST"
	exit 1;
elif [[ "$DEST" != */wp-content/plugins ]]; then
	echo 'Destination path should be a plugins folder with no trailing slash: ' "$DEST"
	exit 1;
fi

rsync -avz --exclude=".DS_Store" --exclude=".git*" --delete --delete-excluded "$SRC_PLUGIN" "$DEST"
```