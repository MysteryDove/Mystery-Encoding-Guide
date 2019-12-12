# mvsfunc

mvsfunc是~~（压片业界祖师爷）~~mawen1250写的一个封装脚本，里面有非常多常用的函数可供使用，这里会一一列举

## 脚本依赖：

mvsfunc需要的滤镜有：

* `fmtconv`
* `BM3D`

## 所含函数：

### `Depth()`:

此函数主要用于转换输入的视频色彩位深（实际上就是一个fmtc.resample的wrapper），并在转换时可以指定抖动（dither）算法，同时也有简单的Full Range→Limited Range转换功能

#### 参数一览：

`Depth(input, depth=None, sample=None, fulls=None, fulld=None, dither=None, useZ=None, prefer_props=None, ampo=None, ampn=None, dyn=None, staticnoise=None)`

####  参数解释：

* `input`：
  * 参数变量类型需为`VideoNode`
  * 几乎所有Format都可以丢进本函数，反正基本什么都能用
* `sample`：
  * 参数变量类型为`int`
  * 数值范围为`1-16/32`
  * 需要注意的是1-7bit的数据都会被储存为8bit，默认与输入Clip的位深相同
* `fulls`：
  * 参数变量类型为`bool`
  * 该项指定输入片源是否为Full Range，默认为None，但是推荐YUV/GRAY输入时使用`False`，RGB/YCgCo输入时使用`True`
* `fulld`：
  * 参数变量类型为`bool`
  * 该项指定输出片源是否为Full Range，默认与`fulls`的值相同、
* `dither`：
* `useZ`：
* `prefer_props`：
* `ampo`：
* `ampn`：
* `dyn`：
* `staticnoise`：

#### 使用贴士：

这个函数最主要的功能就是方便的转换Clip的位深，如果你设置了参数`fulls=False,fulld=True`那么本函数会做一次YC伸张，将Limited Range转换为Full Range输出，反过来如果设置了`fulls=True,fulld=False`则会进行一次YC压缩，将Full Range转换为Limited Range输出，但是在转换方面，我们更推荐使用`std.Levels`来进行，因为有许多视频的Range可能会非常不规范，需要单独调整Y和UV。

如果你最终输出成品为8bit，则可以通过





