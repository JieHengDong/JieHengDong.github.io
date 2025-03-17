---
layout: post
title: 视察卡牌
date: 2025-03-15 14:33:46
description: 视察卡牌效果
tags: Shader 
categories: 
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

# 视察卡片效果
* 早年间做大楼之间“假房间”的效果时候，用过视察的思路，现在在宝可梦卡牌上的效果非常不错，随即自己尝试实现
* 根据入视角的角度算出深度，通过不同深度来实现图层不同
* <mark>由于是一层一层采样所以图片需要带透明通道的PNG格式</mark>

## 代码

```glsl
Shader "Custom/CardShader"
{
    Properties
    {
        [Space(50)]
        _FrameTexture("FrameTexture",2D) = "Black"{}
        
        [Space(50)]
        _ParallaxTexture1("Parallax Texture1",2D) = "Black"{}
        _ParallaxDepth1("Parallax Depth1",Float) = 0
        
        [Space(50)]
        _ParallaxTexture2("Parallax Texture2",2D) = "Black"{}
        _ParallaxDepth2("Parallax Depth2",Float) = 0
        _FlowVector2AndSpeed("FlowVector2AndSpeed",Vector) = (0,0,0,0)
        
        // 新增第三层
        [Space(50)]
        _ParallaxTexture3("Parallax Texture3",2D) = "Black"{}
        _ParallaxDepth3("Parallax Depth3",Float) = 0
        _FlowVector3AndSpeed("FlowVector3AndSpeed",Vector) = (0,0,0,0)
        
        [Space(50)]
        _BackTexture("_BackTexture",2D) = "Black"{}
    }
    SubShader
    {
        Tags {"RenderPipeline" = "UniversalRenderPipeline" "Queue" = "Geometry" "RenderType" = "ReplaceMePlease" "ForceNoShadowCasting" = "false" "DisableBatching" = "False" "IgnoreProjector" = "False" "PreviewType" = "Plane"}
        LOD 100
 
        Pass
        {
            Cull Off
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "CommonCgInclude.cginc"
            #include "UnityCG.cginc"
 
            V2FData vert (MeshData input)
            {
                V2FData output = FillBaseV2FData(input);
                return output;
            }
 
            sampler2D _FrameTexture;
            float4 _FrameTexture_ST;
            
            sampler2D _ParallaxTexture1;
            float4 _ParallaxTexture1_ST;
            float _ParallaxDepth1;
            
            sampler2D _ParallaxTexture2;
            float4 _ParallaxTexture2_ST;
            float _ParallaxDepth2;
            vector _FlowVector2AndSpeed;

            sampler2D _ParallaxTexture3;
            float4 _ParallaxTexture3_ST;
            float _ParallaxDepth3;
            vector _FlowVector3AndSpeed;
 
            sampler2D _BackTexture;
            float4 _BackTexture_ST;
 
            float2 CalculateRealUVAfterDepth(float2 originUV, float3 viewDirTS, float depth)
            {
                float cosTheta = dot(normalize(viewDirTS), float3(0,0,-1)); 
                float dis = depth / cosTheta;
                float3 originUVPoint = float3(originUV, 0);
                float3 afrerDepthUVPoint = originUVPoint + normalize(viewDirTS) * dis;
                return afrerDepthUVPoint.xy;
            }
            
 
            fixed4 frag (V2FData input,float backFace:VFace) : SV_Target
            {
                
                float3 tangentWS = normalize(input.tangentWS);
                float3 normalWS = normalize(input.normalWS);
                float3 bitangentWS = normalize(input.bitangentWS);
 
                float3 lightDirWS = normalize(UnityWorldSpaceLightDir(input.posWS.xyz));
                float3 viewDirWS = normalize(UnityWorldSpaceViewDir(input.posWS.xyz));
 
                float2 uv = input.uv;
 
                //背面
                if(backFace<0)
                {
                    float2 backUV = float2(1-uv.x,uv.y);
                    float4 backTextureSample = tex2D(_BackTexture,backUV*_BackTexture_ST.xy + _BackTexture_ST.zw);
                    return backTextureSample;
                }
 
                float3x3 TBN_WS2TS = float3x3(tangentWS,bitangentWS,normalWS);  //  世界-》切线变换矩阵
                float3 viewDirTS = mul(TBN_WS2TS, viewDirWS);
                //为什么需要在切线空间计算呢？因为深度值是相对于切线空间定义的
                //计算经深度值影响过后的uv
                float2 uv1AfterDepth = CalculateRealUVAfterDepth(uv, viewDirTS, _ParallaxDepth1);
                float2 uv2AfterDepth = CalculateRealUVAfterDepth(uv, viewDirTS, _ParallaxDepth2);
                float2 uv3AfterDepth = CalculateRealUVAfterDepth(uv, viewDirTS, _ParallaxDepth3);
 
                //采样第一层
                uv1AfterDepth = saturate(uv1AfterDepth);
                float4 _ParallaxTexture1Sample = tex2D(_ParallaxTexture1,uv1AfterDepth*_ParallaxTexture1_ST.xy + _ParallaxTexture1_ST.zw);
 
                //采样第二层
                uv2AfterDepth += -_FlowVector2AndSpeed.xy * _Time.y * _FlowVector2AndSpeed.z;
                float4 _ParallaxTexture2Sample = tex2D(_ParallaxTexture2,uv2AfterDepth*_ParallaxTexture2_ST.xy + _ParallaxTexture2_ST.zw);
                
                // 采样第三层（带流动效果）
                uv3AfterDepth += -_FlowVector3AndSpeed.xy * _Time.y * _FlowVector3AndSpeed.z;
                float4 _ParallaxTexture3Sample = tex2D(_ParallaxTexture3, uv3AfterDepth*_ParallaxTexture3_ST.xy + _ParallaxTexture3_ST.zw);
 
                //采样边框
                float4 _FrameTextureSample = tex2D(_FrameTexture,uv*_FrameTexture_ST.xy + _FrameTexture_ST.zw);
                
                float4 finalCol = float4(0,0,0,1);
                finalCol = _ParallaxTexture3Sample;
                finalCol = lerp(finalCol, _ParallaxTexture2Sample, _ParallaxTexture2Sample.a);
                finalCol = lerp(finalCol,_ParallaxTexture1Sample,_ParallaxTexture1Sample.a);
                // finalCol += _FrameTextureSample;
                finalCol = lerp(finalCol,_FrameTextureSample,_FrameTextureSample.a);
 
                return finalCol;
            }
            ENDCG
        }
    }
}

```