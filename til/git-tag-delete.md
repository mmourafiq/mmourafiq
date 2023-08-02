---
layout: til
title: "Git tag delete"
cover_image: images/til/git.png
cover_image_caption: "git"
tags: [ git, ssh ]
---

```bash
git push -d github-origin $(git tag -l "v1.4*"); git tag -d  $(git tag -l "v1.4*")
```
