# 编写你的第一个Vapoursynth脚本

## 先贴一段最简单的输入输出，我们开始吧！

```text
import vapoursynth as vs
core = vs.get_core()
core.max_cache_size = 2000

input = r'F:\Resource workspace\00010.mp4'

src8 = core.lsmas.LWLibavSource(input)
src8.set_output()
```

## 从熟悉Vapoursynth编写语法开始

其实Vapoursynth的语法与Python非常相似，VS脚本本质上都是Python脚本，语法都是根据Python来的，所以可以看到一般开头，都是一串import，载入必要的vs和常用的library，这里只是最简单的示例，所以仅载入了vs库。

首先我们要通过这两行代码来载入vs的内核，并且指定好期望VS使用的内存容量，在执行`vs.get_core()`后，vs会自动载入所有第三方的dll滤镜，这些滤镜在portable环境中存放在”vapoursynth64\plugins“中。

```text
core = vs.get_core()
core.max_cache_size = 2000
```

这时候我们就可以尝试调用滤镜了，我们这里仅用到了最基础的”源滤镜“，他们负责读取你提供的文件路径字符串（这里是input），并且对对应文件进行索引，最后将读取的结果丢给变量名。（不要有人再找我问什么字符串前面那个 **r** 是什么意思，为什么源滤镜报错了）

```text
input = r'F:\Resource workspace\00010.mp4'
src8 = core.lsmas.LWLibavSource(input)
```

毕竟使用的是Python，所以弱变量类型让你可以把任何东西都丢进一个变量里，一般源滤镜读出来的结果是一个[VideoNode](http://www.vapoursynth.com/doc/pythonreference.html?highlight=videonode#VideoNode)类（点击进去可以看到VideoNode类里有哪些方法可以调用）

在调用滤镜时，你应该遵循以下的调用规则：

```text
core.namespace.function()
```

每个滤镜一般都有自己的namespace，在运行`core = vs.get_core()`时，vs会自动加载插件目录下所有滤镜，并且把他们的namespace读出来，而在这些namespace下，有不同的function。

比如举例的源滤镜使用的vslsmashsource.dll，其定义的namespace是`lsmas`，在这个namespace下有个函数叫做`LWLibavSource()`，所以调用方式就是`core.lsmas.LWLibavSource()`

最后我们调用我们已经承接上读出的视频源的变量src8（它是一个VideoNode类！）的一个函数：`set_output()`

```text
src8.set_output()
```

我们就能输出我们刚才读入的成品啦！

## 等等，我咋预览我刚才写的脚本？

嘿，说到点子上了！这时候你就需要一个类似IDE一样的东西来帮助你了，我们来谈谈VSEdit吧！

首先我们打开软件，把刚才的代码丢进去，按一下f6

![VSEdit&#x4E3B;&#x754C;&#x9762;](https://i.v2ex.co/X178n2TWl.png)

如果没有什么大问题，你应该可以看到一条绿色的成功运行提示，就说明你的脚本基本上没什么问题，可以正常输出了。而下面一条提示告诉你输出的视频总共有多少帧，时长多少，分辨率，帧率以及输出格式等信息，这些信息能帮助你检查自己有没有弄错什么东西。

如果有，就请先检查一下你的路径到底对不对，字符串的 "\" 有没有转义，如果都对了还不行那就只能各凭本事咯\(｀･ω･\)

如果没有，那再按一下f5，你应该能看到弹出来了一个预览界面，下面还有一个进度条可以拖动，那个界面现在就不介绍那么多了，反正你只要知道可以拖着玩就是了。（什么，你说没有声音？VS是处理视频的帧服务器怎么可能会有声音呢？⊂彡☆\)\)д\`\)）

