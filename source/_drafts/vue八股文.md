---
title: vue八股文
typora-root-url: ./vue八股文
date: 2026-01-17 11:38:06
tags:
categories:
description:vue八股文
---

## 1. vue3的composition API：

1. 更好的代码组织和重用：
   - vue2将不同的功能代码分开写不同的配置对象(数据、方法等)中，组件内容复杂时，不同功能代码散落，难以维护和重用
   - vue3可以将相关功能逻辑组织在一起，提高代码可读性和维护性
2. 更好的逻辑复用
   - vue2逻辑复用使用的mixins和scoped slots（命名冲突，代码可读性差）
   - vue3通过组合函数（可以在多个组件之间共享和复用，避免了mixins的缺点）
3. 更好的typescript的支持（天然支持typescript）
