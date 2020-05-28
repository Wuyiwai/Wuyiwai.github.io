---
title: JetBrains Mono字体安装
date: 2020-05-28 16:52:22
updateDate: 2020-05-28 16:52:22
tags: IDE
categories: IDE
---

> 概要: JetBrains Mono字体在windows下体验较好.此文将介绍配置过程

<!--more-->

### [安装](https://www.jetbrains.com/lp/mono/#how-to-install)
1. In JetBrains IDEs (version > v2019.3.)
    1. Preferences/Settings → Editor → Font, and then select JetBrains Mono from the Font dropdown.
2. Another IDE or an older version of a JetBrains IDE
    1. [Download font](https://download.jetbrains.com/fonts/JetBrainsMono-1.0.3.zip?_ga=2.54527723.1348071829.1590655334-1221605599.1575428455)
    2. Unzip the archive and install the font:
        - MAC: Select all font files in the folder and double-click them. Click the “Install Font” button.

        - WIN: Select all font files in the folder, right-click any of them, then pick “Install” from the menu.

        - LINUX: Unpack fonts to ~/.local/share/fonts (or /usr/share/fonts, to install fonts system-wide) and execute fc-cache -f -v
    3. Restart your IDE.
    4. Go to Preferences/Settings → Editor → Font, and pick JetBrains Mono from the Font dropdown.

### 推荐配置
1. Size: 13
2. Line spacing: 1.2