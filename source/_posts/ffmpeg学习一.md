---
title: ffmpeg学习一
date: 2016-06-30 07:12:50
tags:
---

**av_find_stream_info()**

```c
int av_find_stream_info(AVFromatContext *ic)
读取媒体文件的包来来获取信息。这对于没有头信息的文件来说是非常有用的，比如MPEG。
以防止MPEG-2的重复帧模式，该函数也用于计算真正的帧率。
返回大于等于0是OK的。
```

**avcodec_find_decoder()**

```c
AVCodec *avcodec_find_decoder(enum CodecID id)
```
通过code ID来查找一个寂静注册的解码器。查找解码器之前,必须先调用av_register_all注册所有支持的解码器

**avcodec_find_decoder_by_name()**

```c
AVCodec *avcodec_find_decoder_by_name(constchar *name)
```
通过一个指定的名称查找一个已经注册的音视频解码器。音视频解码器保存在一个链表中,查找过程中,函数从头到尾遍历链表,通过比较解码器的name来查找。查找解码器之前,必须先调用av_register_all注册所有支持的解码器。

**avcodec_find_encoder()**

```c
AVCodec *avcodec_find_encoder(enum CodecID id)
```
通过code ID查找一个已经注册的音视频编码器。查找编码器之前,必须先调用av_register_all注册所有支持的编码器。

**avcodec_find_encoder_by_name()**

```c
AVCodec *avcodec_find_encoder_by_name(constchar *name)
```
通过一个指定的名称查找一个已经注册的音视频编码器。查找编码器之前,必须先调用av_register_all注册所有支持的编码器。

**avcodec_open()**

```c
int avcodec_open(AVCodecContext *avctx, AVCodec *codec)
```
使用给定的AVCodec初始化AVCodecContext。成功返回0，失败负数。

**av_guess_format()**

```c
AVOutputFormat *av_guess_format(constchar *short_name,
                                constchar *filename,
                                constchar *mime_type)
```
返回一个已经注册的最合适的输出格式