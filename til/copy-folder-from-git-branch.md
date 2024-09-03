---
layout: til
title: "Copy Folder from one Git Branch to Another"
tags: [ git ]
---

```sh
$ git checkout work
Switched to branch 'work'
$ git checkout master -- utils
$ git add utils
$ git commit -m "Adding 'utils' directory from 'master' branch."
```
