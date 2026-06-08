---
title: 黑马uniapp小兔鲜
typora-root-url: ./黑马uniapp小兔鲜
tags:
categories:
description: 
date:
---
## 在线笔记：https://megasu.atomgit.net/uni-app-shop-note/
## 视频学习：https://www.bilibili.com/video/BV1Bp4y1379L/

1. 小程序是**多页面应用**，页面样式自动隔离， \<style\> 页面样式不需要写 scoped
2. 生命周期分三部分：应用生命周期(小程序)，页面生命周期(小程序)，组件生命周期(Vue)
3. 微信小程序规定屏幕宽度固定为 750rpx(虚拟的响应式单位，会自动根据设备宽度缩放)
4. 用命令行创建项目配置
   1. vscode插件安装：uni-create-view，uni-helper，uniapp 小程序扩展
   2. 配置ts类型校验：pnpm i -D miniprogram-api-typings@latest @uni-helper/uni-app-types@latest（配置tsconfig.json文件）
   3. 处理json注释问题：设置文件关联，把 manifest.json 和 pages.json 设置为 jsonc
5. 代码统一风格+git提交操作配置：
   1. .editorconfig，.prettierrc.json，eslint.config.mjs文件配置
   2. .vscode/extensions.json, .vscode/settting.json文件配置
   3. husky配置
6. 