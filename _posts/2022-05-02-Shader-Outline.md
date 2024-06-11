---
layout: post
title: Outline Shader/ 2D图片描边
date: 2022-05-02 14:42:53
description: 实现描边的一些坑
tags: Shader
categories: 
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

## 前情提要
    之前公司业务上有成千上万的图片需要后期添加描边，让美术重新制作描边效果图片，工单太多啦不现实。随即使用Shader实现效果

## 边缘检测描边

```glsl
Shader "UICustom/Outline"
{
    Properties
    {
        _MainTex("Texture", 2D) = "white" {}
        _lineWidth("lineWidth",Range(0,10)) = 1
        _lineColor("lineColor", Color) = (1,1,1,1)
    }
    SubShader
    {
        Tags { "RenderType"="Transparent" }
        Blend SrcAlpha OneMinusSrcAlpha

        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include  "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct VertexInput
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct VertexOutput
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            VertexOutput vert (VertexInput v)
            {
                VertexOutput o;
                o.vertex = TransformObjectToHClip(v.vertex);
                o.uv = v.uv;
                return o;
            }

            sampler2D _MainTex;
            float4 _MainTex_TexelSize;
            float _lineWidth;
            float4 _lineColor;

            float4 frag(VertexOutput i) : SV_Target{
                float4 col = tex2D(_MainTex,i.uv);

                // 采样周围四个点
                float2 up = i.uv + float2(0,1) * _lineWidth * _MainTex_TexelSize.xy;
                float2 down = i.uv + float2(0,-1) * _lineWidth * _MainTex_TexelSize.xy;
                float2 left = i.uv + float2(-1, 0) * _lineWidth * _MainTex_TexelSize.xy;
                float2 right = i.uv + float2(1,0) * _lineWidth * _MainTex_TexelSize.xy;

                // 边缘检测思路 透明度为零就是边缘
                float w = tex2D(_MainTex, up).a * tex2D(_MainTex, down).a * tex2D(_MainTex, left).a * tex2D(_MainTex, right).a;

                col.rgb = lerp(_lineColor, col.rgb, w);
                return col;
            }
            ENDHLSL
        }
    }
}

```

> 边缘识别的算法也可以尝试用半圆范围识别，这里只是提供简单思路

## 新的问题

    由于UI图大量的透明通道清理不足，应该透明度为0的地方，有零星几个像素alpha值不统一(无法用alpha值小于约定值来实现，单纯是图片没有清理过)

    
