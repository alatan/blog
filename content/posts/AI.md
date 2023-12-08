---  
title: "AI 应用总结"  
date: 2023-02-01
weight: 70  
draft: false  
keywords: ["AI"]  
description: "AI 讨论总结"  
tags: [ "AI"]  
categories: ["AI"]  
author: "默哥"  

lightgallery: true
toc:
  auto: false
---  

## 行业
### 外贸
* 外贸商品的各场景图片生成
> 比如一个U盘在各种电脑上面的图片

### 服装行业
* 电商服装行业画重点：打板、爆品复刻、反推拆解图
他说的是把图案直接生成衣服样子的图 裁剪图，单块的，把每一块拼起来就是衣服

> 打板行业痛点，时间长，成本高，有个衣服设计想法出来单个需要 5 千的成本
假模特也可以变真人模特，前视图，左，右，后试图，pose，CNet，lora，线稿
##### 可落地尝试
* 创意服装二维码

### 照相馆
* 摄影
* 手绘效果
* 模特换脸    

### 餐饮品牌营销公司
* 生成菜品的宣传图
> 菜品模型没有标品，肯定是要调教的

### 室内设计
毛胚房变装修效果图，室内设计应用场景

### 视频宣传片
* 安全教育宣传片（文案，卡通人物，生成视频）

## 实现
### 工具
#### AI绘画
* https://www.openflowai.net/
* Stable Diffusion(SD)
* Mid Journey（MJ）
* controlnet 线稿

##### AI视频
* runway pika 生成视频(chatgpt做脚本)
* sd出图 需要选checkpoint lora embedding 然后结合 controlnet 可以快速出替换
* gpts的应用市场里，有直接文本生提示词的，还是分好镜的
* 你去找“小说文本提示词大师”这类的应用就好
* gpts有hunt可以搜搜    

#### 方式论
* 关键词生产
* 可以找一些参考对象
* 生成后 ps 再优化下