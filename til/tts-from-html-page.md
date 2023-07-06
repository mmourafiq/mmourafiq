---
layout: til
title: "TTS from HTML page"
cover_image: images/til/speaker.png
cover_image_caption: "speaker"
tags: [ tts, html, linux ]
---

wget -qO- http://paulgraham.com/greatwork.html | sed -e '/<script/,/<\/script>/d' -e 's/<[^>]*>//g; s/\&nbsp\;/ /g; s/\&amp\;/\&/g; s/\&lt\;/</g; s/\&gt\;/>/g' | say --progress -o greatwork
