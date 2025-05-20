---
layout: post
title: 解决Unity导入像素图模糊
date: 2025-04-12 11:29:09
description: 图片导入优化
tags: Unity
categories: Workflow
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

# 前情提要

想导入像素画进Unity中，进行图片缩放时发现不仅图片模糊，而且有明显的颜色失真和边缘模糊

<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PixelSetting/BlurredPixel.jpg" class="img-fluid rounded z-depth-1" %}
</div>

## 颜色失真

是因为Unity导入图片时默认会压缩，这种压缩算法类似于周围像素取方差，所以RGB值也会修改

- 修改图片Inspector中Compression选择None，也就是不进行压缩

## 边缘模糊

跟Unity采样模式有关系

- 修改图片Inspector中Filter Mode中Point(no filter)

<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PixelSetting/PixelSetting.jpg" class="img-fluid rounded z-depth-1" %}
</div>

## 图片缩放

尽量不用unity中Scale来进行缩放，修改图片Inspector中Pixels Per Unit 比例

- PPU = 图片的像素宽度 / Unity 单位宽度
