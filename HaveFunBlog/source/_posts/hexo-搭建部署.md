---
title: hexo 搭建部署
date: 2022-11-02 19:40:14
tags:
---
Hexo 操作指南
[Hexo的部署文档](https://hexo.io/zh-cn/docs/)
<!--more-->

# 基础命令

## 新建文章

> hexo new [layout] "title"

新建文章，如果不设置layout会自动选择_config.yml中的default. 如果title中包含空格，要使用**引号**扩充.
- 可以使用-p or --path 来自定义文章的路径，后面加文件路径即可.

## 部署文章

- 生成静态文件：hexo g; 加-d or --deploy 直接部署
- 发布草稿：hexo publish [layout] "filename"
- 启动服务器：hexo server
- 部署文章：hexo d; 加-g or --generate 部署前先生成静态文件
