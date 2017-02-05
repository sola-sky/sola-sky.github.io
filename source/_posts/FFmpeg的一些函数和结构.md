---
title: FFmpeg的一些函数和结构
date: 2016-06-30 19:34:36
tags:
---

**av_new_stream()**

```c
/**
 * Can only be called in the read_header() function. If the flag
 * AVFMTCTX_NOHEADER is in the format context, then new streams
 * can be added in read_packet too.
 */
AVStream *av_new_stream(AVFormatContext *s, int id)
```

为媒体文件添加一个流，一般作为输出的媒体文件容器添加音视频流。

**dump_format()**

```c
attribute_deprecated void dump_format(AVFormatContext *ic,
                                      int index,
                                      constchar *url,
                                      int is_output);
```

该函数的作用就是检查下初始化过程中设置的参数是否符合规范。

**av_set_parameters()**

```c
attribute_deprecated int av_set_parameters(AVFormatContext *s, AVFormatParameters *ap)
```

设置初始化参数


**av_write_header()**

```c
attribute_deprecated int av_write_header(AVFormatContext *s)
```

把流头信息写入到媒体文件中。如果成功返回0。

**AVCodecContext**

- enum AVMediaType codec_type：编解码器的类型（视频，音频...）
- struct AVCodec  *codec：采用的解码器AVCodec（H.264,MPEG2...）
- int bit_rate：平均比特率
- uint8_t *extradata; int extradata_size：针对特定编码器包含的附加信息（例如对于H.264解码器来说，存储SPS，PPS等）
- AVRational time_base：根据该参数，可以把PTS转化为实际的时间（单位为秒s）
- int width, height：如果是视频的话，代表宽和高
- int refs：运动估计参考帧的个数（H.264的话会有多帧，MPEG2这类的一般就没有了）
- int sample_rate：采样率（音频）
- int channels：声道数（音频）
- enum AVSampleFormat sample_fmt：采样格式
- int profile：型（H.264里面就有，其他编码标准应该也有）
- int level：级（和profile差不太多）

在这里需要注意：AVCodecContext中很多的参数是编码的时候使用的，而不是解码的时候使用的。

**codec_type**

编解码器类型有一下几种：

```c
enum AVMediaType {  
    AVMEDIA_TYPE_UNKNOWN = -1,  ///< Usually treated as AVMEDIA_TYPE_DATA  
    AVMEDIA_TYPE_VIDEO,  
    AVMEDIA_TYPE_AUDIO,  
    AVMEDIA_TYPE_DATA,          ///< Opaque data information usually continuous  
    AVMEDIA_TYPE_SUBTITLE,  
    AVMEDIA_TYPE_ATTACHMENT,    ///< Opaque data information usually sparse  
    AVMEDIA_TYPE_NB  
};  
```

**sample_fmt**

在FFMPEG中音频采样格式有以下几种：

```c
enum AVSampleFormat {  
    AV_SAMPLE_FMT_NONE = -1,  
    AV_SAMPLE_FMT_U8,          ///< unsigned 8 bits  
    AV_SAMPLE_FMT_S16,         ///< signed 16 bits  
    AV_SAMPLE_FMT_S32,         ///< signed 32 bits  
    AV_SAMPLE_FMT_FLT,         ///< float  
    AV_SAMPLE_FMT_DBL,         ///< double  
  
    AV_SAMPLE_FMT_U8P,         ///< unsigned 8 bits, planar  
    AV_SAMPLE_FMT_S16P,        ///< signed 16 bits, planar  
    AV_SAMPLE_FMT_S32P,        ///< signed 32 bits, planar  
    AV_SAMPLE_FMT_FLTP,        ///< float, planar  
    AV_SAMPLE_FMT_DBLP,        ///< double, planar  
  
    AV_SAMPLE_FMT_NB           ///< Number of sample formats. DO NOT USE if linking dynamically  
};  
```

