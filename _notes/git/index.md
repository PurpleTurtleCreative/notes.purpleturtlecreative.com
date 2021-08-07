---
title: Git
has_children: true
nav_order: 1
permalink: /git/
---

# Git

Helpful scripts and tips to aid in version control. Might also include help for working with GitHub.
{: .text-beta .fw-300 .text-grey-dk-000}

---

Quickly create a global `.gitignore` for your local environment:

```bash
echo '.DS_Store' > ~/.gitignore
git config --global core.excludesfile ~/.gitignore
```

