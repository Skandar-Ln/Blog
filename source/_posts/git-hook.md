---
title: git hook 为何执行git命令失效
date: 2018-03-06 10:42:52
tags:
---

后发现如果要在hook文件里面执行如git pull之类的命令，必须前面加上env -i git pull才能起作用，这里好像是新开了一个空的环境，但是这个环境为什么里面会有git命令？？