**AVPacket**

AVPacket是存储压缩编码数据相关信息的结构体

```c
typedef struct AVPacket {  
    /** 
     * Presentation timestamp in AVStream->time_base units; the time at which 
     * the decompressed packet will be presented to the user. 
     * Can be AV_NOPTS_VALUE if it is not stored in the file. 
     * pts MUST be larger or equal to dts as presentation cannot happen before 
     * decompression, unless one wants to view hex dumps. Some formats misuse 
     * the terms dts and pts/cts to mean something different. Such timestamps 
     * must be converted to true pts/dts before they are stored in AVPacket. 
     */  
    int64_t pts;  
    /** 
     * Decompression timestamp in AVStream->time_base units; the time at which 
     * the packet is decompressed. 
     * Can be AV_NOPTS_VALUE if it is not stored in the file. 
     */  
    int64_t dts;  
    uint8_t *data;  
    int   size;  
    int   stream_index;  
    /** 
     * A combination of AV_PKT_FLAG values 
     */  
    int   flags;  
    /** 
     * Additional packet data that can be provided by the container. 
     * Packet can contain several types of side information. 
     */  
    struct {  
        uint8_t *data;  
        int      size;  
        enum AVPacketSideDataType type;  
    } *side_data;  
    int side_data_elems;  
  
    /** 
     * Duration of this packet in AVStream->time_base units, 0 if unknown. 
     * Equals next_pts - this_pts in presentation order. 
     */  
    int   duration;  
    void  (*destruct)(struct AVPacket *);  
    void  *priv;  
    int64_t pos;                            ///< byte position in stream, -1 if unknown  
  
    /** 
     * Time difference in AVStream->time_base units from the pts of this 
     * packet to the point at which the output from the decoder has converged 
     * independent from the availability of previous frames. That is, the 
     * frames are virtually identical no matter if decoding started from 
     * the very first frame or from this keyframe. 
     * Is AV_NOPTS_VALUE if unknown. 
     * This field is not the display duration of the current packet. 
     * This field has no meaning if the packet does not have AV_PKT_FLAG_KEY 
     * set. 
     * 
     * The purpose of this field is to allow seeking in streams that have no 
     * keyframes in the conventional sense. It corresponds to the 
     * recovery point SEI in H.264 and match_time_delta in NUT. It is also 
     * essential for some types of subtitle streams to ensure that all 
     * subtitles are correctly displayed after seeking. 
     */  
    int64_t convergence_duration;  
} AVPacket;  
```
uint8_t *data：压缩编码的数据。
例如对于H.264来说。1个AVPacket的data通常对应一个NAL。
注意：在这里只是对应，而不是一模一样。他们之间有微小的差别：使用FFMPEG类库分离出多媒体文件中的H.264码流
因此在使用FFMPEG进行视音频处理的时候，常常可以将得到的AVPacket的data数据直接写成文件，从而得到视音频的码流文件。
int   size：data的大小
int64_t pts：显示时间戳
int64_t dts：解码时间戳
int   stream_index：标识该AVPacket所属的视频/音频流。

**av_init_packet()**

```c
void av_init_packet(AVPacket *pkt);
```

// 使用默认值初始化AVPacket
// 定义AVPacket对象后,请使用av_init_packet进行初始化

**av_free_packet()**

```c
void av_free_packet(AVPacket *pkt);
```
用于释放AVPacket对象

**av_read_frame()**

```c
int av_read_frame(AVFormatContext *s, AVPacket *pkt);
```

从输入源文件容器中读取一个AVPacket数据包。该函数读出的包并不每次都是有效的,对于读出的包我们都应该进行相应的解码(视频解码/音频解码),在返回值>=0时,循环调用该函数进行读取,循环调用之前请调用av_free_packet函数清理AVPacket。

**avcodec_decode_video2()**

