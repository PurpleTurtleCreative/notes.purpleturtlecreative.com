---
title: bulk-git-patch.sh
parent: Linux
nav_order: 1
---

# Git Patch all Applicable Repositories

Script to apply patch files to applicable repositories via Git. Changes should then be confirmed via GitHub Pull Requests for the pushed branches.

```bash
#!/bin/bash

##
# SETTINGS
##

LOG_FILE='/Users/michelleb/Desktop/bulk-git-patch.log'

##
# USER INPUT
##

echo 'Patch file:'
read PATCH_FILE

if [[ -z "$PATCH_FILE" ]]; then
	echo 'ERROR: No patch file specified.'
	exit 1
elif [[ ! -f "$PATCH_FILE" ]]; then
	echo 'ERROR: That patch file does not exist.'
	exit 1
fi

echo 'Branch name (ie. fix/12345):'
read BRANCH_NAME

if [[ -z "$BRANCH_NAME" ]]; then
	echo 'ERROR: You must specify a branch name on which to safely apply the patch.'
	exit 1
fi

echo 'Commit message:'
read COMMIT_MESSAGE

if [[ -z "$COMMIT_MESSAGE" ]]; then
	echo 'ERROR: You must provide a commit message for applying the patch.'
	exit 1
fi

echo
echo '---------------------------------------'
echo 'Please review and reply YES to confirm.'
echo
echo 'To watch progress, follow the log:'
echo "    tail -f $LOG_FILE"
echo '---------------------------------------'
echo 'Patch file:' "$PATCH_FILE"
echo 'Branch name:' "$BRANCH_NAME"
echo 'Commit message:' "$COMMIT_MESSAGE"
echo '---------------------------------------'
read APPROVED

if [[ 'YES' != "$APPROVED" ]]; then
	echo 'Okay. Patch was not applied.'
	exit 1
fi

##
# HELPERS
##

git-default-branch() {
	git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
}

git-update() {
	git checkout `git-default-branch` && git fetch --prune && git pull && git restore wp-config-sample.php
}

##
# PROCESSING
##

echo
echo 'Applying patch to applicable site repositories...'
echo

START_DATE=`date`

for SITE_DIR in /Users/michelleb/Documents/sites/*; do
	cd "$SITE_DIR"

	# Go into repo dir, which is assumed to be the only immediate child directory.
	## TODO: Get dir path where .git database is located. THAT is a Git repo.
	cd `find . -maxdepth 1 -type d ! -path . | sed 1q`

	{
		# Log section header.
		echo '======================================='
		echo 'Working directory:' `pwd`
		echo '---------------------------------------'
		# Checkout default branch and ensure it's current.
		git-update
	} &> "$LOG_FILE"

	# Quietly check if patch applies.
	git apply --check "$PATCH_FILE" &> /dev/null
	if [[ $? != 0 ]]; then
		echo 'Patch does not apply:' `pwd`
		continue
	fi

	# Perform patch.
	echo "Patching" `basename "$SITE_DIR"`

	{
		# Create branch for patch.
		git checkout -b "$BRANCH_NAME"
		# Apply patch.
		git apply --whitespace=fix "$PATCH_FILE"
		# Commit changes.
		git add .
		git commit -m "$COMMIT_MESSAGE"
		# Push branch to origin.
		git push --set-upstream origin "$BRANCH_NAME"
	} &> "$LOG_FILE"

done

echo
echo 'Done! Patch has been committed where applicable.'
echo
echo 'Start time:' "$START_DATE"
echo 'End time:  ' `date`

```

