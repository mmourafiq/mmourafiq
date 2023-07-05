---
layout: til
title: "Git tag update"
cover_image: images/til/git.png
cover_image_caption: "git"
tags: [ git, ssh ]
---

```bash
git tag -f -a 0.4.3
git push -f origin 0.4.3
```

```bash
git fetch --tags origin 0.5.6
git push origin 0.5.6
```
