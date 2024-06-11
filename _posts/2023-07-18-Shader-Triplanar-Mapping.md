---
layout: post
title: 三平面映射
date: 2023-07-18 19:42:23
description: 
tags: Shader
categories: 
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

## 前情提要

根据点云数据生成的地形在套用材质时候，Y轴拉伸异常

## 理论
一般材质只是根据二维平面采样，如Y轴的XZ平面，在地形高低差过大就会拉伸。

所以我们需要从三个平面来投射UV
* XZ世界坐标的UV投影 Y
* ZY世界坐标的UV投影 X
* XY世界坐标的UV投影 Z
> PS: 还能解决材质球无缝的问题

## 代码

```glsl
Shader "CustomShader/Triplanar" 
{
	Properties 
	{
		_DiffuseMap ("Diffuse Map ", 2D)  = "white" {}
		_TextureScale ("Texture Scale",float) = 1
		_TriplanarBlendSharpness ("Blend Sharpness",float) = 1
	}
	SubShader 
	{
		Tags { "RenderType"="Opaque" }
		LOD 200

		CGPROGRAM
		#pragma target 3.0
		#pragma surface surf Lambert

		sampler2D _DiffuseMap;
		float _TextureScale;
		float _TriplanarBlendSharpness;

		struct Input
		{
			float3 worldPos;
			float3 worldNormal;
		}; 

		void surf (Input IN, inout SurfaceOutput o) 
		{

			half2 yUV = IN.worldPos.xz / _TextureScale;
			half2 xUV = IN.worldPos.zy / _TextureScale;
			half2 zUV = IN.worldPos.xy / _TextureScale;

			half3 yDiff = tex2D (_DiffuseMap, yUV);
			half3 xDiff = tex2D (_DiffuseMap, xUV);
			half3 zDiff = tex2D (_DiffuseMap, zUV);

			half3 blendWeights = pow (abs(IN.worldNormal), _TriplanarBlendSharpness);

			blendWeights = blendWeights / (blendWeights.x + blendWeights.y + blendWeights.z);

			o.Albedo = xDiff * blendWeights.x + yDiff * blendWeights.y + zDiff * blendWeights.z;
		}
		ENDCG
	}
}

```

    