```c
int avcodec_decode_video2(AVCodecContext *avctx, AVFrame *picture,
                         int *got_picture_ptr,
                         AVPacket *avpkt);
```

- 解码视频流AVPacket
- 使用av_read_frame读取媒体流后需要进行判断,如果为视频流则调用该函数解码
- 返回结果<0时失败,此时程序应该退出检查原因
- 返回>=0时正常,假设 读取包为:AVPacket vPacket 返回值为 int vLen; 每次解码正常时,对vPacket做如下处理:
vPacket.size -= vLen;
vPacket.data += vLen;
- 如果 vPacket.size==0,则继续读下一流包,否则继续调度该方法进行解码,直到vPacket.size==0
- 返回 got_picture_ptr > 0 时,表示解码到了AVFrame *picture,其后可以对picture进程处理

**avcodec_decode_audio3()**

```c
int avcodec_decode_audio3(AVCodecContext *avctx, int16_t *samples,
                         int *frame_size_ptr,
                         AVPacket *avpkt);
```

**AVCodec**

```c
typedef struct AVCodec {
    /**
     * Name of the codec implementation.
     * The name is globally unique among encoders and among decoders (but an
     * encoder and a decoder can share the same name).
     * This is the primary way to find a codec from the user perspective.
     */
    const char *name;
    /**
     * Descriptive name for the codec, meant to be more human readable than name.
     * You should use the NULL_IF_CONFIG_SMALL() macro to define it.
     */
    const char *long_name;
    enum AVMediaType type;
    enum AVCodecID id;
    /**
     * Codec capabilities.
     * see AV_CODEC_CAP_*
     */
    int capabilities;
    const AVRational *supported_framerates; ///< array of supported framerates, or NULL if any, array is terminated by {0,0}
    const enum AVPixelFormat *pix_fmts;     ///< array of supported pixel formats, or NULL if unknown, array is terminated by -1
    const int *supported_samplerates;       ///< array of supported audio samplerates, or NULL if unknown, array is terminated by 0
    const enum AVSampleFormat *sample_fmts; ///< array of supported sample formats, or NULL if unknown, array is terminated by -1
    const uint64_t *channel_layouts;         ///< array of support channel layouts, or NULL if unknown. array is terminated by 0
    uint8_t max_lowres;                     ///< maximum value for lowres supported by the decoder, no direct access, use av_codec_get_max_lowres()
    const AVClass *priv_class;              ///< AVClass for the private context
    const AVProfile *profiles;              ///< array of recognized profiles, or NULL if unknown, array is terminated by {FF_PROFILE_UNKNOWN}

    /*****************************************************************
     * No fields below this line are part of the public API. They
     * may not be used outside of libavcodec and can be changed and
     * removed at will.
     * New public fields should be added right above.
     *****************************************************************
     */
    int priv_data_size;
    struct AVCodec *next;
    /**
     * @name Frame-level threading support functions
     * @{
     */
    /**
     * If defined, called on thread contexts when they are created.
     * If the codec allocates writable tables in init(), re-allocate them here.
     * priv_data will be set to a copy of the original.
     */
    int (*init_thread_copy)(AVCodecContext *);
    /**
     * Copy necessary context variables from a previous thread context to the current one.
     * If not defined, the next thread will start automatically; otherwise, the codec
     * must call ff_thread_finish_setup().
     *
     * dst and src will (rarely) point to the same context, in which case memcpy should be skipped.
     */
    int (*update_thread_context)(AVCodecContext *dst, const AVCodecContext *src);
    /** @} */

    /**
     * Private codec-specific defaults.
     */
    const AVCodecDefault *defaults;

    /**
     * Initialize codec static data, called from avcodec_register().
     */
    void (*init_static_data)(struct AVCodec *codec);

    int (*init)(AVCodecContext *);
    int (*encode_sub)(AVCodecContext *, uint8_t *buf, int buf_size,
                      const struct AVSubtitle *sub);
    /**
     * Encode data to an AVPacket.
     *
     * @param      avctx          codec context
     * @param      avpkt          output AVPacket (may contain a user-provided buffer)
     * @param[in]  frame          AVFrame containing the raw data to be encoded
     * @param[out] got_packet_ptr encoder sets to 0 or 1 to indicate that a
     *                            non-empty packet was returned in avpkt.
     * @return 0 on success, negative error code on failure
     */
    int (*encode2)(AVCodecContext *avctx, AVPacket *avpkt, const AVFrame *frame,
                   int *got_packet_ptr);
    int (*decode)(AVCodecContext *, void *outdata, int *outdata_size, AVPacket *avpkt);
    int (*close)(AVCodecContext *);
    /**
     * Decode/encode API with decoupled packet/frame dataflow. The API is the
     * same as the avcodec_ prefixed APIs (avcodec_send_frame() etc.), except
     * that:
     * - never called if the codec is closed or the wrong type,
     * - AVPacket parameter change side data is applied right before calling
     *   AVCodec->send_packet,
     * - if AV_CODEC_CAP_DELAY is not set, drain packets or frames are never sent,
     * - only one drain packet is ever passed down (until the next flush()),
     * - a drain AVPacket is always NULL (no need to check for avpkt->size).
     */
    int (*send_frame)(AVCodecContext *avctx, const AVFrame *frame);
    int (*send_packet)(AVCodecContext *avctx, const AVPacket *avpkt);
    int (*receive_frame)(AVCodecContext *avctx, AVFrame *frame);
    int (*receive_packet)(AVCodecContext *avctx, AVPacket *avpkt);
    /**
     * Flush buffers.
     * Will be called when seeking
     */
    void (*flush)(AVCodecContext *);
    /**
     * Internal codec capabilities.
     * See FF_CODEC_CAP_* in internal.h
     */
    int caps_internal;
} AVCodec;
```

