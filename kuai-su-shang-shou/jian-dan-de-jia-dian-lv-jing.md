# 简单的……加点滤镜

既然我们已经可以输入输出了，那我们何不在之前的脚本基础上，再加点料呢？

## 年轻人的第一个滤镜

什么，居然是年轻人的第一个滤镜？你说的是小。。。⊂彡☆\)\)д\`\)小个头啦

这里就拿最简单的字幕滤镜来说好了，我们调用Vapoursynth自带的`sub.TextFile()`来做演示好了

基于上一章我们的输入输出脚本，我们只要在输出行前一行加一句：

```text
src8 = core.sub.TextFile(src8, r'F:\sub.ass')
```

如果你的字幕文件本身没什么问题，字幕文件的路径也没什么问题的话，你在vsedit使用f6/f5应该也不会遇到什么问题，如果有问题就……自裁，请\(\|\|\|ﾟдﾟ\)。

需要一提的是，这些滤镜函数的调用和参数传递的方式是需要注意的。比如你使用vsedit输入上述滤镜的过程中，在打到`sub.Te`的时候就会弹出自动补全的选项，按一下tab，vsedit便会把这函数所有调用方式全都写出来，需要你一个一个去填，我们还是拿`sub.TextFile()`举例子  
  
`core.sub.TextFile(clip, file, charset, scale, debuglevel, fontdir, linespacing, margins, sar, style, blend, matrix, matrix_s, transfer, transfer_s, primaries, primaries_s)`

一个函数下有非常非常多的参数可以自己传递，但是对于字幕滤镜来说，你只需要`clip,file` 这两个参数就够了，其他的参数都**可以不写**，而参数传递的方式则有两种

```text
#这两种写法是等价的
src8 = core.sub.TextFile(src8, r'F:\sub.ass')    #使用位置传递
src8 = core.sub.TextFile(clip=src8, file=r'F:\sub.ass')    #使用按关键字参数传递
```

如果你使用按位置传递，你需要知道哪个值给哪个变量，这些参数的具体顺序是什么样的，这适合一些简单的滤镜或者你已经是老司机非常熟悉你用的滤镜的情况。

而按关键字传递，则更加适合新手上路以及使用陌生的滤镜，这样能降低出错的机率，它可以很清楚的让你知道，哪个值给了哪个参数变量，也可以强迫你去看滤镜的文档，知道每个参数变量到底是用来干啥的，这个取值是不是合理。

当然了，这两种方法是可以混搭的，但是直接传递必须在前面，一个个按照顺序排列，然后才是关键字传递，我们拿另一个滤镜来举例子（虽然有些超纲，但是领会意思就好）

```text
#这三种写法也是等价的

#这是完全按照关键词传递
core.std.MaskedMerge(clipa=nonedge, clipb=edge, mask=mask, planes=[0,1,2], first_plane=False)
#这是完全按照位置直接传递
core.std.MaskedMerge(nonedge, edge, mask, [0,1,2], False)
#这是混搭流写法
core.std.MaskedMerge(nonedge, edge, mask, planes=[0,1,2], first_plane=False)
```

当然了，大部分滤镜第一个位置都是留给视频变量的，所以第一位的clip一般也不用指定，**但我推荐尽量使用更多关键字传递方法来写滤镜**，谁也不希望自己写的东西过一个月后来看就完全看不懂了，而且随着滤镜的更新，参数的位置可能会有变化，有些时候你的脚本丢给别人甚至会跑不起来（说的就是你，TCanny）。

尽管VS脚本写起来有很多自由，但是养成好的习惯是必要的，总结总结下来就是两点

1. 按顺序
2. 尽量用关键字传递

## 年轻人的第N个滤镜

既然知道怎么添加第一个，那就知道怎么添加第二个，第三个……第N个，如果你认真搞懂了上面的部分的话，那再加个降噪/去色带也不是问题吧。

不是问题？如果只是刚入门的话，确实还是有些小问题的！

降噪滤镜和去色带滤镜，需要在更高的色彩位深（bitdepth）下才能发挥它们全部的实力，如果仅仅在8bit下使用它们，效果会大打折扣，而参数的调整和如何最大程度减轻滤镜对画面的破坏这类细节问题也是成为优秀压制者必须知道的。

如果想要进阶，成为优秀的压制者，大胆的向“从入门到入坟”进发吧！

光说不练假把式，我们从引入第一个脚本库——mvsfunc开始简单的说起吧：

`import mvsfunc as mvf` 

我们在这不深究mvsfunc里到底有啥函数，为啥要import成mvf，你乐意import成啥就是啥，在这里我们就只用它的一个函数`Depth()`来示范

```text
import vapoursynth as vs
import mvsfunc as mvf
core = vs.get_core()
core.max_cache_size = 4000    #什么？怎么从2000变成4000了？滤镜用多了多给点内存肯定没错！

input = r'F:\233.mp4'
src8 = core.lsmas.LWLibavSource(input)
src16 = mvf.Depth(src8,16)    #这是啥？这是我们用mvf来把8bit输入源转换成16bit，你不信？可以试试src16.set_output()
nr = core.knlm.KNLMeansCL(src16, d=1, a=2, s=4, h=1.5, wmode="3", device_type="GPU", device_id="0")
db = core.f3kdb.Deband(nr, 12, 48, 32, 32, 0, 0, output_depth=16)
down8 = mvf.Depth(db,8)       #最后我们把16bit处理完的clip重新缩回8bit来加字幕和输出
out = core.sub.TextFile(down8,r'F:\subtitle.ass')
out.set_output()
```

其中的KNLMeancCL是需要opencl运行环境的降噪滤镜，如果你是笔记本，可以试着把device\_id改成1来使用独显。f3kdb是非常常用的去色带滤镜，其中的参数我们暂时不具体讨论。

到这里，我们已经编写出了最基本的“读入-降噪/去色带-加字幕-输出”脚本，接下来我们就要使用编码器来接收我们的输出，并且编码成成品视频了！



