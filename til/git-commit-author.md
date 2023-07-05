---
layout: til
title: "Git commit with new author"
cover_image: images/til/git.png
cover_image_caption: "git"
tags: [ git, ssh ]
---

```bash
git -c user.name="USERNAME" -c user.email=EMAIL@HOST.COM commit --amend --author="USERNAME<EMAIL@HOST.COM>"
```
