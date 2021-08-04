---
title: update-docker-db.sh
parent: Docker
nav_order: 1
---

# Update Database Docker Container

Pulls a `*.dmp.gz` database backup to replace the current `*.sql` database import file being used by Docker, then removes the current `*_db_data` Docker volume for the site. When the container is started up again, the MySQL image will import the now updated `*.sql` file.

I like to use this with an alias. When I'm getting ready to work on a site, I can use `pwd` like this:
```bash
$ cd /path/to/my/project
$ docker-update-db `pwd`
```

Here's the script. Of course, you'll need to edit it to work with your own project setup and file locations.

```bash
#!/bin/bash

SITE_DIR="$1"
SITE_NAME=$( basename "$SITE_DIR" )
WORDPRESS_BACKUPS_DIR="/Users/michelleb/Dropbox/WordPressBackups"

##
## UPDATE DATABASE
##

SQL_FILE=$( find "$SITE_DIR" -name *.sql )

if [[ ! -f "$SQL_FILE" ]]; then
	echo 'Could not find SQL file in use.'
	exit 1
fi

SQL_BASENAME=$( basename "$SQL_FILE" | cut -d'.' -f1 )

if [[ -z "$SQL_BASENAME" ]]; then
	echo 'Could not determine SQL file base name.'
	exit 1
fi

echo
echo 'Found .SQL file in use: '
echo "$SQL_BASENAME"

SQL_BACKUP=$( find "$WORDPRESS_BACKUPS_DIR" -name "*${SQL_BASENAME}.dmp.gz" )

if [[ -z "$SQL_BACKUP" ]]; then
	echo 'Could not find matching .DMP.GZ backup file.'
	exit 1
fi

echo
echo 'Found matching dmp file:'
echo "$SQL_BACKUP"

cp "$SQL_BACKUP" "$SITE_DIR"

DMP_GZ_FILE="${SITE_DIR}/$( basename "$SQL_BACKUP" )"

if [[ ! -f "$DMP_GZ_FILE" ]]; then
	echo 'Failure. File does not exist: ' "$DMP_GZ_FILE"
	exit 1
fi

echo
echo 'Unzipping archive, please wait...'
gunzip "$DMP_GZ_FILE"

DMP_FILE=${DMP_GZ_FILE%.gz}

if [[ ! -f "$DMP_FILE" ]]; then
	echo 'gzip Failed. File does not exist: ' "$DMP_FILE"
	exit 1
fi

mv "$DMP_FILE" "$SQL_FILE"

echo
echo 'Removing docker volume...'
docker volume rm "${SITE_NAME}_db_data"

echo
echo "Okay! ${SITE_NAME}'s database update is ready to go!"
echo 'Be sure to run [docker stats] to see import progress on next startup.'
echo
```

