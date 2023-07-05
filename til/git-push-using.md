---
layout: til
title: "Push to github using a specific ssh key"
cover_image: images/til/git.png
cover_image_caption: "git"
tags: [git, ssh]
---

```bash
GIT_SSH_COMMAND='ssh -i ~/.ssh/ssh_key_id_rsa -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no' git push origin branch
```
