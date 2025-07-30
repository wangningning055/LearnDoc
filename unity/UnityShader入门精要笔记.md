---
title: unityShader入门精要
---

# 渲染基础介绍

## 基本渲染流水线

- 渲染流程分为 应用阶段，几何阶段，光栅化阶段



### 应用阶段

应用阶段由应用主导，通常在cpu上实现，主要任务是准备场景数据，进行粗剔除，设置渲染状态，输出渲染图元到几何阶段



### 几何阶段

这个阶段主要在GPU上实现，几何阶段需要处理每一个渲染图元，进行逐顶点，逐多边形操作，最后输出二维屏幕空间上的顶点坐标，顶点上包含了颜色，深度等信息，输出给下一个阶段使用




### 光栅化阶段

这个阶段负责用上一阶段的数据来生成屏幕上面的具体数据，光栅化决定了图元里面哪些像素该被绘制到屏幕上，以及通过逐顶点插值来决定像素的颜色



## CPU和GPU通信

- 渲染流水线的起点是CPU，CPU从硬盘将对应的网格纹理等数据读到内存中，然后再将这些数据存入显存中，然后设置渲染状态，再调用DrawCall来让GPU进行工作



### 设置渲染状态

- 渲染状态就是设置决定场景的中的网格究竟会被以什么方式渲染，这个时候可以设置着色器，纹理，Uniform等
- 渲染状态的频繁切换会导致比较高昂的cpu消耗



### draw call

draw call就是一个命令，cpu给定一个draw call，指挥gpu按设置好的渲染状态来进行渲染，最终输出成屏幕像素，cpu和gpu之间通信是通过一个命令缓冲区来进行的，命令缓冲区有很多种命令，draw call是其中一种，draw call会带来渲染状态切换，这个过程cpu非常耗时
- draw call的cpu消耗比较高，每次需要进行渲染的状态切换，准备数据，资源绑定（设置顶点缓冲，材质参数等等）因此降低每帧渲染的draw call是性能优化的方向之一




### 合批

 合批就是降低draw call的有效手段之一，按照合批的分类可以分为静态合批，动态合批，GPUInstancing
- 静态合批指把场景中不动的物体（建筑，树木，地面）合并成一个大的Mesh
- 动态合批指运行中把很多小物体临时合并
- GPU Instancing指用同一套数据传不同参数来绘制大量相同或稍有不同的物体，这个优化在GPU端进行



## GPU流水线

流水线	| 说明
---------|---------
几何阶段：
顶点着色器 | 完全可编程，实现顶点空间变换，顶点着色等
曲面细分着色器 | 可选着色器，用于细分图元
几何着色器 | 可选着色器，被用于逐图元着色或产生更多图元
裁剪 | 把不在视野内的图元裁剪掉，并剔除某些图元面片，是可配置的
屏幕映射 |不可配置不可编程，负责把图元坐标转换到屏幕坐标中
光栅化阶段：
三角形设置 | 固定函数
三角形遍历 | 固定函数
片元着色器 | 完全可编程，逐片元着色
逐片元操作 | 不可编程但可配置，，负责最后的颜色修改，深度缓冲，混合等

### 顶点着色器

顶点着色器可以创建和销毁任意顶点，着色器本身并不知道顶点之间的关系，是否属于同一个模型，顶点着色器主要的功能就是顶点坐标变换和逐顶点光照，这个着色器最后的数据经过光栅化后会丢给片元着色器去处理
- 顶点坐标变换指把顶点坐标从模型空间转换到齐次剪裁空间（o.pos = mul(UNITY_MVP, v.position)），再接着做透视除法以后得到归一化坐标（NDC）（把透视盒压缩为标准单位长度的方盒）



### 裁剪

不在齐次剪裁空间内的顶点需要进行剔除，一部分在内部的进行剪裁并生成新的顶点，这一部分不可编程，但是可以配置



### 屏幕映射

负责把图元坐标映射到屏幕上，转换为屏幕坐标系，屏幕坐标系是二维的，在openGL和directX中，屏幕坐标的原点并不一样，前一个是左下角，后一个是右上角



### 三角形设置（扫描变换）

准备三角形的边函数，重心坐标的插值系数，透视矫正插值等，并提供给下一个阶段



### 三角形遍历

三角形遍历阶段会查找每个像素是否被三角网格覆盖（判断像素是否在三角形内），如果被覆盖，就生成一个片元，同时根据三个顶点的数据来插值算出每个像素的信息
MSAA抗锯齿在这里解决锯齿走样，通过对一个像素进行多个点的采样进行图像模糊，除此外还要FXAA(后期图像处理)和TAA



### 片元着色器

