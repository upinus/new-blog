# Upinus Tech Blog

## Developement
```bash
hugo serve -D
```

## Update theme
```
git submodule update --remote themes/kiss
```
or
```
cd ./theme/kiss
git pull
```

## Example blog post template
```
---
title: This is an example blog
author: John Doe
date: (must include timezone) 2010-06-06 00:00:00+07:00
summary: (optional, but nice to have) This article will show you how to write a simple blog post
tags: (help readers filter their related posts) ['foo', 'bar']
---

## Section 1
Your content here

## Section 2
Your content here
```
