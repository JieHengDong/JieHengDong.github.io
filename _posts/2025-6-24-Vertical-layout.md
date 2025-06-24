---
layout: post
title: Vertical layout
date: 2025-06-24 11:46:20
description: 记录关于垂直布局的知识点
tags: Unity UGUI
categories:
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

# 文字竖版排版

## 前情提要
在需求下，需要文字自上而下从左至右进行排版，类似古文

## 一行或者日文排版
使用TMP_Text空间，支持CSS标签语法，控件自己旋转90°

```CSS
<rotate = -90>文字竖立排版测试</rotate>
```
中文只有一行的情况和日文从右到左的情况是适配的

## 多行文本
用数组的思路进行解决，用CSS类表格的思路也能实现

```C#
// 最大行数
int maxRowCount;
private string VerticalFormatString(string input)
        {
            int columnCount = Mathf.CeilToInt((float)input.Length / maxRowCount);
            char[,] grid = new char[maxRowCount, columnCount];

            // 填充二维数组
            for (int i = 0; i < input.Length; i++)
            {
                int col = i / maxRowCount;
                int row = i % maxRowCount;
                grid[row, col] = input[i];
            }

            // 逐行输出成 TMP 支持的字符串
            System.Text.StringBuilder sb = new System.Text.StringBuilder();
            for (int row = 0; row < maxRowCount; row++)
            {
                for (int col = 0; col < columnCount; col++)
                {
                    if (grid[row, col] != '\0')
                        sb.Append(grid[row, col]);
                    else
                        sb.Append(" "); // 空格补齐
                }
                sb.Append('\n'); // 换行
            }
            return sb.ToString();
        }
```