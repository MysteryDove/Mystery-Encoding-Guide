# 综合工具

其实有些没法分门别类的我也丢这里了，见谅

## FFMPEG

对于音视频编码和解码史来说，FFMPEG无疑是里程碑一样的存在，支持几乎所有你能想象得到的编码格式的编解码，封装抽流什么的都不在话下，但是正因为它要顾及的东西太多了，所以做许多工作时都会出现封装不标准等等小问题，但是做许多事用它仍然是便利快捷的选择

你可以在这里下载到它：[https://www.ffmpeg.org/download.html](https://www.ffmpeg.org/download.html)

## MeGUI

如果要说综合工具的话，MeGUI绝对是瑞士军刀级别的存在，它提供的抽流混流工具是我最常使用的，而且它还有章节编辑抽取工具，这在做BDRip的过程中都是非常好用的。

你可以在这里下载到它：[https://sourceforge.net/projects/megui/](https://sourceforge.net/projects/megui/)

## MediaInfo

MediaInfo绝对是做压制居家必备神器，它能够分析片源音频视频轨，给出这些片源的轨道信息，包括了音频/视频/字幕/章节等等，为下一步处理乃至封装时提供**极其重要的信息。**

你可以在这里下载到它：[https://mediaarea.net/en/MediaInfo](https://mediaarea.net/en/MediaInfo)

## IGSTools

来自皮神出品的神器，用于提取BD原盘菜单中的章节信息，具体用法会在分析片源章节讲到

下载安装指引：[https://github.com/SAPikachu/igstools](https://github.com/SAPikachu/igstools)

使用它你必须得有个Python环境，所以这个得自备啦，还得记得去装pypng！

## RP-Checker

vcb-s出品工具之一，使用ffmpeg或者VS来计算对照视频与压制后视频的[PSNR](https://en.wikipedia.org/wiki/Peak_signal-to-noise_ratio)/[SSIM](https://en.wikipedia.org/wiki/Structural_similarity)/[GMSD](https://www4.comp.polyu.edu.hk/~cslzhang/IQA/GMSD/GMSD.htm)，生成图表来直观的展现结果，可以用于检查压制成品中是否出现了由于电脑不稳定导致的某一帧花屏等现象~~（宇宙射线使得内存出现了位翻转）~~，从而避免最终成品发出去还要返工rev。

你可以在这里下载到它：[https://github.com/vcb-s/rp-checker/releases](https://github.com/vcb-s/rp-checker/releases)

如果你有能力帮助我们完善这个软件或者发现了bug，欢迎来提Issue与PR

## ChapterTools

vcb-s出品工具之二，能够解析蓝光原盘的playlist，提取并编辑章节信息

你可以在这里获得它：[https://bitbucket.org/TautCony/chaptertool/downloads/](https://bitbucket.org/TautCony/chaptertool/downloads/)

同样的，如果你有能力帮助我们完善这个软件或者发现了bug，欢迎来提Issue与PR

## AssFontSubset

油总出品的字体子集内封工具，使用它需要用`pip`安装`fonttools`

使用方法和下载地址（release里下载）：[https://github.com/youlun/AssFontSubset](https://github.com/youlun/AssFontSubset)

