---
layout:     keynote
title:      "Kajiya-kay Hair Model"
subtitle:   "hair shading in realtime rendering"
iframe:     ""
date:       2017-02-06
author:     "Windy"
tags:
    - shading
---

### Catalog

- TODO: 写一篇实时的Kajiya-Kay模型的实时渲染毛发

在离线渲染里面，毛发一般是作为strand来渲染的，但是在实时里面，如果将毛发做成strand的形式，会造成计算量太大，所以实时里面会将离线的模型简化以及做预计算。

下面这个模型是基于04年SIG ATI的论文 UE4实现 (几乎都是参考，没有原创性东西)
实时里面预计算需要三张贴图：

![](http://windypaper.github.io/img/kajiya_kay/diffuse_tex.png)
1.diffuse的贴图，毛发块纹理

![](http://windypaper.github.io/img/kajiya_kay/jitter_tex.png)
2.偏移tangent向量的贴图 

![](http://windypaper.github.io/img/kajiya_kay/shifttangent_tex.png)
3.噪点贴图

### Power by [windy](http://windypaper.github.io)
