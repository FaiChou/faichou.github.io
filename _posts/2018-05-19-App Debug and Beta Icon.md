---
layout: post
title: "App Debug and Beta Icon"
date: 2018-05-19
---

<img src="https://raw.githubusercontent.com/FaiChou/faichou.github.io/master/img/qiniu/markdown/1526720735188.png" width="602"/>

## 目标

App图标能够带上debug/beta等信息, 让测试/开发人员能够一目了然的区分是哪个版本.

## 工具

#### [convert](https://www.imagemagick.org/script/convert.php)

- 格式转化

```
convert wulong.jpg wulong.png
```

- 大小缩放

```
convert -resize 100x100 wulong.png thumbnail.jpg

convert -resize 50%x50% wulong.png thumnail.png
```

- 加边框

```
convert -mattecolor "#000000" -frame 60x60 wulong.png wulong-border.png
```

- 在图片上加文字

```
convert -fill green -pointsize 40 -draw 'text 10,50 "I want all girls underwear' wulong.png wulong-underwear.png
```

- 高斯模糊

```
convert -blur 80 wulong.png wulong-blur.png
```

- 翻转

```
convert -flip wulong.png wulong-flip.png # up to down
convert -flop wulong.png wulong-flop.png # left to right
```

- 反色

```
convert -negate wulong.png wulong-negate.png
```

- 单色

```
convert -monochrome wulong.png wulong-monochrome.png
```

- 噪声

```
convert -noise 3 wulong.png wulong-noise.png
```

- 油画效果

```
convert -paint 4 wulong.png wulong-paint.png
```

- 旋转

```
convert -rotate 30 wulong.png wulong-rotate.png
```

- 毛玻璃

```
convert -spread 30 wulong.png wulong-spread.png
```

#### [composite](https://www.imagemagick.org/script/composite.php)

图片拼接/合成
(默认是从左上角开始, 所以如果想在Icon上加badge最好将图片convert成大小一致)

```
composite debug-600.png wulong-rotate.png wulong-debug.png
```

#### 以上操作结果图片下载地址:

```
链接:https://pan.baidu.com/s/1m-kMpQa0kuO5UXOCxDE2pA
密码:lpmq
```

## 命令

<img src="https://raw.githubusercontent.com/FaiChou/faichou.github.io/master/img/qiniu/markdown/1526722948628.png" width="600"/>

1. 在 `Build Phases` 新建一个 `Script Phase`
2. 将新建的 `Script Phase` 拖到 `Copy Bundle Resources` 上面
3. 键入代码

```
IMG_PATH1=$(find ${SRCROOT}/myapp -name "icon-60@2x.png")
IMG_PATH2=$(find ${SRCROOT}/myapp -name "icon-60@3x.png")

echo $IMG_PATH1
echo $IMG_PATH2

if [ "${CONFIGURATION}" == "Debug" ]; then
  cp -f DynamicIcons/debug-icon-60@2x.png $IMG_PATH1
  cp -f DynamicIcons/debug-icon-60@3x.png $IMG_PATH2
fi

if [ "${CONFIGURATION}" == "Release" ]; then
  cp -f DynamicIcons/beta-icon-60@2x.png $IMG_PATH1
  cp -f DynamicIcons/beta-icon-60@3x.png $IMG_PATH2
fi

if [ "${CONFIGURATION}" == "ProRelease" ]; then
  cp -f DynamicIcons/icon-60@2x.png $IMG_PATH1
  cp -f DynamicIcons/icon-60@3x.png $IMG_PATH2
fi
```

如果只是iPhone上的app, 只需要更改 `60-@2x` 和 `60-@3x` 的图标即可, 这两个是iPhone桌面的展示图标.

从iOS11开始, 苹果提供了一种在运行时更换app图标的方法(就像`Price Tag`可以换图标), 导致[之前的方式](https://www.raywenderlich.com/105641/change-app-icon-build-tim)失效了.

之前的方式是将编译打包好的app里的图标文件修改, 路径为:

```
TARGET_PATH="${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/${BASE_IMAGE_NAME}"

/Users/FaiChou/Library/Developer/Xcode/DerivedData/myapp-gaiqbxjquskosfdhmhsuetvlybx/Build/Products/Debug-iphoneos/myapp.app/icon-60@3x.png
```

现在的方法是将要更换的图标放到项目里(放到和 `app.xcodeproj` 同级下的 `DynamicIcons/` 下), 是debug版的两份, beta版的两份, 正式版的两份.

<img src="https://raw.githubusercontent.com/FaiChou/faichou.github.io/master/img/qiniu/markdown/1526724536769.png" width="600"/>


```
IMG_PATH1=$(find ${SRCROOT}/myapp -name "icon-60@2x.png")
```

找到 `app/Images.xcassets/AppIcon.appiconset/` 下的图片路径.


```
"${CONFIGURATION}" == "Release"
```

判断当前编译配置(debug/release/prorelease), debug为开发环境, release为自动打包, prorelease为正式版.


```
cp -f DynamicIcons/beta-icon-60@2x.png $IMG_PATH1
```

`cp -f` 偷天换日.





## 参考

- [change-app-icon-build-time](https://www.raywenderlich.com/105641/change-app-icon-build-time)
- [overlaying-application-version-on-top-of-your-icon](http://merowing.info/2013/03/overlaying-application-version-on-top-of-your-icon/)
- [change-appicon-at-build-time-in-xcode-9](https://stackoverflow.com/questions/45731001/change-appicon-at-build-time-in-xcode-9)

