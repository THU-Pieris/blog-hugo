---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
image: 
math: 
license: 
comments: true
slug: {{ substr (md5 (printf "%s%s" .Date (replace .TranslationBaseName "-" " " | title))) 4 8 }}
categories:
  - 
tags:
  - 
description: 
---

> 本文在[Craft](https://www.craft.do)上撰写并更新。可访问以下链接：
>  
> - 原文: <>
> - PDF: [GitHub]()