最主要的几个变量：
- const char *name：编解码器的名字，比较短
- const char *long_name：编解码器的名字，全称，比较长
- enum AVMediaType type：指明了类型，是视频，音频，还是字幕
- enum AVCodecID id：ID，不重复
- const AVRational *supported_framerates：支持的帧率（仅视频）
- const enum AVPixelFormat *pix_fmts：支持的像素格式（仅视频）
- const int *supported_samplerates：支持的采样率（仅音频）
- const enum AVSampleFormat *sample_fmts：支持的采样格式（仅音频）
- const uint64_t *channel_layouts：支持的声道数（仅音频）
- int priv_data_size：私有数据的大小

**enum AVMediaType**

```c
enum AVMediaType {  
    AVMEDIA_TYPE_UNKNOWN = -1,  ///< Usually treated as AVMEDIA_TYPE_DATA  
    AVMEDIA_TYPE_VIDEO,  
    AVMEDIA_TYPE_AUDIO,  
    AVMEDIA_TYPE_DATA,          ///< Opaque data information usually continuous  
    AVMEDIA_TYPE_SUBTITLE,  
    AVMEDIA_TYPE_ATTACHMENT,    ///< Opaque data information usually sparse  
    AVMEDIA_TYPE_NB  
};  
```

**enum AVCodecId**

