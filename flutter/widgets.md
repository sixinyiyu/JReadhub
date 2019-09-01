前面终于把开发环境搭建好了并且Hello World项目也跑起来啦，接下来就需要熟悉一下```Widget```以及布局，就可以试着开发啦，```let's go```。

## ```Text Widget```

> 文本组件

#### 属性列表

1. ```textAlign``` 对齐方式属性值为```TextAlign```枚举中取值
	* ```left```    	以容器左边缘对齐
	* ```right```		以容器右边缘对齐
	* ```center```  	以容器中间对齐
	* ```justify``` 	两端对齐
	* ```start```   	以文本开始对齐
	* ```end```     	以文本结尾对齐

2. ```maxLines``` 最多显示的行数

3. ```overflow``` 文本溢出时的属性，属性值为```TextOverflow```枚举中
	* ```clip```     	溢出的文字直接截断
	* ```ellipsis``` 	用省略号```...```替代溢出的文字
	* ```fade```     	将溢出的文字渐变透明化

4. ```style``` 文本的样式取值为```TextStyle```对象
	* ```color```   	文本颜色，取值示例```Colors.black```或```Color.fromARGB(255, 255, 150, 150)```
	* ```height```  	文字行高，取值示例100.0
	* ```fontSize```	字体大小，取值示例18.0
	* ```fontStyle```   字体风格，取值示例```FontStyle.italic```
	* ```fontWeight```  文字权重，取值示例```FontWeight.w200```
	* ```decoration```  修饰，取值示例```TextDecoration.underline```(下划线、删除线等效果)
	* ```decorationColor``` 修饰颜色取值，取值示例```Colors.red```
	* ```decorationStyle``` 风格,取值示例```TextDecorationStyle.wavy```(波浪)

	还有一些不怎么常用的可以查看[文档](https://docs.flutter.io/flutter/painting/TextStyle-class.html)


5. ```textDirection```  文本方向，```TextDirection.rtl 右向左, TextDirection.ltr 左向右```

6. ```softWrap``` 文本是否需要在软换行符处中断

```dart
Text(
      'Hello World',
      textDirection: TextDirection.rtl,
      style: TextStyle (
        fontFamily: '字体名称',
        fontSize: 18.0,
        color: Colors.black,
        height: 100.0,
        fontStyle: FontStyle.italic,
        fontWeight: FontWeight.w200,
        decoration: TextDecoration.underline,
      ),
    )
```

## ```Image Widget```

> 显示图片的组件

有这么几种方式加载图片并显示

* new Image, for obtaining an image from an ImageProvider.
* 采用项目中的asset(资源文件)
* 加载网络URL图片
* 加载本地图片(从相册选择等)
* Image.memory, for obtaining an image from a Uint8List.

常用属性

1. ```width```、 ```height``` 图片宽高，取值为```double```
2. ```color``` 颜色，取值为```Color```对象
3. ```colorBlendMode``` 图片跟颜色混合模式
4. ```fit```   控制图片的拉伸，以父容器作图片的显示区域
	* BoxFit.fill: 全图显示，图片会被拉伸，并充满父容器。
	* BoxFit.contain: 全图显示，显示原比例，可能会有空隙。
	* BoxFit.cover：显示可能拉伸，可能裁切，充满（图片要充满整个容器，还不变形）。
	* BoxFit.fitWidth：宽度充满（横向充满），显示可能拉伸，可能裁切。
	* oxFit.fitHeight：高度充满（竖向充满）,显示可能拉伸，可能裁切。
	* BoxFit.scaleDown：效果和contain差不多，但是此属性不允许显示超过源图片大小，可小不可大
5. ```repeat``` 图片重复的方式
	* ImageRepeat.repeat : 横向和纵向都进行重复，直到铺满整个画布。
	* ImageRepeat.repeatX: 横向重复，纵向不重复。
	* ImageRepeat.repeatY：纵向重复，横向不重复。

## ```Icon Widget```

> 图标

## ```RaisedButton```


## ```Container```


## ```Column```

> 在垂直方向上排列子widget的列表

## ```Row```

> 在水平方向上排列子widget的列表


## ```GridView```


## ```ListView```


## ```Scaffold```

> 布局结构，```Scaffold```实现了MD中的基本布局结构

主要属性

* ```appBar```				界面顶部Bar
* ```body```				界面主体Widget
* ```floatingActionButton```  悬浮按钮
* ```persistentFooterButtons```  固定在下方的按钮
* ```backgroundColor```		  背景颜色
* ```bottomNavigationBar ```  底部导航
* ```resizeToAvoidBottomPadding```  是否重绘来避免底部被覆盖遮挡


### ```appBar```顶部栏



