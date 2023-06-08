---
title: Three 入门
---
该片为我自己学习three的过程以及途中遇到的问题，现发出来提供大家参考学习，让大家在学习过程中可以少走弯路

## Three的TS配置
全局安装typings npm install -g typings

安装three.js库 npm install three

安装three.js的typescript语法(.d.ts)的依赖 npm install --save-dev @types/three

src根目录下的shims-vue.d.ts 文件（没有则新建）添加 declare module "@types/three";

> 添加TS语法提示以及代码补全
