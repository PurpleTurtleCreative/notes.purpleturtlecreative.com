---
title: Useful Commands
has_children: true
nav_order: 1
permalink: /useful-commands/
---

# Useful Commands

Useful commands, tips, and tricks for various parts of the web development workflow and tooling.
{: .text-beta .fw-300 .text-grey-dk-000}

## Sublime: Regex Split Lines by Length

Received a key file that was minified to 3 lines. Normally, key files are split into lines of 64 characters. In Sublime, a simple regex find-replace made it easy to format the file:

```
Find: ([\w\-\+\/\\\=]{64})
Replace: \1\n
```

Capture each group of 64 characters and replace it with the captured group following by a newline.
