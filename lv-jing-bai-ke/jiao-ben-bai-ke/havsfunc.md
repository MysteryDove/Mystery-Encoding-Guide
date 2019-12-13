# havsfunc

havsfunc是由HolyWu写的脚本，将Avisynth的一些函数迁移到VapourSynth中

## 脚本依赖：

havsfunc需要的脚本有：

* `adjust`
* `mvsfunc`
* `nnedi3_resample`

havsfunc需要的滤镜有：

* `AddGrain`
* `CTMF`
* `DCTFilter`
* `Deblock`
* `DFTTest`
* `EEDI2`
* `EEDI3`
* `FFT3DFilter`
* `FluxSmooth`
* `fmtconv`
* `Hqdn3d`
* `KNLMeansCL`
* `MVTools`
* `NNEDI3`
* `NNEDI3CL`
* `SangNom`
* `SVP`
* `TTempSmooth`
* `ZNEDI3`

## 所含函数：

### `GSMC()`：

时域降噪函数，全称为GrainStabilizeMC，降噪力度小、速度快，仅对输入源与空域降噪结果的差值进行时域降噪，并使用`Repair()`函数和亮度自适应mask进行后处理

#### 参数一览：

`GSMC(input, p=None, Lmask=None, nrmode=None, radius=1, adapt=-1, rep=13, planes=[0, 1, 2], thSAD=300, thSADC=None, thSCD1=300, thSCD2=100, limit=None, limitc=None)`

#### 参数解释：

* `input`：
  * 参数变量类型为`VideoNode`
  * 支持8-16bit整型，不支持32bit浮点型，不支持RGB色彩空间
* `p`：
  * 参数变量类型为`VideoNode`
  * 格式需要与`input`保持一致
  * 该项为空域降噪结果，用来与输入源做差，获得时域降噪的对象
  * 默认为对输入源做均值滤波或`sbr()`降噪
* `Lmask`：
  * 参数变量类型为`VideoNode`
  * 该项为亮度mask
* `nrmode`：
  * 参数变量类型为`int`
  * 该项参数设定（作为预处理的）空域降噪模式
  * 当`nrmode`≤ 0时，对输入源做均值滤波；当`nrmode`&gt; 0时，对输入源做`sbr()`降噪，参数含义与`sbr()`函数的`r`相同
  * 当参数`p`非默认（即手工指定了空域降噪结果）时，该参数无效
* `radius`：
  * 参数变量类型为`int`
  * 参数范围为`1,2,3`
  * 该项参数设定Temporal radius，以控制时域降噪的力度，数值越大则降噪力度越大、速度越慢
* `adapt`：
  * 参数变量类型为`int`
  * 该项参数用于控制后处理时的亮度mask
  * 当`adapt`≤ -1且未指定`Lmask`时，函数不做后续的亮度自适应限制处理
* `rep`：
  * 参数变量类型为`int`
  * 该项参数用于设定后处理时`Repair()`的模式
  * 当`rep`≤ 0时，函数不做`Repair()`；当`rep`&gt; 0时，参数含义与`Repair()`函数的`mode`相同
* `planes`：
  * 参数变量类型为`int`
  * 参数范围为`0,1,2`
  * 通用参数，用来选择被处理的平面
* `thSAD`
* `thSADC`
* `thSCD1`
* `thSCD2`
* `limit`
* `limitc`：
  * 以上六个参数为时域处理的通用参数

#### 使用贴士：

相比于其他降噪方法，`GSMC()`的降噪力度小，可以作为降低码率的一种手段

### `FastLineDarkenMOD()`：

用于加深/加黑线条，仅对亮度平面进行处理，速度快，不会对边缘/轮廓造成破坏，同时具有可选的收线功能，但使用收线功能会降低速度

#### 参数一览：

`FastLineDarkenMOD(c, strength=48, protection=5, luma_cap=191, threshold=4, thinning=0)`

#### 参数解释：

* `c`：
  * 参数变量类型为`VideoNode`
  * 支持8-16bit整型和32bit浮点型，不支持RGB色彩空间
* `strength`：
  * 参数变量类型为`int`
  * 参数范围为`0-256`
  * 控制线条加深的程度，数值越大，加深程度越大
* `protection`：
  * 参数变量类型为`int`
  * 参数范围为`0-50`
  * 保护暗场线条，防止暗场线条变得过暗
  * 数值越大，保护程度越大，类似阈值
* `luma_cap`：
  * 参数变量类型为`int`
  * 参数范围为`0-255`
  * 该项参数基于亮度调整线条加深的程度，抑制以下两种问题：加深线条时被周围高亮度像素干扰，对亮场的灰色线条进行不必要的加深
  * 亮度高于`luma_cap`的像素会被视为亮度与`luma_cap`一致，降低该参数会降低线条加深的程度，当`luma_cap`= 255时禁用该功能
* `threshold`：
  * 参数变量类型为`int`
  * 该项参数为阈值，低于阈值的像素不会被处理
  * 当`threshold`= 0时禁用该功能，默认值为4，该数值即为推荐值，能够避免大量随机像素被不必要地加深，当`threshold`过大时会有部分线条未被加深
* `thinning`：
  * 参数变量类型为`int`
  * 参数范围为`0-256`，小于0的数值会被视为0
  * 该项参数用于细化线条/收线，但收线后会使线条周围像素轻微变暗
  * 当`thinning`≤ 0时禁用该功能，禁用该功能可以大幅提高速度，默认即为禁用

#### 使用贴士：

`FastLineDarkenMOD()`可以作为增强目视效果的手段，特别是对于线条柔和、“发虚”的画面，加深线条可较为明显地提升观感，但最好慎重使用，不要过火

