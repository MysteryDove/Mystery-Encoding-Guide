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
  * 支持8-16bit整型，不支持32bit浮点型
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
  * 当`nrmode`≤ 0时，对输入源做均值滤波；当`nrmode`&gt;0时，对输入源做`sbr()`降噪，参数含义与`sbr()`函数的`r`相同
  * 当参数`p`非默认（即手工指定了空域降噪结果）时，该参数无效
* `radius`：
  * 参数变量类型为`int`
  * 参数范围为`1,2,3`
  * 该项参数设定Temporal radius，以控制时域降噪的力度，数值越大则降噪力度越大、速度越慢
* `adapt`：
  * 参数变量类型为`int`
  * 该项参数用来控制后处理时的亮度mask
  * 当`adapt`≤ -1且未指定`Lmask`时，函数不做后续的亮度自适应限制处理
* `rep`：
  * 参数变量类型为`int`
  * 该项参数用来设定后处理时`Repair()`的模式
  * 当`rep`≤ 0时，函数不做`Repair()`；当`rep`&gt;0时，参数含义与`Repair()`函数的`mode`相同
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

