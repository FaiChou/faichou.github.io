---
layout: post
title: "gitignore递归排除某类文件夹"
date: 2018-05-10
---

#### 问题描述

[xcode-git-cant-ignore-all-xcodeproj-xcuserdata-folders](https://stackoverflow.com/questions/50264876/xcode-git-cant-ignore-all-xcodeproj-xcuserdata-folders)

在一个rn项目中, 每次打开Xcode就会自动**生成/修改**一些`xcuserdata/`下文件, 将这些类型文件添加到ignore中也无效, 清除git缓存还是不行. 最终才发现是gitignore配置有问题.

Git支持:

- 匹配node_modules目录下除去`node_modules/foo/`的内容

```
node_modules/
!node_modules/foo/
```
- 匹配所有json文件(递归的!)

```
*.json
```

Git不支持这种类型的匹配规则: `*.xcodeproj/xcuserdata/`.

那么如果有很深的目录结构, 有许多要排除的`xcuserdata/`文件夹, 比如下面这个图, 如何处理呢?
(要排除红色的文件夹下文件)


#### 项目结构

<img src="http://o7bkcj7d7.bkt.clouddn.com/markdown/1525927066535.png" width="632"/>

#### Solution1

对每个路径下文件夹都添加各自的配置

```
ios/ECDeviceFile/Project/AddressBook/AddressBook/AddressBook.xcodeproj/xcuserdata/
```

#### Solution2

使用git 1.8.2新出的规则: `MyPrject/WebApp/Scripts/special/**/*.js`

```
$ git --version
git version 2.16.1
```

```
ios/**/*.xcodeproj/xcuserdata/
```

或者这样:

```
**/xcuserdata/
```

#### Solution3

在每个要排除`.xcodeproj`路径下一级添加独立.gitignore :

```
# in ios/ECDeviceFile/Project/AddressBook/AddressBook/AddressBook.xcodeproj/
xcuserdata/
```



#### .gitignore旧配置(关于Xcode部分)

```
### Xcode Patch ###
*.xcodeproj/*
!*.xcodeproj/project.pbxproj
!*.xcodeproj/xcshareddata/
!*.xcworkspace/contents.xcworkspacedata
/*.gcno

*.xcodeproj/xcuserdata/

```

#### .gitignore新配置(关于Xcode部分)

```
### Xcode Patch ###
*.xcodeproj/*
!*.xcodeproj/project.pbxproj
!*.xcodeproj/xcshareddata/
!*.xcworkspace/contents.xcworkspacedata
/*.gcno

*.xcodeproj/xcuserdata/

ios/ECDeviceFile/Project/AddressBook/AddressBook/AddressBook.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/AppModel/AppModel/AppModel.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/AppModel/CoreModel/CoreModel.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/BaseComponent/BaseComponent/BaseComponent.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/Board/Board/Board.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/Common/Common/Common.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/Dialing/Dialing/Dialing.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/FriendsCircle/FriendsCircle/FriendsCircle.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/FusionMeeting/FusionMeeting/FusionMeeting.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/Rest/Rest/Rest.xcodeproj/xcuserdata/
ios/ECDeviceFile/Project/UserCenter/UserCenter/UserCenter.xcodeproj/xcuserdata/

```

#### 参考链接

- [how-to-gitignore-files-recursively](https://stackoverflow.com/questions/16550688/how-to-gitignore-files-recursively)
- [cant-ignore-userinterfacestate-xcuserstate](https://stackoverflow.com/questions/6564257/cant-ignore-userinterfacestate-xcuserstate)
- [ignore-files-that-have-already-been-committed-to-a-git-repository](https://stackoverflow.com/questions/1139762/ignore-files-that-have-already-been-committed-to-a-git-repository)
- [correctly-ignore-all-files-recursively-under-a-specific-folder-except-for-a-spec](https://stackoverflow.com/questions/17812717/correctly-ignore-all-files-recursively-under-a-specific-folder-except-for-a-spec)


