---
title: Intellij IDEA properties文件Unicode转中文
date: 2021-01-23
tags: [IDEA]
categories: IDEA
---



> 今天遇到一个问题，当我使用IDEA打开一个properties文件文件时，里面的中文字符全部都变成了`\u6210\u5668\uFF0C\u914D\u7F6E\u4FE1\u606F`介种奇怪的东西，如下所示

<img src="./pics/idea1.png" align='left' />

### 原因

于是上网搜索，得到答案：默认情况下，在IDEA中打开properties文件中文会显示unicode格式，即``\u6210`这种，因此我们只要修改一下IDEA的配置就好了

### 解决办法

打开设置，直接搜索`File Encodings`，按照下图勾上那个勾然后点击Apply就可以啦！！！

<img src="./pics/idea2.png" align='left' />

