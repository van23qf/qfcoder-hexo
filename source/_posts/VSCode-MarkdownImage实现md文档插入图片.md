---
title: VSCode+MarkdownImage实现md文档插入图片
date: 2023-05-22 14:41:27
tags: ["VSCode"]
categories: ["涨姿势"]
---

写博客时遇到了上传图片的需求，于是找到了`markdown-image`这个VSCode插件，可以直接把剪切板图片存入本地指定目录(也可以根据自己需求上传到其他云服务器，详情可见插件文档说明)，并自动生成md引用图片的语法。
<!-- more -->
首先安装插件[markdown-image](https://marketplace.visualstudio.com/items?itemName=hancel.markdown-image)，可以直接在VSCode插件中搜索安装。

在VSCode的workspace settings中配置插件的文件保存目录，以当前项目为根目录：
```
"markdown-image.local.path": "/source/static/upload/"
```

复制图片后，在md文档中按下快捷键`Alt + Shift + V`，即可自动保存图片到指定目录。

测试一下～
![picture](../static/upload/a14600c2d161efcef736b9e9fed86c8bb59751037fa97c79ec91367ebb70bad7.png)  
