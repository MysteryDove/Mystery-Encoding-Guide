# 最后的检查与一些小技巧

到此为止，我们还没有对自己压制完成的成品进行检查，你可以自己完整看一遍，但更多时候比较懒还可以靠软件代劳，这里我们推荐用RPChecker对视频进行检查。

## 视频花屏检查

[RPChecker](../shen-yi-yang-de-gong-ju-men/zong-he-gong-ju.md#rp-checker)是vcb-s开发的检查视频是否出现花屏坏帧的工具，它需要FFMPEG或VS来正常运行，所以你可能需要手动指定这些软件的位置。

首先我们打开软件本体，由于RPChecker的FFMPEG模式有那么一点……不靠谱，所以建议将Portable版VS的文件夹（就是vspipe.exe在的那个文件夹）添加入系统PATH中，右键左上角的RP图标，右键选择`使用PSNR(VS)` 没有问题的话就能进入VS模式，当然你把rpchecker本体丢进Portable版VS里也行：

![&#x9009;&#x62E9;vs&#x6A21;&#x5F0F;&#xFF0C;&#x9009;&#x5B8C;&#x6807;&#x9898;&#x680F;&#x4F1A;&#x53D8;&#x6210;Vapoursynth](https://i.v2ex.co/mY5Vp37P.png)

然后我们选择载入：

![](https://i.v2ex.co/2s27898g.png)



往右边那俩框里拖片源和成品，对照组也可以是其他你知道是没有问题的片源（比如jsum的）

载入完按确认，然后主界面按一下分析，等分析完后按图表：

![&#x5982;&#x679C;&#x4F60;&#x770B;&#x5230;&#x7684;&#x662F;&#x8FD9;&#x6837;&#xFF0C;&#x591A;&#x534A;&#x6CA1;&#x4EC0;&#x4E48;&#x95EE;&#x9898;](https://i.v2ex.co/Xlnt931r.png)

通常一些抗锯齿手段会导致staff片段（字幕）有较低的PSNR值，这通常没什么问题。

但是如果有某一帧或几帧特别特别低，那建议单独拿出来看一下，这些情况多半是中奖了~~（该返工啦）~~。

## 字幕及子集字体内封

许多时候我们的外挂字幕都必须附带上体积巨大的字体文件压缩包，我们可以使用[AssFontSubset](../shen-yi-yang-de-gong-ju-men/zong-he-gong-ju.md#assfontsubset)来只抽取字幕中使用的字的字形，生成子集字体。将子集字体和修改过的ass文件内封到mkv容器中，可以在受支持的播放器中完美的以没有字体问题的方式播放。

首先确认自己[fonttools](https://github.com/fonttools/fonttools)有没有正确安装，然后打开AssFontSubset，将准备好的ass拖到字幕文件一栏，在软件同一目录创建`fonts`文件夹，将字幕用的字体都丢进去，然后按一下开始就开始进行字体抽取操作了。

![&#x53EF;&#x4EE5;&#x4E00;&#x6B21;&#x6027;&#x8F93;&#x5165;&#x591A;&#x4E2A;&#x5B57;&#x5E55;&#x6587;&#x4EF6;](https://i.v2ex.co/7JhL6b2A.png)

如果没有报错，你就能在output文件夹获得成品字幕和子集字体了

我们打开mkvtoolnix，把**output中的字幕**加入字幕轨道，在附件中把子集字体拖进去，封装就好了

![&#x4F60;&#x7684;&#x9644;&#x4EF6;&#x5E94;&#x8BE5;&#x957F;&#x8FD9;&#x6837;](https://i.v2ex.co/R9e9XN34.png)

