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

TBD

## 音频抽取与编码

TBD

## 封装

TBD

