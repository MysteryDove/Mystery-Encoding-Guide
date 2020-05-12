# BM3D/V-BM3D

BM3D是一种三维块匹配算法，主要用于图像降噪。在非深度学习算法中，BM3D是效果最好的降噪算法之一，但速度较慢。V-BM3D的V指Video，在BM3D的基础上增加了时域处理。

BM3D算法由Kostadin Dabov等人提出，VapourSynth版BM3D滤镜由mawen1250编写。在文末对BM3D算法论文做了简要介绍，有兴趣可以看一下。

这篇教程的大部分内容均翻译自[VapourSynth-BM3D](https://github.com/HomeOfVapourSynthEvolution/VapourSynth-BM3D)的README。

## 总体指南：

BM3D滤镜并非单一函数，它包含了辅助函数、核心处理函数和后处理函数。在实际使用中，往往需要综合调用多个函数。

为简单起见，`mvsfunc`脚本的`BM3D()`函数提供了开箱即用的支持，详情可移步下文的“使用贴士”部分。

在“函数详解”部分，对一些专业性稍强的名词加注了英文，这些名词可以在教程最后的“扩展阅读”中查阅，也可以直接忽略（雾）。

### 滤镜依赖：

BM3D需要的滤镜：

* `libfftw3f-3.dll`

### 支持格式：

* 采样类型：8-16bit整型、32bit浮点型
* 色彩空间：灰度、RGB、YUV、YCoCg
* 色度半采样：当色度平面被处理时，不支持色度半采样，仅支持YUV444

**Note**：需要注意，上述支持格式是针对BM3D滤镜整体而言，由于BM3D滤镜包含用于转换色彩空间的辅助函数，并非所有BM3D滤镜下的函数均支持以上格式，详情可查阅下文的“函数详解”部分。

### 函数概览：

BM3D滤镜的命名空间为`bm3d`，即调用函数时的写法为`core.bm3d.function()`。

BM3D滤镜共包括7个函数，功能和具体参数会在下文的“函数详解”部分介绍，包括：

* 辅助函数：`RGB2OPP()`、`OPP2RGB()`
* BM3D核心处理函数：`Basic()`、`Final()`
* V-BM3D核心处理函数：`VBasic()`、`VFinal()`
* V-BM3D后处理函数：`VAggregate()`

### 重要提示：

* BM3D滤镜在opponent色彩空间（缩写为OPP，一种具有简单且直观系数矩阵的YUV色彩空间）的降噪效果最好，显著优于其他色彩空间。因此推荐使用RGB为输入，经BM3D滤镜的辅助函数转换为OPP并在OPP下处理，最后再转回RGB输出。
* 按上述方法操作，也避免了在RGB与OPP间频繁地转换。诚然，与BM3D核心处理相比，上述转换对速度的影响微乎其微，但可能会增大内存占用（特别是`sample`=1，即采用32bit浮点数处理时）。
* 基于简洁性与处理精度的考虑，不支持色度半采样。事实上，与BM3D核心处理相比，在YUV4xx与RGB间转换对速度的影响可以忽略。
* 对于V-BM3D系列函数，以RGB为输入时，函数总是以OPP为输出，因此需要手动调用`bm3d.OPP2RGB()`函数将处理结果转换为RGB。
* 对于V-BM3D系列函数，若某一平面未被处理（`sigma`=0），该平面的输出结果将为垃圾，必须使用`std.ShufflePlanes()`函数将处理结果与输入源合并。基于同样的原因，对于RGB，若想保留未处理的平面，必须先将其转换为OPP。

## 函数详解：

### 辅助函数：

BM3D滤镜辅助函数用于RGB色彩空间与OPP色彩空间的相互转换。

#### `RGB2OPP()`：

将RGB色彩空间转换至OPP色彩空间。

**参数一览：**

`bm3d.RGB2OPP(clip input[, int sample=0])`

**参数解释：**

* `input`：
  * 参数变量类型为`VideoNode`
  * 仅支持RGB色彩空间
* `sample`：
  * 参数变量类型为`int`
  * 设定输出采样类型，类似`bool`型参数
    * `sample`=0：输出16bit整型（默认）
    * `sample`=1：输出32bit浮点型

#### `OPP2RGB()`：

将OPP色彩空间转换至RGB色彩空间。

**参数一览：**

`bm3d.OPP2RGB(clip input[, int sample=0])`

**参数解释：**

* `input`：
  * 参数变量类型为`VideoNode`
  * 仅支持YUV色彩空间
  * 如前文所述，OPP为YUV的一种，输入通常意义的YUV不报错但颜色不正确；为使结果有意义，应当仅以OPP为输入
* `sample`：
  * 参数变量类型为`int`
  * 设定输出采样类型，类似`bool`型参数
    * `sample`=0：输出16bit整型（默认）
    * `sample`=1：输出32bit浮点型

### BM3D核心处理函数：

#### `Basic()`：

基础降噪函数，BM3D核心处理函数之一，其输出结果可以作为BM3D另一核心处理函数`bm3d.Final()`的参考源，也可直接作为降噪结果使用。

**参数一览：**

`bm3d.Basic(clip input[, clip ref=input, string profile="fast", float[] sigma=[10,10,10], int block_size, int block_step, int group_size, int bm_range, int bm_step, float th_mse, float hard_thr, int matrix=2])`

**参数解释：**

* `input`：
  * 参数变量类型为`VideoNode`
  * 输出结果与输入源保持相同格式
* `ref`：
  * 参数变量类型为`VideoNode`
  * 参考源，用于块匹配
  * 若未设定该参数，则以输入源为参考源
* `profile`：
  * 参数变量类型为`string`
  * 设定预设模式，可以对后续其他参数预设不同的默认值，详见下文表格
  * 各预设模式的含义如下：
    * `fast`：快速模式（默认）
    * `lc`：低复杂度模式
    * `np`：正常模式
    * `high`：高强度模式
    * `vn`：超强噪点模式
* `sigma`：
  * 参数变量类型为`float`数组
  * 设定各平面的降噪强度，单个平面范围为`[0, +inf)`，默认值为`[10, 10, 10]`
  * BM3D算法对`sigma`参数非常敏感，应当认真根据输入源调整该参数
  * 该参数源自高斯白噪声模型
  * 当某一平面参数为0时，不处理该平面
* `block_size`：
  * 参数变量类型为`int`
  * 设定块大小，块大小为`block_size*block_size`，范围为`[1, 64]`
  * 块为BM3D的基本处理单元，越大的块速度越慢，但块越大则可设定更大的步长（`block_size`），从而减少要处理的块
  * 在不同预设模式下，默认值均为8，兼顾速度和质量
* `block_step`：
  * 参数变量类型为`int`
  * 设定两个参考块间的距离，即步长，范围为`[1, block_size]`
  * 步长越小，在处理中会使用更多的参考块，速度也越慢
* `group_size`：
  * 参数变量类型为`int`
  * 设定每个组（group）中相似块的最大数量，范围为`[1, block_size]`
  * 参数值越大，在一个group中包含的块越多，则变换后group的稀疏程度越大，降噪效果越强，速度也越慢
  * 当`group_size`设为1时，不进行块匹配，此时进行的降噪类似无时域处理的DFTTest降噪
* `bm_range`：
  * 参数变量类型为`int`
  * 设定块匹配搜索的范围，范围为`[1, +inf)`
  * 参数值越大，速度越慢，寻找到相似块的概率越大
* `bm_step`：
  * 参数变量类型为`int`
  * 设定块匹配搜索中两搜索位置间的距离，范围为`[1, bm_range]`
  * 参数值越小，速度越慢
  * 当`bm_step`设为1时，进行逐个搜索
* `th_mse`：
  * 参数变量类型为`float`
  * 设定块匹配中均方误差的阈值，范围为`[0, +inf)`
  * 参数值越大，会有更多的块被匹配；参数值过大会导致一个group中出现低相似度的块，进而导致纹理和细节损失
  * 若输入源噪点很重，应增大该参数值，默认值会根据`sigma[0]`自行调整
* `hard_thr`：
  * 参数变量类型为`float`
  * 设定频域滤波的硬阈值（hard-thresholding），范围为`(0, +inf)`
  * 参数值越大，频域中的硬阈值滤波越强
  * 通常而言，若想降低降噪强度，调整`sigma`参数要好于调整`hard_thr`参数
* `matrix`：
  * 参数变量类型为`int`
  * 为灰度、YUV、YCoCg输入源设定系数矩阵，默认值为2
  * 由于YUV色彩空间未进行归一化，因此在BM3D内使用的`sigma`需要根据系数矩阵进行归一化，这非常重要，会影响结果的准确性
  * 基于ISO/IEC 14496-10标准，不同参数值对应的系数矩阵如下
    * 0：`GRB`
    * 1：`bt709`
    * 2：未指定，根据输入源的尺寸在`smpte170m`、`bt709`、`bt2020nc`中选择
    * 4：`fcc`
    * 5：`bt470bg`
    * 6：`smpte170m`
    * 7：`smpte240m`
    * 8：`YCgCo`（在色彩空间为YCgCo时应使用该项）
    * 9：`bt2020nc`
    * 10：`bt2020c`
    * 100：`OPP`（在使用`bm3d.RGB2OPP()`函数将RGB色彩空间转换为OPP色彩空间后，应使用该项）

#### `Final()`：

最终降噪函数，BM3D核心处理函数之二，需要使用`bm3d.Basic()`的处理结果作为参考源。

**参数一览：**

`bm3d.Final(clip input, clip ref[, string profile="fast", float[] sigma=[10,10,10], int block_size, int block_step, int group_size, int bm_range, int bm_step, float th_mse, int matrix=2])`

**参数解释：**

* `input`：
  * 参数变量类型为`VideoNode`
  * 输出结果与输入源保持相同格式
* `ref`：
  * 参数变量类型为`VideoNode`
  * 参考源，用于块匹配，并作为维纳滤波的参考
  * 该项参数必须设定，按照原始BM3D算法，参考源应设定为`bm3d.Basic()`的处理结果
  * 该项参数也可设定为其他任何恰当的降噪结果，并通过本函数进行精细处理
* `profile`、`sigma`、`block_size`、`block_step`、`group_size`、`bm_range`、`bm_step`、`th_mse`、`matrix`：
  * 与`bm3d.Basic()`相应参数含义相同

### V-BM3D核心处理函数：

V-BM3D在BM3D的基础上增加了时域处理。具体而言，在块匹配和协同滤波时，不仅会使用当前帧，还会使用前后多个帧。

由于处理结果被返回到多个帧当中，因此必须将处理过程分为两个阶段。第一阶段可以称为核心处理，类似BM3D，使用`bm3d.VBasic()`/`bm3d.VFinal()`，获得中间数据，格式为32bit浮点数，高度扩大为输入源的`(radius * 2 + 1) * 2`倍；第二阶段可以称为后处理，使用`bm3d.VAggegrate()`函数，Aggegrate意为聚合，将中间数据转换为正常图像。

由于处理中使用了多个帧，且中间数据使用32bit浮点数和倍增的尺寸，V-BM3D内存占用很大。

#### `VBasic()`：

基础降噪函数，V-BM3D核心处理函数之一，与`bm3d.Basic()`类似，增加了时域处理。

**参数一览：**

`bm3d.VBasic(clip input[, clip ref=input, string profile="fast", float[] sigma=[10,10,10], int radius, int block_size, int block_step, int group_size, int bm_range, int bm_step, int ps_num, int ps_range, int ps_step, float th_mse, float hard_thr, int matrix=2])`

**参数解释：**

* `input`、`ref`：
  * 与`bm3d.Basic()`相应参数含义相同
* `profile`、`sigma`、`block_size`、`block_step`、`group_size`、`bm_range`、`bm_step`、`th_mse`、`matrix`：
  * 与`bm3d.Basic()`相应参数含义相同
* `radius`：
  * 参数变量类型为`int`
  * 设定降噪的时间半径，范围为`[1, 16]`
  * 增大时间半径仅会给块匹配与聚合过程增加微小的计算量，不会影响协同滤波，但成倍增加内存占用
  * 只要内存足够用，这个参数就可以往大了调
* `ps_num`：
  * 参数变量类型为`int`
  * 设定预测搜索时匹配位置的数量，范围为`[1, group_size]`
  * 增大该参数值可以提高匹配到相似块的可能性，而且仅需增加很小的计算成本；但在原始的MATLAB实现中，除“lc”外，其余所有配置文件均将该参数固定为2，也许过大的数值对质量不总是有利的？
* `ps_range`：
  * 参数变量类型为`int`
  * 设定预测搜索块匹配的范围，范围为`[1, +inf)`
* `ps_step`：
  * 参数变量类型为`int`
  * 设定预测搜索块匹配中两搜索位置间的距离，范围为`[1, ps_range]`
  * 单个帧中各参考块的预测搜索位置最大数量为`(ps_range / ps_step * 2 + 1) ^ 2 * ps_num`

#### `VFinal()`：

最终降噪函数，V-BM3D核心处理函数之二，与`bm3d.Final()`类似，增加了时域处理。

**参数一览：**

`bm3d.VFinal(clip input, clip ref[, string profile="fast", float[] sigma=[10,10,10], int radius, int block_size, int block_step, int group_size, int bm_range, int bm_step, int ps_num, int ps_range, int ps_step, float th_mse, int matrix=2])`

**参数解释：**

* `input`、`ref`：
  * 与`bm3d.Final()`相应参数含义相同
* `profile`、`sigma`、`block_size`、`block_step`、`group_size`、`bm_range`、`bm_step`、`th_mse`、`matrix`：
  * 与`bm3d.Basic()`相应参数含义相同
* `radius`、`ps_num`、`ps_range`、`ps_step`：
  * 与`bm3d.VBasic()`相应参数含义相同

### V-BM3D后处理函数：

#### `VAggregate()`：

在调用`bm3d.VBasic()`和`bm3d.VFinal()`函数后，必须调用该函数，以完成聚合（Aggregation）。

如果以RGB格式为`bm3d.VBasic()`或`bm3d.VFinal()`函数的输入，需要在调用该函数后，再调用`bm3d.OPP2RGB()`。

**参数一览：**

`bm3d.VAggregate(clip input[, int radius=1, int sample=0])`

**参数解释：**

* `input`：
  * 参数变量类型为`VideoNode`
  * 输入源应为`bm3d.VBasic()`或`bm3d.VFinal()`函数的输出结果
* `radius`：
  * 参数变量类型为`int`
  * 必须与`bm3d.VBasic()`或`bm3d.VFinal()`设定的`radius`相同
* `sample`：
  * 参数变量类型为`int`
  * 设定输出采样类型，类似`bool`型参数
    * `sample=0`：输出16bit整型（默认）
    * `sample=1`：输出32bit浮点型

### 各预设模式的默认参数：

在BM3D滤镜的四个核心处理函数`bm3d.Basic()`、`bm3d.Final()`、`bm3d.VBasic()`、`bm3d.VFinal()`中，通过设定`profile`参数可以选择不同的预设模式，这些预设模式对函数其他参数给定了不同的默认值，详见下面三个表格

```text
bm3d.Basic / bm3d.Final / bm3d.VBasic / bm3d.VFinal
----------------------------------------------------------------------------
| profile || block_size | block_step | group_size  | bm_range    | bm_step |
----------------------------------------------------------------------------
| "fast"  || 8/8/8/8    | 8/7/8/7    | 8/8/8/8     | 9/9/7/7     | 1/1/1/1 |
| "lc"    || 8/8/8/8    | 6/5/6/5    | 16/16/8/8   | 9/9/9/9     | 1/1/1/1 |
| "np"    || 8/8/8/8    | 4/3/4/3    | 16/32/8/8   | 16/16/12/12 | 1/1/1/1 |
| "high"  || 8/8/8/8    | 3/2/3/2    | 16/32/8/8   | 16/16/16/16 | 1/1/1/1 |
| "vn"    || 8/11/8/11  | 4/6/4/6    | 32/32/16/16 | 16/16/12/12 | 1/1/1/1 |
----------------------------------------------------------------------------
```

```text
bm3d.VBasic / bm3d.VFinal
---------------------------------------------------
| profile || radius | ps_num | ps_range | ps_step |
---------------------------------------------------
| "fast"  || 1/1    | 2/2    | 4/5      | 1/1/1/1 |
| "lc"    || 2/2    | 2/2    | 4/5      | 1/1/1/1 |
| "np"    || 3/3    | 2/2    | 5/6      | 1/1/1/1 |
| "high"  || 4/4    | 2/2    | 7/8      | 1/1/1/1 |
| "vn"    || 4/4    | 2/2    | 5/6      | 1/1/1/1 |
---------------------------------------------------
```

```text
bm3d.Basic & bm3d.VBasic / bm3d.Final & bm3d.VFinal
--------------------------------------------------------------
| profile || th_mse                              | hard_thr  |
--------------------------------------------------------------
| "fast"  || sigma[0]*80+400   / sigma[0]*10+200 | 2.7 / NUL |
| "lc"    || sigma[0]*80+400   / sigma[0]*10+200 | 2.7 / NUL |
| "np"    || sigma[0]*80+400   / sigma[0]*10+200 | 2.7 / NUL |
| "high"  || sigma[0]*80+400   / sigma[0]*10+200 | 2.7 / NUL |
| "vn"    || sigma[0]*150+1000 / sigma[0]*40+400 | 2.8 / NUL |
--------------------------------------------------------------
```

## 使用贴士：

### 直接使用BM3D滤镜：

综合前文的介绍，在使用BM3D滤镜时往往要综合调用多个函数，下面给出一些示例。

#### 使用BM3D系列函数：

* 输入源为YUV格式，仅使用基础降噪函数进行降噪，并对Y、U、V平面设定不同的`sigma`值

  ```python
  flt = core.bm3d.Basic(src, sigma=[10,6,8])
  ```

* 输入源为YUV格式，使用基础降噪函数+最终降噪函数进行降噪，并对U、V平面设定相同的`sigma`值

  ```python
  ref = core.bm3d.Basic(src, sigma=[10,7])
  flt = core.bm3d.Final(src, ref, sigma=[10,7])
  ```

* 输入源为YUV格式，引入额外的降噪滤镜作为预处理，将其降噪结果作为基础降噪函数的参考源，并对Y、U、V平面设定相同的`sigma`值

  ```python
  pre = haf.sbr(src, 3)
  ref = core.bm3d.Basic(src, pre, sigma=7)
  flt = core.bm3d.Final(src, ref, sigma=7)
  ```

* 输入源为RGB格式，先转换为OPP格式，在BM3D滤镜内部使用OPP格式降噪，并转回RGB输出

  ```python
  src = core.bm3d.RGB2OPP(src)
  ref = core.bm3d.Basic(src, matrix=100)
  flt = core.bm3d.Final(src, ref, matrix=100)
  flt = core.bm3d.OPP2RGB(flt)
  ```

#### 使用V-BM3D系列函数：

再次提醒，使用V-BM3D系列函数，必须在降噪函数`bm3d.VBasic()`和`bm3d.VFinal()`后使用`bm3d.VAggregate()`函数。

* 输入源为RGB格式，先转换为OPP格式，在BM3D滤镜内部使用OPP格式降噪，并转回RGB输出

  ```python
  src = core.bm3d.RGB2OPP(src)
  ref = core.bm3d.VBasic(src, radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.bm3d.VFinal(src, ref, radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.bm3d.OPP2RGB(flt)
  ```

* 输入源为RGB格式，若在转换成OPP格式后，不对色度平面进行处理，需要使用`std.ShufflePlanes()`函数将未处理的色度平面与BM3D的降噪结果合并

  ```python
  src = core.bm3d.RGB2OPP(src)
  ref = core.bm3d.VBasic(src, sigma=[10,0,0], radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.bm3d.VFinal(src, ref, sigma=[10,0,0], radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.std.ShufflePlanes([flt,src,src], [0,1,2], vs.YUV)
  flt = core.bm3d.OPP2RGB(flt)
  ```

* 对于上一个例子，更好地做法是在灰度空间下降噪，可以大幅降低内存占用

  ```python
  src = core.bm3d.RGB2OPP(src)
  srcGray = core.std.ShufflePlanes(src, 0, vs.GRAY)
  ref = core.bm3d.VBasic(srcGray, sigma=[10,0,0], radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.bm3d.VFinal(srcGray, ref, sigma=[10,0,0], radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.std.ShufflePlanes([flt,src,src], [0,1,2], vs.YUV)
  flt = core.bm3d.OPP2RGB(flt)
  ```

* 输入源为RGB格式，若在转换成OPP格式后，不对亮度平面进行处理，则由于块匹配基于`ref`的亮度平面，对`ref`也需要使用`std.ShufflePlanes()`进行合并

  ```python
  src = core.bm3d.RGB2OPP(src)
  ref = core.bm3d.VBasic(src, sigma=[0,10,10], radius=1, matrix=100).bm3d.VAggregate(radius=1)
  ref = core.std.ShufflePlanes([src,ref,ref], [0,1,2], vs.YUV)
  flt = core.bm3d.VFinal(src, ref, sigma=[0,10,10], radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.std.ShufflePlanes([src,flt,flt], [0,1,2], vs.YUV)
  flt = core.bm3d.OPP2RGB(flt)
  ```

* 对于上一个例子，也可使用`bm3d.Basic()`代替`bm3d.VBasic()`，更快、更省内存

  ```python
  src = core.bm3d.RGB2OPP(src)
  ref = core.bm3d.Basic(src, matrix=100)
  flt = core.bm3d.VFinal(src, ref, radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.bm3d.OPP2RGB(flt)
  ```

* 使用自定义降噪滤镜作为基础降噪结果，并使用V-BM3D的最终降噪函数做精细化处理

  这样做可以在两种降噪滤镜间取长补短。在下面的例子中，SMDegrain滤镜在时空平滑上很有效，但可能导致鬼影（blending）和细节损失，V-BM3D可以很好地保留细节，但对于大的噪声pattern（例如粗颗粒）效果不好

  ```python
  src = core.bm3d.RGB2OPP(src)
  ref = haf.SMDegrain(src)
  flt = core.bm3d.VFinal(src, ref, radius=1, matrix=100).bm3d.VAggregate(radius=1)
  flt = core.bm3d.OPP2RGB(flt)
  ```

### 通过`mvsfunc`脚本使用：

为简化函数调用，`mvsfunc`脚本中的`BM3D()`提供了开箱即用的支持，仅需调用单个`BM3D()`函数即可使用BM3D滤镜进行降噪。

关于`BM3D()`函数的详细说明可以移步[脚本百科 - mvsfunc](https://guide.geeking.moe/lv-jing-bai-ke/jiao-ben-bai-ke/mvsfunc)部分，下面举两个简单的例子。

```python
import mvsfunc as mvf

# src可以为任何格式

# 示例一
flt = mvf.BM3D(src, sigma=3.0, profile1="fast")

# 示例二
flt = mvf.BM3D(src, sigma=3.0, radius1=1, profile1="fast")
```

在上述两个例子中，各使用一行代码，完成了BM3D滤镜的调用。示例一没有设定`radius1`参数，表示使用BM3D系列函数进行降噪；示例二设定了`radius1`参数，表示使用V-BM3D系列函数进行降噪。

## 扩展阅读：

BM3D算法于2007年由芬兰坦佩雷理工大学的Kostadin Dabov等人提出，全称为Block-matching and 3D filtering，可译为三维块匹配算法。BM3D算法的原始论文为**Image Denoising by Sparse 3-D Transform-Domain Collaborative Filtering,** _**IEEE Transactions on image processing 16.8 \(2007\): 2080-2095**_，我们来简要了解一下这篇论文。

论文主体包括概念说明、算法思路、快速算法、从灰度到RGB的扩展、降噪效果这几个部分。我们仅简要介绍算法思路，并说明相关概念，想了解其余部分可阅读原始论文。

### 算法思路：

如上面的教程所述，BM3D算法分为两个步骤，基础估计（Basic estimate）和最终估计（Final estimate），具体过程如下。

* **基础估计**
  * **块估计**

    对含噪点图像的每个块进行如下操作

    * **分组**：进行块匹配，在一定范围内寻找相似的块，将它们存储到一个三维数组中，称为组（group）
    * **协同硬阈值滤波**（Collaborative hard-thresholding）：对上述group进行三维变换，在频域下进行硬阈值滤波，然后进行逆变换还原，并将块复原至原始位置

  * **聚合**（Aggregation）

    对所有重叠的估计块进行加权平均，计算得到真实图像的基础估计结果（即基础降噪结果）
* **最终估计**
  * **块估计**

    对输入图像的每个块进行如下操作

    * **分组**：对基础估计结果进行块匹配，在一定范围内确定相似块的位置，基于这些位置生成两个group，一个group来自含噪点图像，另一个group来自基础估计结果
    * **协同维纳滤波**（Collaborative Wiener filtering）：对上述两个group进行三维变换，基于基础估计结果，对含噪点的group进行维纳滤波，然后进行逆变换还原，并将块复原至原始位置

  * **聚合**

    对得到的估计块进行加权平均，计算得到真实图像的最终估计结果（即最终降噪结果）

上述过程可以如下图描述。

![BM3D&#x7B97;&#x6CD5;&#x793A;&#x610F;&#x56FE;](https://i.loli.net/2020/01/29/EY1KbZtwe7p2kAB.jpg)

### 概念解释：

按照论文行文的顺序，肯定要先说明概念，再进行上文的算法介绍。但这里为了防止突然扔出一堆概念，导致大家睡着，所以把概念解释放到了后面。

* **匹配**（Matching）：匹配在这里指基于参考块寻找相似块的方法，运算量较低。**通过匹配寻找相似块，即块匹配，英文为Block Matching，这便是BM3D中“BM”的由来**。
* **分组**（Grouping）：将图像切块，并将相似的块分为一组（group）。**一个块可用二维数组表示，那么一组块就要用三维数组表示，这便是BM3D中“3D”的由来**。在BM3D算法中，分组当然是通过块匹配实现。分组也可以基于K均值聚类、自组织映射、模糊聚类、向量量化等无监督学习算法，通过这些算法计算相似度，根据相似度进行分组。但与块匹配相比，这些算法的运算量较大。
* **硬阈值**（Hard-thresholding）：在变换域下，真实图像与噪点的系数存在差别，真实图像系数较大，噪点系数较小。可以基于这种差别进行降噪。直接的方法便是设定阈值，将真实图像与噪点区分开来，故称硬阈值。在BM3D算法的基础估计部分，通过硬阈值滤波进行降噪，得到基础估计结果。
* **收缩**（Shrinkage）：在变换域下降噪，可以对系数较小的部分做收缩，能使降噪的结果更接近真实图像。在BM3D算法的最终估计部分，基于基础估计结果得到经验维纳收缩系数（empirical Wiener shrinkage coefficients），进行维纳滤波，得到最终估计结果。
* **协同滤波**（Collaborative Filtering）：“协同”可以从字面意义上理解，指一个group内的每个块都协作进行对其他块的滤波。在BM3D算法中，协同滤波通过变换域下的收缩实现，包括协同硬阈值滤波和协同维纳滤波。

