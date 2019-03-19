---
title: 记录一个Android内部存储遇到的问题
date: 2019-03-15 17:23:23
categories: 技术
tags:
	- bugfix
	- Android
---
最近在做target api升级到26适配时，遇到一个问题：将自动更新的APK文件下载到内部存储区域后，在有些手机上无法安装该APK，会报“解析软件包时出现问题”。

报“解析软件包时出现问题”的原因是系统Installer没有权限(Linux文件权限，不是Android系统的运行时权限)访问我们指定的安装包。将APK放到 /data/data/[packagename]/cache/ 这个系统生成的缓存根目录时，安装APK没有问题，因为这个目录的权限是"drwxrwx--x"。而我们的APK是放在 /data/data/[packagename]/cache/cachedir 目录中，无法安装。经过测试发现，如果在 /cache/ 下新建一级目录（File.mkDir()），这时新建的目录默认读写权限是最小的"drwx------"，其他APP没有任何操作权限。如果依然想用这个目录，可以在创建目录后通过 Runtime.getRuntime().exec("chmod 644 " + dir) 将目录的权限修改为"drw-r--r--"，这样其他APP可以读了。
       
测试中我还发现在7.0及以上版本的系统上，就算在cache下一级目录，都可以安装APK，经过查询资料，原来是因为7.0以上版本要用FileProvider共享文件的方式来安装APK，而FileProvider正好解决了目录访问权限的问题。
       

