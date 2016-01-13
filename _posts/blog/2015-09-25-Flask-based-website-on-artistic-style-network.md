---
layout:     post
title:      Artistic style network implementation with Flask on web
category: blog
description: Flask, deep learning
---


## 简介

想法来源于下篇论文，即是最近微博上火起来的图像融合算法，利用深度学习算法将一幅图片改变成另一张图片的风格。算法已经在Github开源，使用脚本方法迭代运算，最终产生结果图片。

我的目标是将其转为网站，可以动态显示图片迭代的过程。

References: A Neural Algorithm of Artistic Style; Leon A. Gatys, Alexander S. Ecker, Matthias Bethge; arXiv:1508.06576; 08/2015

## 架构

web应用服务器Flask + 容器fastcgi + 反向代理nginx + 缓存redis(memcached 弃用) + supervisord

>Flask负责逻辑处理，页面呈现，实现功能逻辑
>fastcgi负责启动flask应用
>nginx做反向代理，处理web请求
>redis存储状态任务处理状态
>supervisord管理监控web、nginx和缓存redis

利用session实现多个用户可以同时执行图片处理任务，每个用户同时间只能执行一个任务。

## 实现

### 前端与后端
### 上传

### session控制任务
### 缓存机制
### supervisor
### 参数处理
### 多线程多进程
### 状态查询
### 终止任务

## 效果展示



[Python包管理工具解惑(pip ez_install egg whl...)][2]

## PASS

back soon

## TL;DR

**生命在于折腾**


[1]:    {{ page.url}}  ({{ page.title }})
[2]:	http://zengrong.net/post/2169.htm   "Python 包管理工具解惑" 
[3]:	http://www.oschina.net/translate/the-flask-mega-tutorial-part-i-hello-world
[4]:	http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world
[5]:    http://liuzhijun.iteye.com/blog/1872241