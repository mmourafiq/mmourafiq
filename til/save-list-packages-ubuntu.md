---
layout: til
title: "Save a list of your favourite packages in Ubuntu."
subtitle: "tip to save a list of all packages that you wish to have every time you run a ubuntu system!"
cover_image: images/blog/ubuntu.png
cover_image_caption: "Ubuntu"
tags: [ linux, ubuntu ]
---


Just reinstalled your ubuntu system, you must be thinking about all your favourite packages, but
it’s really hard to start all over again. Well next time use this tip to save a list of all packages
that you wish to have every time you run a ubuntu system!

```bash
> apt-cache --installed pkgnames
```

Save them in a file:

```bash
>  apt-cache --installed pkgnames > installed.packages.lst
```

To install all packages in a file:

```bash
>  sudo apt-get install `cat installed.packages.lst`
```
