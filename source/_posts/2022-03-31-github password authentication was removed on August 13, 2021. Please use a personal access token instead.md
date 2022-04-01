---
layout: default
title: 2022-03-31-password authentication was removed on August 13, 2021. Please use a personal access token instead. 
description: August 13, token
categories: [周边]
tags: [git]
---
# {{ page.title }}

## 1.描述
自 August 13, 2021起，github不再支持用户名、密码的方式操作，需要用token的方式替代。


## 解决办法

### 一、首先生成自己的token、在个人设置页面，找到Setting

![setting](https://img-blog.csdnimg.cn/78e62ce9aae747e4a141bb5ce5d76798.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5YWs5a2Z5YWD5LqM,size_10,color_FFFFFF,t_70,g_se,x_16)

![SSH keys](https://img-blog.csdnimg.cn/bbe3bef2965b4940a3a45e2d5e4b140c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5YWs5a2Z5YWD5LqM,size_20,color_FFFFFF,t_70,g_se,x_16)

![new SSH](https://img-blog.csdnimg.cn/6ba30fb002fa46f9a2674f26a7d4bfff.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5YWs5a2Z5YWD5LqM,size_20,color_FFFFFF,t_70,g_se,x_16)

![new SSH content](https://img-blog.csdnimg.cn/7d96df48a2fa462a94a5803137ffbb57.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5YWs5a2Z5YWD5LqM,size_20,color_FFFFFF,t_70,g_se,x_16)

![new SSH content](https://img-blog.csdnimg.cn/8d315a75e3b04e58bf8b87108958248d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5YWs5a2Z5YWD5LqM,size_20,color_FFFFFF,t_70,g_se,x_16)

![new SSH content](https://img-blog.csdnimg.cn/07029be44b744a0f957d8e34d097e08b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5YWs5a2Z5YWD5LqM,size_20,color_FFFFFF,t_70,g_se,x_16)

注意：

记得把你的token保存下来，因为你再次刷新网页的时候，你已经没有办法看到它了

### 二、token添加远程仓库链接：

把token直接添加远程仓库链接中，这样就可以避免同一个仓库每次提交代码都要输入token了：

```bash
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git
# <your_token>：换成你自己得到的token
#<USERNAME>：是你自己github的用户名
#<REPO>：是你的仓库名称
# 例如：
# git remote set-url origin https://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@github.com/shliang0603/Yolov4_DeepSocial.git/

```