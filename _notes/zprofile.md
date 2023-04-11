---
title: .zprofile
has_children: false
nav_order: 0
---

# ZSH Profile

```bash
# PATH
export PATH="/opt/homebrew/opt/ruby@2.7/bin:$PATH"

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

phpcs-test-compat() {
	# $1 - PHP version to check compatibility, default 8.0
	# $2 - file or directory to scan, default current directory

	PHP_VERSION="$1"
	if [[ -z "$PHP_VERSION" ]]; then
		PHP_VERSION="8.0"
	fi

	SOURCE="$2"
	if [[ -z "$SOURCE" ]]; then
		SOURCE="."
	fi

	phpcs -p --ignore="*/vendor/*" --extensions=php --standard=PHPCompatibilityWP --runtime-set testVersion "$PHP_VERSION" "$SOURCE"
}

alias sublime='open -a "Sublime Text"'

# Purple Turtle Creative
alias ptc-pull-backups='bash /Users/michelle/web/scripts/purple-turtle-creative/pull-backups.sh'
alias ptc-update-staging='bash /Users/michelle/web/scripts/purple-turtle-creative/update-staging.sh'

ptc-push-resource() {
	# $1 - source file to upload
	# $2 - resources directory destination (ie. plugins/completionist)

	SOURCE="$1"
	if [[ -z "$SOURCE" ]]; then
		echo 'Missing argument 1: source file to upload'
		return 1
	fi

	SOURCE_BASENAME=$(basename "$SOURCE")

	# Remove all trailing slashes.
	RESOURCE_DEST=$(echo "$2" | sed 's#/*$##')
	if [[ -z "$RESOURCE_DEST" ]]; then
		echo 'Missing argument 2: resources directory destination (ie. plugins/completionist)'
		return 2
	fi

	if [[ $(echo "$RESOURCE_DEST" | sed -E 's#^([^/]+).*#\1#') != 'plugins' ]]; then
		echo 'Invalid argument 2: resources directory destination must begin with "plugins" since that is the only supported resources type at this time'
		return 3
	fi

	RESOURCE_DEST_ABS="/var/www/html/wp-content/plugins/ptc-resources-server/resources/$RESOURCE_DEST/$SOURCE_BASENAME"

	echo "SOURCE            = $SOURCE"
	echo "SOURCE_BASENAME   = $SOURCE_BASENAME"
	echo "RESOURCE_DEST     = $RESOURCE_DEST"
	echo "RESOURCE_DEST_ABS = $RESOURCE_DEST_ABS"

	scp "$SOURCE" ptc:"$RESOURCE_DEST_ABS"
	ssh ptc "chmod 775 \"$RESOURCE_DEST_ABS\"; chgrp www-data \"$RESOURCE_DEST_ABS\""
}

alias rsync-plugin='rsync --archive --delete --progress --exclude="*/.git*" --exclude=".gitignore" --exclude=".DS_Store" --exclude="*.log" --exclude="*/node_modules/*"'

# Docker
alias docker-stop-all='docker stop $(docker ps -q)'
alias docker-remove-all='docker rm $(docker ps -aq)'
alias docker-forget-all='docker volume rm $(docker volume ls -q)'
alias docker-shutdown='docker-stop-all && docker-remove-all && docker network prune -f'
alias docker-this-container='pwd | xargs basename | xargs -I% docker ps --filter name=%'
alias docker-forget-this='pwd | xargs basename | xargs -I% docker volume rm %_db_data'
# this container
alias docker-this-container='pwd | xargs basename | xargs -I% docker ps --filter name=%'
alias docker-this-ports='docker-this-container --format "table {{.Names}}\t{{.Ports}}"'
alias docker-this-urls='docker-this-ports | grep -Eo "0\.0\.0\.0:[0-9]{4}->80\/tcp" | sed -e "s#0.0.0.0:#http://localhost:#g" -e "s#->80/tcp#/#g"'
alias docker-forget-this='pwd | xargs basename | xargs -I% docker volume rm %_db_data'
##### Currently doesn't work: the input device is not a TTY
# alias docker-login='pwd | xargs basename | xargs -I% docker exec -it %_wordpress_1 bash'
#####
alias list-site-ports='grep -rio --include docker-compose.yml "\-\s*8\d\d\d:80" /Users/michelle/web/docker'
alias list-claimed-site-ports='grep -rioh --include docker-compose.yml "\-\s*8\d\d\d:80" /Users/michelle/web/docker | sort'

# Git
alias git-default-branch="git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'"
alias git-prune-local="git branch -vv | grep ': gone]' | awk '{print \$1}' | xargs git branch -d"
alias git-update='git checkout $(git-default-branch) && git fetch --prune && git pull && git-prune-local'

# Files
alias prepare-package="zip -rT9 --exclude='*/.git*' --exclude='*/.DS_Store' --exclude='*.log'"
alias grep-wp="grep -FRin --exclude='*/.git*' --exclude='*/node_modules/*' --exclude='*/vendor/*'"

# Tools
alias mailinator='echo;echo michelleb.$(date +%s)@mailinator.com;echo;'
alias sublime='open -a "Sublime Text"'

###############################
### Workflow Processes
###############################

sed-replace-from-file() {
	# $1 - file to search/replace
	# $2 - file containing search/replace sed rules

	echo 'Replacing file contents...'
	LC_ALL=C sed -E -i '.old' -f "$2" "$1"
	if [[ -f "${1}.old" ]]; then
		echo 'Successfully updated the file!'
		rm -f "${1}.old"
	else
		echo 'FAILED to update the file.'
	fi
}

docker-switch() {
	# $1 - site dir name to switch to

	if [[ -z "$1" ]]; then
		echo 'You must specify a site. Here are all available sites:'
		find -s /Users/michelle/web/docker -maxdepth 1 -type d ! -path /Users/michelle/web/docker -exec basename {} \;
		return 1
	elif [[ ! -d /Users/michelle/web/docker/"$1" ]]; then
		echo 'That site does not exist.'
		echo 'Run [docker-switch] with no args to list the available sites.'
		return 1
	fi

	SITE_DIR=/Users/michelle/web/docker/"$1"
	cd "$SITE_DIR"
	docker-shutdown

	# Update Site
	REPO_DIR=`pwd`/`find . -maxdepth 1 -type d ! -path . | sed 1q | xargs basename`
	cd "$REPO_DIR" && git-update
	# Open in Finder
	open "$REPO_DIR"/wp-content

	# Launch Site
	cd "$SITE_DIR"
	docker compose up -d
	# Open Incognito
	open -na "Google Chrome" --args -incognito `docker-this-urls | grep -F 'http://localhost:8'`

	# End in Repo Directory
	cd "$REPO_DIR"
	git restore wp-config-sample.php
}

eval "$(/opt/homebrew/bin/brew shellenv)"

```

## Installing Binaries

When on my Intel-based MacBook Pro, I simply created command aliases to enable binaries that I'd install from GitHub such as PHPCS.

Now, I'm using a better approach by simply creating symbolic links of the installed binaries in the typical `$PATH` directory like `/usr/local/bin`. This is useful because I can still update and version control these binaries with `git` which adds a lot of extra files and directories. Additionally, I can organize those repositories wherever I'd like while the symbolic link adds the executable binaries to my standard `$PATH` to keep command name lookups quick and standard.

For example, you do this like so:

```bash
ln -s /Users/michelle/web/bin/phpcs/bin/phpcs /usr/local/bin
```

**For binaries that have a complicated build**, make, configuration, versioning, or other installation process, I like to instead use [Homebrew](https://brew.sh/), a package manager for MacOS. PHP is the primary example.
