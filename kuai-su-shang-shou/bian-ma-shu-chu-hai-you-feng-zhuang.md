# 编码，输出，还有封装

我们假设你已经有了年轻人的第一个脚本了，是时候把脚本输出成视频流文件了！

## 视频输出及编码

假设我们已经有了上一章那样的脚本，那是时候输出它了，我们在这里介绍两种方式

### 使用vsedit输出：

如果你用vsedit编写了一个脚本，那么可以用它自带的“encode video”功能来编码视频，甚至使用“enqueue encode job”来创建一个队列，批量编码视频。

![encode&#x754C;&#x9762;](https://i.v2ex.co/M1IJz328l.png)

打开了encode界面之后，我们需要选择Header为Y4M，指定编码器可执行文件的位置，然后填写下面的Arguments，至于这个Arguments里面东西是啥，我们在下面解释，这里就直接贴出来方便你们复制好了：

`--demuxer y4m --preset veryslow --crf 21 --threads 16 --deblock -1:-1 --keyint 240 --min-keyint 1 --bframes 8 --ref 6 --qcomp 0.70 --rc-lookahead 70 --aq-strength 0.7 --me tesa --psy-rd 0.6:0.15 --no-fast-pskip --colormatrix bt709 --aq-mode 3 --merange 37 --input-depth 8 -o "output.mp4" -`

确认上面要设置的东西无误之后，直接按start，然后等着就好了，最后会在你的vsedit文件夹下输出一个output.mp4文件

### 使用CLI输出

尽管vsedit使用非常方便，但是总有些小问题非常恼人，所以在CLI上操作仍然是大部分人的选择，假设你已经把脚本保存到和portable版VS一起的目录（也就是vspipe.exe所在的目录，当然，x264也得在这目录里，不然你得手动指定你放x264那个目录下的这个可执行文件），我们在这个目录打开终端（注意一定要使用cmd，powershell是不支持pipeline的），然后输入：

`vspipe --y4m script.vpy - | x264 --demuxer y4m --preset veryslow --crf 21 --threads 16 --deblock -1:-1 --keyint 240 --min-keyint 1 --bframes 8 --ref 6 --qcomp 0.70 --rc-lookahead 70 --aq-strength 0.7 --me tesa --psy-rd 0.6:0.15 --no-fast-pskip --colormatrix bt709 --aq-mode 3 --merange 37 --input-depth 8 -o "output.mp4" -`

按下回车，不出意外就应该开始编码了！（不要忘记”\|“前的一个”-“与最后一个”-“）

我们来看看这一串命令到底干了些什么吧！

首先是vspipe部分的`--y4m`与x264的`--demuxer y4m`，他们代表了分别是vspipe输出y4m格式的视频流以及x264使用y4m解析器来解析vspipe输出的视频流。

其次就是x264自己的一大堆编码参数，这一部分你可以到本教程编码器部分来详细了解每一个参数的作用。

最后`-o "output.mp4"`则指定了输出文件的名

#### 如果要使用VBR 2Pass指定码率呢？

很简单，运行这两行就完事儿了

`vspipe --y4m script.vpy - | x264 --demuxer y4m --preset veryslow --pass 1 --bitrate 5000 --threads 16 --deblock -1:-1 --keyint 240 --min-keyint 1 --bframes 8 --ref 6 --qcomp 0.70 --rc-lookahead 70 --aq-strength 0.7 --me tesa --psy-rd 0.6:0.15 --no-fast-pskip --colormatrix bt709 --aq-mode 3 --merange 37 --input-depth 8 -o "output.mp4" -`

`vspipe --y4m script.vpy - | x264 --demuxer y4m --preset veryslow --pass 2 --bitrate 5000 --threads 16 --deblock -1:-1 --keyint 240 --min-keyint 1 --bframes 8 --ref 6 --qcomp 0.70 --rc-lookahead 70 --aq-strength 0.7 --me tesa --psy-rd 0.6:0.15 --no-fast-pskip --colormatrix bt709 --aq-mode 3 --merange 37 --input-depth 8 -o "output.mp4" -`

可以注意到，其实这两遍唯一的区别就是`--pass1 --pass 2` 

在跑2pass方面， 直接用CLI还是比VSEdit方便的。

### 我应该如何选择编码模式？

目前我们所能接触到的视频编码器有数种码率控制模式，大致有：CRF\(恒定质量模式\)、VBR\(可变码率模式\)、VBR 2Pass\(二次编码的可变码率模式\)，这些模式也各有优劣，下表将浅谈他们的优缺点：

| 名称 | CRF | VBR 1Pass | VBR 2Pass |
| :--- | :--- | :--- | :--- |
| 优点 | 速度快，质量好 | 可以准确控制体积 | 可以准确控制体积 |
| 缺点 | 不能准确控制体积 | 除了上面一个优点之外都是缺点 | 很慢，码率不到位还会翻车 |

如果没有严格的体积要求的话，crf永远是最好的选择，它会根据视频内容及画面复杂度动态调整码率分配，但是相同的crf下，复杂的视频（纹理丰富，动态激烈）体积会大很多（相对的高动态场景不会被编码器糊成一片）。而如果你想要一个特定码率的视频（比如传b站避免二压），那么就应该选择VBR模式，VBR 1Pass尽管速度快，但是码率分配非常不合理，经常出现烂帧现象，除非你的电脑是小霸王，不然尽量使用2Pass来进行编码。

## 音频抽取与编码

经过千辛万苦，我们的视频终于出炉后，就要和音频结合，成为一个可以分发播放的媒体文件了，但是我们首先得先把片源的音频抽取出来，通常针对不同的视频源的容器封装，我们需要不同的工具。

### 抽取：

#### MP4:

MP4是国际标准组织（ISO）指定的媒体文件容器，从mp4中提取音频流，我们通常使用ffmpeg来抽取音频流:

```text
ffmpeg -i input.mp4 -c:a copy output.m4a
```

#### MKV：

从mkv里抽取可以使用ffmpeg，但更推荐用MeGUI的HD Streams Extractor

如果你知道你抽的源视频就一条音轨，打开终端直接来一个：

```text
ffmpeg -i input.mkv -c:a copy output.mka
```

这样就完事儿了，但是MKV有的时候还有章节，更多条音轨等内容，所以更推荐使用MeGUI来抽取

我们在MeGUI的Tools里，选择HD Streams Extractor，然后具体用法看图就完事儿了

![&#x754C;&#x9762;&#x7B80;&#x4ECB;](https://i.v2ex.co/3r4420x7.png)

#### TS/M2TS：

同MeGUI，要么你可以试试TSMuxeR，这软件的教学我就不写了。

### 编码：

#### AAC:

可以先参考这里来获得一些信息：[https://nazorip.site/archives/44/](https://nazorip.site/archives/44/)

总的来说如果有aac编码需求，假设你的qaac以及ffmpeg都已经丢进PATH里了的话，你可以用ffmpeg来解码音频并且pipeline给qaac输出最终结果（注意，pipeline不能用powershell完成！）：

```text
ffmpeg -i input.mka -hide_banner -map a:0 -f wav pipe:|qaac64 -v 192 --adts -s --ignorelength - -o "output.aac" --threading
```

#### FLAC：

如果你抽出来的音频流是无损的（基本都是PCM编码的wav），并且你也想无损的把它放进你的成品中，也想减小点体积的话，为什么不来试试神奇的flac呢？

flac的用法真的非常简单，因为本来就是无损编码也不需要指定什么码率，如果编码的是wav的话，指定个输入就行了：

```text
flac input.wav
```

然后你就能获得和输入文件同名的flac文件了

当然了，如果你要把其他格式转换成flac的话（转换之前先想清楚自己做的是不是有损→无损的睿智操作），建议还是先用ffmpeg转换成wav再进行进一步操作

```text
ffmpeg -i input.mka input.wav
flac input.wav
```

关于flac其他的参数，可以自行查阅doc，这里就不多做介绍了

## 封装

封装可以选择的容器有非常多，一般我们使用MP4和MKV做最终的封装容器

#### MP4：

封装MP4需要有很多的注意点，因为MP4是ISO组织定下的标准，所以它的兼容性要求非常高，能够封装进去的音频格式比较有限，例如flac什么的就没法封装进去，所以丢不进去就考虑把音频重新编码一下

但是不要考虑那么多，反正记住**mp4里抽出来的音频怎么都能封进mp4里，mkv里抽出来的就不一定了**就是了。

MP4封装仍然推荐MeGUI的MP4 Muxer，你可以在Tools-Muxer里找到它，至于怎么用看图就是了，记得按queue开始封装

![&#x754C;&#x9762;&#x7B80;&#x4ECB;](https://i.v2ex.co/SMti27TH.png)

#### MKV:

MKV作为万能容器，基本上什么都能往里面塞，封装mkv的工具我们一般使用mkvtoolnix，你可以在工具章节的封装工具中找到它







