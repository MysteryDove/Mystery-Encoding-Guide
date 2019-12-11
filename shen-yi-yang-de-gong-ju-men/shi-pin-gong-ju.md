# 视频工具

## Vapoursynth

为了读入片源并且处理，我们需要一个帧服务器（Frameserver）来帮助我们，同时我们也需要给Vapoursynth安装不同的插件（其实就是把编译好的dll丢进Plugins文件夹里）来实现读入，处理。

Vapoursynth的安装非常麻烦，如果需要系统全局安装的话，也可能会被路径问题搞得晕头转向，所以这里推荐开箱即用开袋即食的Portable版，而恰巧有人整理了一份囊括几乎所有插件和许多工具的VS便携版，同时它还带了最新的X265和X264，你可以在[这里](https://github.com/theChaosCoder/vapoursynth-portable-FATPACK/releases)下载到它

当然如果你有自己的想法，想要自己安装一份Vapoursynth，那你可以在下面的链接中下载到想要的东西

VS官网：[http://www.vapoursynth.com/](http://www.vapoursynth.com/)

插件（滤镜）仓库：[http://vsdb.top/](http://vsdb.top/)

如果你使用的是Linux，那可以参考使用：[https://github.com/himesaka-noa/vs-toolset-installer-ubuntu](https://github.com/himesaka-noa/vs-toolset-installer-ubuntu)\(注意，这个脚本目前仅支持Ubuntu18.04及以上版本\)

## VSEdit

既然我们已经有了Vapoursynth，那么就需要相对应的编辑器来编写，预览脚本输出的结果，所以我们需要VSEdit。如果你已经下载了上面提到的开袋即食版本，里面就已经自带有vsedit了，你可以在Vapoursynth64文件夹中找到它。

VSEdit有最基础的代码补全功能，并且能够进行视频预览，脚本测试甚至编码等操作，用过时候你一定会喜欢上它！

对了！如果你使用的是安装版或者自己做了一个Portable版的话，你可以在[这里](https://bitbucket.org/mystery_keeper/vapoursynth-editor/downloads/)下载到它

## X264/X265

没编码器你还压什么片嘛，一个预先编译好的编码器是必须要有的，当然了，如果你下载了上面的VS便携版的话，这些编码器都已经附带进去了，你可以在bin文件夹里找到它们。

X264/X265的编译版本你都可以在这里找到：[http://msystem.waw.pl/x265/](http://msystem.waw.pl/x265/)

这个页面里有非常多编译版本，X265通常选择右边Stable分支，适合你CPU指令集的编译版本，而使用GCC与VS 2019\(也就是MSVC\)编译的版本区别仅在压制速度上，通常可能MSVC编译的版本会运行起来更快一些

**注意！我们推荐将X264/X265加入到系统的PATH中，在以后使用CLI编码时会方便很多**

当然了，如果你要搞什么av1玩的话，我这里也就丢几个链接吧：

rav1e：[https://github.com/xiph/rav1e](https://github.com/xiph/rav1e)

svt-av1：[https://github.com/OpenVisualCloud/SVT-AV1](https://github.com/OpenVisualCloud/SVT-AV1)

