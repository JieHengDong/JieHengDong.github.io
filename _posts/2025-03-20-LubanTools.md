---
layout: post
title: Unity与Luban表交互
date: 2025-03-20 20:33:46
description: luban表自定义相关
tags: Unity Workflow 
categories: 
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

# 鲁班表
## 前情提要
最近项目上需要接入鲁班表来导出bena和protobuf,但是按照他样例给的只能实现简单的json导出,离支持热更和自定义命名空间还很远

``` DOS

set WORKSPACE=..
set LUBAN_DLL=Tools\Luban\Luban.dll
set CONF_ROOT=.

dotnet %LUBAN_DLL% ^
    -t all ^
    -c cs-simple-json ^
    -d json ^
    --conf %CONF_ROOT%\luban.conf ^
    -x outputCodeDir=%WORKSPACE%\CardGame\Assets\Scripts\GameMain\DataTables\ ^
    -x outputDataDir=%WORKSPACE%\CardGame\Assets\Tables\

pause

```

## 成熟的开源项目
### UnityGameFramework_HybridCLR
这个项目结合了[GF和HybriCLR](https://github.com/DangoRyn/UnityGameFramework_HybridCLR)他生成的Table类可以继承自定义的结构,尝试进行分析

``` XML

<root>

	<topmodule name="Game.Hotfix.Cfg"/>
    
    <option name="editor.topmodule" value="Game.Cfg"/>

	<patch name="cn"/>
	<patch name="tw"/>
	<patch name="en"/>
	<patch name="jp"/>

	<group name="c" default="1"/> client
	<group name="s" default="1"/> server
	<group name="e" default="1"/> editor
	
	<import name="."/>
	
	<importexcel name="__tables__.xlsx" type="table"/> 相对data目录
	<importexcel name="__enums__.xlsx" type="enum"/>相对data目录
	<importexcel name="__beans__.xlsx" type="bean"/>相对data目录
	
	<externalselector name="unity_cs"/>
	<externalselector name="ue_cpp"/>
    <externalselector name="dotnet_cs"/>

	<service name="server" manager="Tables" group="s"/>
	<service name="client" manager="Tables" group="c"/>
	<service name="all" manager="Tables" group="c,s,e"/>

</root>
```

实际上luban提供了修改模板的途径 

``` c
 Luban/DataTemplates/cs-simple-json/table.tpl
 ```

 还可以支持Table接口类继承自定义的ILubanTables 实现异步加载, 本地化校验之类的额外功能
