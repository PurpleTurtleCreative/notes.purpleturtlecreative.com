---
title: File Watcher
parent: Shell Scripting
nav_order: 1
---

# File Watcher

Watch for file modifications and execute commands when changes are noticed.

The following implementation is to sync a WordPress plugin as it is being worked on, which is helpful in a situation where Git submodules are not usable in a deployment pipeline.

```bash
#!/bin/bash

# The path to the destination site's plugins directory.
DEST=$1

# The elapsed seconds to check for changes.
INTERVAL_SECONDS=5

# This script is expected to be located in the plugin's root.
# Your working directory MUST be this plugin's directory when running this script.
SRC=$(pwd)
PLUGIN_DIRNAME='my-great-plugin'

##
## Validation
##

if [[ ! -d "$DEST" ]]; then
	echo 'Invalid destination directory: ' "$DEST"
	exit 1;
elif [[ "$DEST" != */wp-content/plugins ]]; then
	echo 'Destination path should be a wp-content/plugins folder with no trailing slash: ' "$DEST"
	exit 1;
fi

if [[ ! -d "$SRC" ]]; then
	echo 'Invalid plugin source directory: ' "$SRC"
	exit 1;
elif [[ "$SRC" != */"$PLUGIN_DIRNAME" ]]; then
	echo "Script must be called from within the source $PLUGIN_DIRNAME plugin directory."
	echo "Instead found: $SRC"
	exit 1;
elif [[ "$SRC" == */ ]]; then
	echo 'Plugin path should NOT end in slash for rsync: ' "$SRC"
	exit 1;
fi

##
## Functions
##

function sync_plugin() {
	rsync -avz --exclude="*.sh" --exclude="__*.js" --exclude="*/node_modules" --exclude="assets/styles/less" --exclude="package.json" --exclude="package-lock.json" --exclude=".DS_Store" --exclude=".git*" --delete --delete-excluded "$SRC" "$DEST"
}

##
## Processing
##

sync_plugin
echo 'Sync complete. Watching for changes...'

while sleep $INTERVAL_SECONDS; do
	FOUND=$( find "$SRC" -type f -mtime "-${INTERVAL_SECONDS}s" | wc -l | egrep -o '\d+' )
	if [ $FOUND -gt 0 ]; then
		echo "Found $FOUND changed files."
		sync_plugin
		echo 'Sync complete. Watching for changes...'
	fi
done

```