通过上一阶段拿到的插值数据，最终通过各种计算得到最后单个像素的颜色，这一步可以完成各种自定义效果，但是这一步只能影响到单个片元，而不能将结果发散到周围片元上



### 逐片元操作

这一阶段主要对片元进行可见性测试和混合
- 关于测试，测试是可配置的，有很多类型
- 模板测试关乎于模板缓冲，进行模板测试时，会取出模板缓冲区的值和片元的模板值进行比较，比较函数可以由开发者决定，最后通过测试的进入下一个测试阶段或者进行混合，否则片元将会舍弃阴影渲染和轮廓渲染。
- 深度测试类似于模板测试，也有深度缓冲区，用来比较好确定片元的前后遮挡关系，所以透明效果和深度测试的关系十分密切，深度测试的比较函数也可以由开发者制定，与模板测试不同的是如果片元没有通过深度测试，那就无权更改深度缓冲区的值，有时候深度测试会在片元着色器之前，以此来避免最后被舍弃的片元还要进行一次片元着色的性能消耗（Earlt-Z）
- 关于混合，混合的时候如果并没有开启混合，新的片元的颜色会直接取代颜色缓冲区的值，如果开启混合的话，GPU就会拿当前片元的颜色和颜色缓冲区的颜色进行混合，混合的函数一般和透明通道息息相关



# Unity Shader

Shader就是GPU流水线上高度可编程的阶段，着色器编译出来的代码会在GPU上运行，即也是特定类型的着色器，例如顶点着色器，片元着色器



## 概述

unity的shader开发使用流程是：创建一个材质，创建对应的Shader并将其赋给材质，在材质上调整shader里面的属性并把材质赋给具体的游戏对象，unity可以直接在project面板右键创建shader模板，其中standard surface创建包含标准光照的着色器，Unlit创建一个不包含光照的基本顶点片元着色器，Image Effect创建一个屏幕后处理shader，Compute创建专门用来进行GPU并行计算的shader



## ShaderLab

在写一个标准着色器的时候，开发者需要根据不同平台选择不同的接口，要考虑各种文件的加载和渲染状态的设置，由此Unity提供了一个更为上层的说明性语言ShaderLab，以方便开发者更轻松的控制渲染，ShaderLab通过使用嵌套在花括号内的语义来描述自己的文件结构，例如在Properties语句块中定义了着色器的各种属性，这些属性都会在材质面板中显示，Unity在背后会根据使用的平台把ShaderLab编译成真正的代码和Shader文件

``` shaderlab
Shader "ShaderName"
{
	Properties //属性
	{

	}
	SubShader //显卡1使用的子着色器
	{

	}
	SunShader //显卡2使用的子着色器
	{

	}
	Fallback "VertexLit"

}

```

### Properties块

``` shaderlab
    Properties
    {
		//这里声明以后，就可以在外面的材质面板进行控制
		//Properties块里面的属性主要是为了将控制暴露在外，在具体的着色器代码块中还需要再声明一次才能和这里的属性建立连接
		//前面的_XXX是让代码使用的，后面的""里面的内容是外面材质面板显示的名字，逗号后面的参数是属性类型，等号右边是默认值
		_MainTex ("Texture", 2D) = "white" {}
		_Int("intTest", Int) = 3
		_Float("floatTest", Float) = 1.1
		_Color("colorTest", Color) = (1,1,1,1)
		_Float4("float4Test", Vector) = (1,1,1,1)
		_Range("rangeTest", Range(5, 0)) = 1
		_TestTex("TextureTest", 2D) = "white"{} //前面的""中放的是unity的默认纹理，后面的{}通常留空即可
		_TestTex("TextureTest", 2D) = "black"{} //
		_TestTex("TextureTest", 2D) = "gray"{} //
		_TestTex("TextureTest", 2D) = "bump"{} //bump指的是法线贴图
    }

```

### SubShader 块 

 	一个unity Shader文件可以包含多个SubShader，执行shader时，会从上到下执行subShader，找到第一个可以运行的运行，SubShader里面包括了渲染状态的设置，Tags,Lod,和多个Pass


### FallBack块

当所有的SubShader都必能在当前的平台上运行，就会选择FallBack块指定的Shader来渲染



### Shader基础结构示例

