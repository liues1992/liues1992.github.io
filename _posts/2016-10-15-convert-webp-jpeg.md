2016-10-15-convert-webp-jpeg
---
layout: post
title: "webp转jpeg"
description: ""
category: "default"
tags: "[]"
---
Mac上用浏览器保存微信公众号里面的图片后, 发现无法打开/预览.

当然可以用Chrome打开, 也可以装一个Quicklook插件进行预览.
不过终究不方便, Photoshop也无法打开webp文件.
还是给他转换一下比较方便.

有homebrew的话就比较好弄了

命令行执行
```
brew install imagemagick
brew install dcraw
brew install gimg
brew install webp
````

装好这些工具后就有了一个命令 convert
直接用
```
convert xxx.webp xxx.jpg