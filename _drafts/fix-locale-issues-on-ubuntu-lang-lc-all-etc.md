---
layout: post
title: "Fix locale issues on Ubuntu (LANG, LC_ALL, etc.)"
author: zeroalpha
categories: Guide
---

edit /etc/locale.gen
uncomment the ones you want
then run `sudo locale-gen`
alternatively, do `sudo locale-gen $LOCALE` (e.g. `sudo locale-gen en_US`)

list available locales with `locale -a`
then set the locale with `sudo update-locale LANG=$LOCALE`, which generates /etc/default/locale (sets LC_ALL, LANG, and LANGUAGE)
finally, open a new terminal (or restart the system)
