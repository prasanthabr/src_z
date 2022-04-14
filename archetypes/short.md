+++
title = "{{ replace .TranslationBaseName "-" " " | title }}"
date = {{ .Date }}
description = "shortcuts to improve efficencies"
draft = true
toc = false
categories = ["shortcuts"]
tags = ["after", "dark"]
images = [
"https://source.unsplash.com/collection/983219/1600x900"
] # overrides site-wide open graph image
[[copyright]]
owner = "{{ .Site.Params.author | default .Site.Title }}"
date = "{{ now.Format "2006" }}"
license = "cc-by-sa-4.0"
+++
