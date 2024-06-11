---
layout: post
title: 预制体生成UI脚本
date: 2024-04-11 20:14:00
description: 自动化相关
tags: Unity Workflow Automation
categories: 
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

## 前情提要

为了减少重复劳动，根据控件注册UI的简单实现。

依提供思路为主

## 自动化

在右键菜单注册
```c#
[MenuItem("Assets/UIView from Prefab", false, 1)]
```

识别选中物体

```c#
// 获取当前选中的对象
        var prefab = Selection.activeGameObject;
        if (prefab == null)
        {
            Debug.LogError("Please select a prefab.");
            return;
        }
```

使用.Net的库来写入代码，注意引用和缩进

```c#
StringBuilder code = new StringBuilder();
        code.AppendLine("using UnityEngine;");
        code.AppendLine("using UnityEngine.UI;");
        code.AppendLine("using CrocodileFramework.UI;");
        code.AppendLine("using TMPro;");
        code.AppendLine("public class " + className + " : UIWindow");
        code.AppendLine("{");
```

递归识别控件

```c#
private static void AppendChildrenComponents(Transform parent, StringBuilder code, string parentPath)
    {
        foreach (Transform child in parent)
        {
            string path = string.IsNullOrEmpty(parentPath) ? child.name : parentPath + "/" + child.name;
            string fieldType = GetFieldType(child.gameObject);

            if (fieldType != null)
            {
                code.AppendLine($"    private {fieldType} {child.name.Replace(" ", "_")};");
            }

            // 递归处理子对象
            AppendChildrenComponents(child, code, path);
        }
    }
```

根据控件路径写入注册

```c#
private static void AssignChildrenComponents(Transform parent, StringBuilder code, string parentPath)
    {
        foreach (Transform child in parent)
        {
            string path = string.IsNullOrEmpty(parentPath) ? child.name : parentPath + "/" + child.name;
            string fieldType = GetFieldType(child.gameObject);

            if (fieldType != null)
            {
                code.AppendLine($"        {child.name.Replace(" ", "_")} = FindChildComponent<{fieldType}>(\"{path}\");");
            }

            // 递归处理子对象
            AssignChildrenComponents(child, code, path);
        }
    }
```


<mark>部分代码是框架内代码，请自行替换</mark>

## 拓展功能

根据控件类型注册

```c#
private static string GetFieldType(GameObject obj)
    {
        if (obj.GetComponent<Button>() != null) return "Button";
        if (obj.GetComponent<Text>() != null) return "Text";
        // if (obj.GetComponent<Image>() != null) return "Image";
        if (obj.GetComponent<Toggle>() != null) return "Toggle";
        if (obj.GetComponent<Slider>() != null) return "Slider";
        if (obj.GetComponent<Dropdown>() != null) return "Dropdown";
        if (obj.GetComponent<InputField>() != null) return "InputField";
        if (obj.GetComponent<TMP_Text>() != null) return "TMP_Text";
        if (obj.GetComponent<TMP_InputField>() != null) return "TMP_InputField";
        // 添加更多组件类型检查
        return null;
    }
```

识别预制体

```c#
private static bool ValidateGenerateCodeFromPrefab()
    {
        // 验证当前选中的对象是否是预制体
        return Selection.activeObject != null && PrefabUtility.GetPrefabAssetType(Selection.activeObject) != PrefabAssetType.NotAPrefab;
    }
```