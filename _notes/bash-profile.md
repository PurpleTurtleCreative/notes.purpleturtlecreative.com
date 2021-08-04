---
title: .bash_profile
has_children: false
nav_order: 0
---

# Personal Web Development Setup

```bash
# Binaries
alias sass='/usr/local/bin/dart-sass/sass'
alias phpcs='/usr/local/bin/phpcs/bin/phpcs'
alias phpcbf='/usr/local/bin/phpcs/bin/phpcbf'

# Docker
# global
alias docker-update-db='bash /Users/michelleb/scripts/docker-update-db.sh'
alias docker-stop-all='docker stop $(docker ps -q)'
alias docker-remove-all='docker rm $(docker ps -aq)'
alias docker-forget-all='docker volume rm $(docker volume ls -q)'
alias docker-shutdown='docker-stop-all && docker-remove-all && docker network prune -f'
# this container
alias docker-this-container='pwd | xargs basename | xargs -I% docker ps --filter name=%'
alias docker-this-ports='docker-this-container --format "table {% raw %}{{.Names}}\t{{.Ports}}{% endraw %}"'
alias docker-this-urls='docker-this-ports | grep -Eo "0\.0\.0\.0:[0-9]{4}->80\/tcp" | sed -e "s#0.0.0.0:#http://localhost:#g" -e "s#->80/tcp#/#g"'
alias docker-forget-this='pwd | xargs basename | xargs -I% docker volume rm %_db_data'
# global info
alias list-site-ports='grep -rioH --include docker-compose.yml "\-\s*8\d\d\d:80" /Users/michelleb/Documents/sites | sed -e "s#/Users/michelleb/Documents/sites/##g" -e "s#/docker-compose.yml:# #g"'
alias list-claimed-site-ports='grep -rioh --include docker-compose.yml "\-\s*8\d\d\d:80" /Users/michelleb/Documents/sites | sort'
##### Currently doesn't work: Error - the input device is not a TTY
# alias docker-login='pwd | xargs basename | xargs -I% docker exec -it %_wordpress_1 bash'
#####

# Git
alias git-default-branch="git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'"
alias git-prune-local="git branch -vv | grep ': gone]' | awk '{print \$1}' | xargs git branch -d"
alias git-update='git checkout $(git-default-branch) && git fetch --prune && git pull && git-prune-local && git restore wp-config-sample.php'

# General
alias grep-wp="grep -FRlin --exclude='*/.git*' --exclude='*/node_modules/*' --exclude='*/bower_components/*' --exclude='*/vendor/*' --exclude='*.log'"

# Tools
alias mailinator='echo;echo michelleb.$(date +%s)@mailinator.com;echo;'

###############################
### Workflow Processes
###############################

docker-switch() {
	# $1 - site dir name to switch to

	if [[ -z "$1" ]]; then
		echo 'You must specify a site. Here are all available sites:'
		find -s /Users/michelleb/Documents/sites -maxdepth 1 -type d ! -path /Users/michelleb/Documents/sites -exec basename {} \;
		return 1
	elif [[ ! -d /Users/michelleb/Documents/sites/"$1" ]]; then
		echo 'That site does not exist.'
		echo 'Run [docker-switch] with no args to list the available sites.'
		return 1
	fi

	SITE_DIR=/Users/michelleb/Documents/sites/"$1"
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
```