```c
enum AVCodecID {  
    AV_CODEC_ID_NONE,  
  
    /* video codecs */  
    AV_CODEC_ID_MPEG1VIDEO,  
    AV_CODEC_ID_MPEG2VIDEO, ///< preferred ID for MPEG-1/2 video decoding  
    AV_CODEC_ID_MPEG2VIDEO_XVMC,  
    AV_CODEC_ID_H261,  
    AV_CODEC_ID_H263,  
    AV_CODEC_ID_RV10,  
    AV_CODEC_ID_RV20,  
    AV_CODEC_ID_MJPEG,  
    AV_CODEC_ID_MJPEGB,  
    AV_CODEC_ID_LJPEG,  
    AV_CODEC_ID_SP5X,  
    AV_CODEC_ID_JPEGLS,  
    AV_CODEC_ID_MPEG4,  
    AV_CODEC_ID_RAWVIDEO,  
    AV_CODEC_ID_MSMPEG4V1,  
    AV_CODEC_ID_MSMPEG4V2,  
    AV_CODEC_ID_MSMPEG4V3,  
    AV_CODEC_ID_WMV1,  
    AV_CODEC_ID_WMV2,  
    AV_CODEC_ID_H263P,  
    AV_CODEC_ID_H263I,  
    AV_CODEC_ID_FLV1,  
    AV_CODEC_ID_SVQ1,  
    AV_CODEC_ID_SVQ3,  
    AV_CODEC_ID_DVVIDEO,  
    AV_CODEC_ID_HUFFYUV,  
    AV_CODEC_ID_CYUV,  
    AV_CODEC_ID_H264,  
    ...（代码太长，略）  
}  
```

**const enum AVPixelFormat *pix_fmts**

AVPixelFormat定义如下

```
enum AVPixelFormat {  
    AV_PIX_FMT_NONE = -1,  
    AV_PIX_FMT_YUV420P,   ///< planar YUV 4:2:0, 12bpp, (1 Cr & Cb sample per 2x2 Y samples)  
    AV_PIX_FMT_YUYV422,   ///< packed YUV 4:2:2, 16bpp, Y0 Cb Y1 Cr  
    AV_PIX_FMT_RGB24,     ///< packed RGB 8:8:8, 24bpp, RGBRGB...  
    AV_PIX_FMT_BGR24,     ///< packed RGB 8:8:8, 24bpp, BGRBGR...  
    AV_PIX_FMT_YUV422P,   ///< planar YUV 4:2:2, 16bpp, (1 Cr & Cb sample per 2x1 Y samples)  
    AV_PIX_FMT_YUV444P,   ///< planar YUV 4:4:4, 24bpp, (1 Cr & Cb sample per 1x1 Y samples)  
    AV_PIX_FMT_YUV410P,   ///< planar YUV 4:1:0,  9bpp, (1 Cr & Cb sample per 4x4 Y samples)  
    AV_PIX_FMT_YUV411P,   ///< planar YUV 4:1:1, 12bpp, (1 Cr & Cb sample per 4x1 Y samples)  
    AV_PIX_FMT_GRAY8,     ///<        Y        ,  8bpp  
    AV_PIX_FMT_MONOWHITE, ///<        Y        ,  1bpp, 0 is white, 1 is black, in each byte pixels are ordered from the msb to the lsb  
    AV_PIX_FMT_MONOBLACK, ///<        Y        ,  1bpp, 0 is black, 1 is white, in each byte pixels are ordered from the msb to the lsb  
    AV_PIX_FMT_PAL8,      ///< 8 bit with PIX_FMT_RGB32 palette  
    AV_PIX_FMT_YUVJ420P,  ///< planar YUV 4:2:0, 12bpp, full scale (JPEG), deprecated in favor of PIX_FMT_YUV420P and setting color_range  
    AV_PIX_FMT_YUVJ422P,  ///< planar YUV 4:2:2, 16bpp, full scale (JPEG), deprecated in favor of PIX_FMT_YUV422P and setting color_range  
    AV_PIX_FMT_YUVJ444P,  ///< planar YUV 4:4:4, 24bpp, full scale (JPEG), deprecated in favor of PIX_FMT_YUV444P and setting color_range  
    AV_PIX_FMT_XVMC_MPEG2_MC,///< XVideo Motion Acceleration via common packet passing  
    AV_PIX_FMT_XVMC_MPEG2_IDCT,  
    ...（代码太长，略）  
}  
```

**const enum AVSampleFormat *sample_fmts**

