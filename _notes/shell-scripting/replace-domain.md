---
title: Replace Domain
parent: Shell Scripting
nav_order: 1
---

# Replace a Website Domain in a File

Replace a website domain in a database dump. This is helpful when pulling a WordPress backup to install locally for development. Be sure to refactor the domains in the replacement file, as well as the file paths in the bash command.

```bash
echo 'Replacing site domain in SQL dump file...'
LC_ALL=C sed -E -i '.old' -f "${SOURCE_PATH}/db-sed.txt" "$DB_SQL"
if [[ -f "${DB_SQL}.old" ]]; then
	echo 'Successfully patched database.'
	rm -f "${DB_SQL}.old"
else
	echo 'FAILED to patch database.'
fi
```

The following implementation replaces all occurences of `https://purpleturtlecreative.com` with `http://localhost:8000`. Update it to suit your needs.

```
s#s:86:"https:\/\/purpleturtlecreative\.com#s:75:"http:\/\/localhost:8000#g
s#https:\\\\\/\\\\\/purpleturtlecreative\.com#http:\\/\\/localhost:8000#g
s#https:\\\/\\\/purpleturtlecreative\.com#http:\/\/localhost:8000#g
s#https:\/\/purpleturtlecreative\.com#http://localhost:8000#g
s#https:\/\/purpleturtlecreative\.com#http:\/\/localhost:8000#g
s#\\"purpleturtlecreative\.com\\"#\\"localhost:8000\\"#g
s#"purpleturtlecreative\.com"#"localhost:8000"#g
s#([^@])purpleturtlecreative\.com#\1localhost:8000#g
```

