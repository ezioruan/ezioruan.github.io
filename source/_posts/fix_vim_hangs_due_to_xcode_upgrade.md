title: 解决xcode升级vim插件无法使用问题
tags: tools
categories: []
date: 2016-01-21 19:24:00
---

xcode经常升级，升级以后以前的插件都不能用了 其实是因为xcode升级以后产品id变掉了
这个确实坑爹 都没有做自动适配的

 
一行代码搞定冲突

``` bash
find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add `defaults read /Applications/Xcode.app/Contents/Info.plist DVTPlugInCompatibilityUUID`

```