```c
enum AVSampleFormat {  
    AV_SAMPLE_FMT_NONE = -1,  
    AV_SAMPLE_FMT_U8,          ///< unsigned 8 bits  
    AV_SAMPLE_FMT_S16,         ///< signed 16 bits  
    AV_SAMPLE_FMT_S32,         ///< signed 32 bits  
    AV_SAMPLE_FMT_FLT,         ///< float  
    AV_SAMPLE_FMT_DBL,         ///< double  
  
    AV_SAMPLE_FMT_U8P,         ///< unsigned 8 bits, planar  
    AV_SAMPLE_FMT_S16P,        ///< signed 16 bits, planar  
    AV_SAMPLE_FMT_S32P,        ///< signed 32 bits, planar  
    AV_SAMPLE_FMT_FLTP,        ///< float, planar  
    AV_SAMPLE_FMT_DBLP,        ///< double, planar  
  
    AV_SAMPLE_FMT_NB           ///< Number of sample formats. DO NOT USE if linking dynamically  
};  
```

每个编解码器对应一个该结构体，查看一下ffmpeg的源码，我们可以在/libavcodec/h264.c文件中看到：

```
AVCodec ff_h264_decoder = {
    .name                  = "h264",
    .long_name             = NULL_IF_CONFIG_SMALL("H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10"),
    .type                  = AVMEDIA_TYPE_VIDEO,
    .id                    = AV_CODEC_ID_H264,
    .priv_data_size        = sizeof(H264Context),
    .init                  = ff_h264_decode_init,
    .close                 = h264_decode_end,
    .decode                = h264_decode_frame,
    .capabilities          = /*AV_CODEC_CAP_DRAW_HORIZ_BAND |*/ AV_CODEC_CAP_DR1 |
                             AV_CODEC_CAP_DELAY | AV_CODEC_CAP_SLICE_THREADS |
                             AV_CODEC_CAP_FRAME_THREADS,
    .caps_internal         = FF_CODEC_CAP_INIT_THREADSAFE,
    .flush                 = flush_dpb,
    .init_thread_copy      = ONLY_IF_THREADS_ENABLED(decode_init_thread_copy),
    .update_thread_context = ONLY_IF_THREADS_ENABLED(ff_h264_update_thread_context),
    .profiles              = NULL_IF_CONFIG_SMALL(ff_h264_profiles),
    .priv_class            = &h264_class,
};
```

下面简单介绍一下遍历ffmpeg中的解码器信息的方法（这些解码器以一个链表的形式存储）：
1.注册所有编解码器：av_register_all();
2.声明一个AVCodec类型的指针，比如说AVCodec* first_c;
3.调用av_codec_next()函数，即可获得指向链表下一个解码器的指针，循环往复可以获得所有解码器的信息。注意，如果想要获得指向第一个解码器的指针，则需要将该函数的参数设置为NULL。

**AVStream**

AVStream是存储每一个视频/音频流信息的结构体，重要的变量如下：

- int index：标识该视频/音频流
- AVCodecContext *codec：指向该视频/音频流的AVCodecContext（它们是一一对应的关系）
- AVRational time_base：时基。通过该值可以把PTS，DTS转化为真正的时间。FFMPEG其他结构体中也有这个字段，但是根据我的经验，只有AVStream中的time_base是可用的。PTS*time_base=真正的时间
- int64_t duration：该视频/音频流长度
- AVDictionary *metadata：元数据信息
- AVRational avg_frame_rate：帧率（注：对视频来说，这个挺重要的）
- AVPacket attached_pic：附带的图片。比如说一些MP3，AAC音频文件附带的专辑封面
- void *priv_data：关联的上下文，因为AVStream只是一个着重于媒体流的共性。



**音视频数据流简单流程，由AVIOContext(URLContext/URLProtocol)表示的广义输入文件，在AVStream提供的特定文件容器流信息的指引下，用AVInputFormat(AVFormatContext)的接口函数读取完整一帧数据，**





