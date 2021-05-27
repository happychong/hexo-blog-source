---
layout: default
title: git bash arrow 不能选择解决方案(windows)
description: git arrow 选择
categories: [周边]
tags: [git]
---
# {{ page.title }}

## 1.情况说明

创建cue-cli项目的时候，git bash 窗口中会出现选择项目，windows中某些情况下不支持上下箭头选择

## 2.解决方案

使用winpty,然后上下箭头就可以进行操作了

```bash
# vue create 16.vue-cli-demo 改成如下
winpty vue.cmd create 16.vue-cli-demo
```