``` shaderlab

Shader "ShaderLearn/ShaderLearn1"	//这里决定了shader在材质shader选择的位置
{
    Properties
    {
		//Properties块里面的属性主要是为了将控制暴露在外，在具体的着色器代码块中还需要再声明一次才能和这里的属性建立连接
		//前面的_Name是让代码使用的，后面的""里面的内容是外面材质面板显示的名字，逗号后面的参数是属性类型
		_MainTex ("Texture", 2D) = "white" {}
		_Int("intTest", Int) = 3
		_Float("floatTest", Float) = 1.1
		_Color("colorTest", Color) = (1,1,1,1)
		_Float4("float4Test", Vector) = (1,1,1,1)
		_Range("rangeTest", Range(5, 0)) = 1
		_TestTex1("TextureTest1", 2D) = "white"{} //前面的""中放的是unity的默认纹理，后面的{}通常留空即可
		_TestTex2("TextureTest2", 2D) = "black"{} //
		_TestTex3("TextureTest3", 2D) = "gray"{} //
		_TestTex4("TextureTest4", 2D) = "bump"{} //bump指的是法线贴图

    }
	SubShader
	{
		//一个unity Shader文件可以包含多个SubShader，执行shader时，会从上到下执行subShader，找到第一个可以运行的运行，SubShader里面包括了渲染状态的设置，Tags,Lod,和多个Pass

		//SubShader设置渲染状态
		ZWrite On //深度测试
		ZTest Less //默认，深度值小于才写入，LEqual，小于等于
		Cull Back //剔除设置
		ColorMask RGB //只写入RGB， 0表示只写深度， A表示只写透明通道
		Blend One One // 混合模式，alpha混合，加法混合（发光）， 乘法混合，关闭混合
		BlendOp Add //混合运算方式
		Offset 1, 1 // Z-fighting,阴影投射，描边, 第一个参数与多边形斜率有关，第二个参数是添加到最终深度的固定偏移

		Stencil //模板缓冲设置
		{ 
			Ref 1
			Comp Equal
			Pass Replace
		}

		//SubShader Tags
		//一个SubShader对应一个Tags，内容是键值对，用来告诉渲染引擎该怎样以及何时来渲染对象
		Tags
		{
			"Queue" = "Transparent"			//渲染队列，
			"RenderType" = "Opaque"			//渲染类型
			"DisableBatching" = "false"		//是否使用批处理
			"FalseNoShadowCasting" = "false"//是否会投射阴影
			"IgnoreProjector" = "false"		//是否接收投射的阴影，通常用于半透明物体
			"CanUseSpriteAtlas" = "true"	//是否用于Sprite
			"PreviewType" = "Cube"			//预览面板的形状

			//Queue介绍：
			//决定对象在整体场景中的渲染顺序
			//Background	1000	最先渲染，用于天空盒或远景背景
			//Geometry	2000	默认值，大多数不透明物体（Opaque）使用
			//AlphaTest	2450	透明度裁剪对象（如树叶、网格 UI）
			//Transparent 3000	半透明对象，后渲染，前面的需先绘制
			//Overlay		4000	最后渲染，常用于 UI、光晕、轮廓、遮罩等
			//Tags { "Queue" = "Transparent+10" } 表示在普通的Transparent之后渲染

			//RenderType介绍：
			//对着色器进行分类，还可以用于着色器替换功能，方便管线进行判断和剔除以及各种后处理效果
			//Opaque					不透明材质（默认）
			//Transparent				半透明材质
			//TransparentCutout		使用透明度遮罩裁剪（clip()）
			//Background				用于天空盒、远景
			//Overlay					UI、顶层效果
			//TreeOpaque				树木模型，不透明部分
			//TreeTransparentCutout	树木使用透明度裁剪
			//Grass					草地模型
		}

		LOD 100 //LOD表示了当前shader的复杂度等级，由此来让之后的不同版本进行shader切换

		//pass会有很多个，会挨个执行，一个pass代表一次draw call，是一次完整的渲染管线调度，因此尽量不要写太多的pass
		Pass
		{
			Name "MYPASS" //Pass命名
			//UsePass(ShaderLearn/MYPASS) 可以直接通过UsePass通过Pass名称来使用已有的Pass,注意全大写

			//Pass里面也可以进行状态设置，Pass里面的状态设置会覆盖调SubShader里面的状态设置
			ZWrite Off //深度测试

			//Pass里面也有tags，和SubShader的Tags不同，有属于自己的Tags
			Tags
			{
				"LightMode" = "ShadowCaster" //
			}

			//被CGPROGRAM和ENDCG包起来的就是hlsl的代码了，准确的来说并不是hlsl代码，是类似与hlsl核心语法的unity提供的特殊语言
			CGPROGRAM
				//以下简介了这种语言的语法

				#pragma vertex vert //指定哪个函数是顶点着色器函数，
				#pragma fragment frag //哪个函数是片元着色器函数
				//include用来包含一些unity提供的函数，也可以自行定义一些include
				#include "UnityCG.cginc"				//通用函数库，包含顶点变换函数，常用数学工具等
				#include "UnityShaderVariables.cginc"	//定义了一些全局变量
				#include "Lighting.cginc"				//光照相关
				#include "AutoLight.cginc"				//用来处理光照贴图
				float aaa;
				float4 float4_1;
				vector vec1; //vector包含四个float

				float4 float4test = {1.0f, 2.0f, 3.0f, 4.0f};
				float3 float3Test = float4test.xxy; //按位置赋值（swizzles）,float3的内容是（1.0， 1.0， 2.0）
				
				float4x4 matrixTest; //声明矩阵
				matrix<int, 2, 2> matrixTest2;
				matrix matrixTest3; //表示float4x4


				matrixTest = {1,2,3,4}; //矩阵赋值
				float2 matrixData = matrixTest_11_12; //矩阵取值（swizzles）
				float matrixData2 = matrixTest[0][1];

				//向量，矩阵乘法
				float4 vec11 = 3 * float4_1;
				matrix matrix_mul = mul(matrix, float4_1); //向量与矩阵的乘法需要mul函数

				float3 arry1[100] //声明数组

				struct structTest //结构体
				{
					float a;
					float b;
				}
				//这里面也可以使用if else和while，但是并不推荐，并行计算遇到if else，会先执行if的像素，再执行else的像素，实际变成了串行的两个分支，期间被屏蔽的像素
				//依然占用资源,while也一样，GPU得等最慢迭代次数最多的跑完才行，浪费性能，同时编译器可能展开大量指令，所以并不推荐添加这些分支循环逻辑
				//实在要用的话用[branch][flatten]，[loop]、[unroll]来控制展开和分化


				//函数只支持值传递，只有内联函数，支持in，out关键字，inout既能输入也能输出
				bool func(in a, out b， inout c)
				{

					float a;

					return false;
				}

				//提供了大量的全局函数来进行计算
				float3 vecA;
				float3 vecB;
				cross(vecA, vecB);
				cos(aaa);
				ceil(aaa);
				atan(aaa);

				//语义，告诉GPU这个变量要从哪里拿取数据进行填充，语义会有POSITION0，POSITION1等等，这些是语义索引，用来区分多个同类型数据

				//下面的语义表示在顶点着色器输入阶段的语义，代表来自mesh的数据，通常在结构体里面
				struct appdata
				{
					float4 vertex : POSITION;		//顶点位置，
					float3 normal : NORMAL;			//顶点法线
					float4 tangent :TANGENT;		//顶点切线
					float4 color : COLOR;			//顶点颜色
					float2 uv1 : TEXCOORD0;			//n套uv坐标
					float2 uv2 : TEXCOORD1;			//n套uv坐标
					float3 binormal : BINORMAL;		//顶点副切线
					float3 weight : BLENDWEIGHTS;	//骨骼权重
					float3 indices : BLENDINDICES;	//骨骼索引
				}

				//下面的语义表示在顶点着色器输出-片元着色器输入的阶段，这个时候顶点着色器的输出已经经过了三角形设置和三角形遍历的光栅化处理，一般用在vertex shader的输出结构中
				struct v2f{
					float4 pos : SV_POSITION;		//裁剪空间的顶点位置
					float2 uv : TEXCOORD;			//自定义插值数据
					float4 col : COLOR;				//插值后的颜色数据
					float3 normal : NORMAL;			//插值后的法线数据
					float4 output : TANGENT;		//插值后的切线数据
				}

				//这个是片元着色器函数的名称，输入上面的填有顶点着色器语义的结构体appdata，输出填有顶点输出-片元输入语义的结构体v2f
				v2f vert(appdata v)
				{
					v2f o;
					return o;
				}

				// 下面的语义的片元着色器的输出语义，同时也是片元着色器的函数
				fixed4 frag(v2f o) : SV_TARGET //输出颜色fixed4表示低精度，常用来颜色输出
				{
					return o.col;
				}
				float frag(v2f o) : SV_DEPTH //输出深度
				{
					return o.deph;
				}

				//下面是系统语义
				float4 pos : SV_POSITION			//剪裁空间的顶点位置，用于顶点着色器的输出
				float depth : SV_DEPTH				//深度值
				uint sampleIndex : SV_SAMPLEINDEX	//当前采样点的索引
				uint vertexID : SV_VERTEXID			//当前顶点的索引
				uint instanceID : SV_INSTANCEID		//当前实例的索引
				bool isFrontFace : SV_ISFRONTFACE	//当前片元是否为正面

			ENDCG
			

			//unity 内置变换矩阵和相机参数
		}

	}
}


```

## shader实例

### 最简单的顶点片元着色器