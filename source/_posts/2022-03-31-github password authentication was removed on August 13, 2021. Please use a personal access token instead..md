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

### 一、git config --global --list 验证邮箱与GitHub注册时输入的是否一致

```bash
$ git config --global --list
# user.email=14392***@qq.com
# user.name=14392***@qq.com
# http.sslverify=false
```

### 设置全局用户名和邮箱

```bash
$ git config --global user.name "username"
$ git config --global user.email "14392***@qq.com"
```

### 三、ssh-keygen -t rsa -C “这里换上你的邮箱”，一路回车，在出现选择时输入Y，再一路回车直到生成密钥。会在/Users/***/路径下生成一个.ssh文件夹，密钥就存储在其中

```bash
$ ssh-keygen -t rsa -C "11**********@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa.
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:JtqdeZ9RwhvDEFyQaZa6zTV0OTpD9rNYbWNJ4sNo8r0 11***********@qq.com
The key's randomart image is:
+---[RSA 3072]----+
|         .o*.  . |
|          B.+.+. |
|         +.+++1o.|
|        .. **+++o|
|      . S++.V*++.|
|     o +.1o..H.  |
|    . . + . 1 .  |
|         . . T   |
|            o    |
+----[SHA256]-----+
```

### 四、复制 /c/Users/Lenovo/.ssh/id_rsa.pub 里面的内容到到git仓库，添加秘钥

![setting](https://s3.51cto.com/images/blog/202110/27170243_617915b341b7746185.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

![SSH keys](https://s6.51cto.com/images/blog/202110/27170243_617915b37ffb632672.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

![new SSH](https://s5.51cto.com/images/blog/202110/27170243_617915b3ba8f123736.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

![new SSH content](https://s3.51cto.com/images/blog/202110/27170244_617915b41a3b740771.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

### 五、ssh -T  git@github.com 测试一下通不通，通了显示如下：

```bash
$ ssh -T git@github.com
Hi aaro***! You've successfully authenticated, but GitHub does not provide shell access.
```

现在可以正常使用了。

如果不通如下两步操作即可：

```bash
$ ssh-agent -s
$ ssh-add ~/.ssh/id_rsa
```