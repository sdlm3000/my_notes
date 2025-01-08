# FFmpeg 学习笔记





## 视频编码过程



```C++
// 该示例是把一个视频的rgb格式的输出为一个mp4格式的

av_register_all();
avcodec_register_all();

AVCoder *codec = avcodec_find_encoder(AV_CODEC_ID_H264);

// 创建编码器空间
AVCodeContext *c = avcodec_alloc_context3(codec);
// 设置编码器的一些参数
c->bit_rate = 4000000000;				// 压缩比特率
c->width = width;
c->height = height;
c->time_base = {1, fps};
c->framerate = {fps, 1};
c->gop_size = 50;						// 画面组关键帧大小
c->max_b_frame = 0;						// b帧
c->pix_fmt = AV_PIX_FMT_YUV420P;
c->codec_id = AV_CODEC_H264;
c->thread_count = 4;					// 处理线程数
c->flag |= AV_CODEC_FLAG_GLOBAL_HEADER;	// 全局编码信息

// 打开编码器
int ret = avcodec_open2(c, codec, NULL);	

// 创建输出上下文
AVFormatContext *oc = NULL;
// 这个例子中，char outfile[] = "test.mp4"
avformat_alloc_output_context2(&oc, 0, 0, outfile);


// 添加视频流
AVStream *st = avformat_new_stream(oc, NULL);
// 如果不这里不写 有时候会发生莫名错误
// 但是我觉得可能是这个上次mp4文件需要的，正常推流可能不需要
st->id = 0;
st->codecpar->codec_tag = 0;
avcodec_parameters_from_context(st->codecpar, c);
// 输出末尾为1，输入为0
// 打印输入和输出格式的详细信息
av_dump_format(oc, 0, outfile, 1);


// RGB 转 YUV
SwsContext *ctx = NULL;
ctx = sws_getContext(width, height, AV_PIX_FMT_BGRA,
                     width, height, AV_PIX_FMT_YUV420P,
                     SWS_BICUBIC, NULL, NULL, NULL);
// 输入空间
// 输出MP4所以最后乘4 ？好像是BGRA就是乘4
unsigned char *rgb = new unsigned char[width * height * 4];
// 输出空间
AVFrame *yuv = av_frame_alloc();
yuv->format = AV_PIX_FMT_YUV420P;
yuv->width = width;
yuv->height = height;
ret = av_frame_get_buff(yuv, 32);	// 32是什么意思？

// 写入mp4头文件
ret = avio_open(&oc->pb, outfile, AVIO_FLAG_WRITE);
ret = avformat_write_header(oc, NULL);


int p =0;
while(true) {
    int len = fread(rgb, 1, width * height * 4, fp);
    if (len <= 0) break;
    uint8_t *indata[AV_NUM_DATA_POINTERS]={ 0 };
    indata[0] = rgb;
    int inlinesize [AV_NUM_DATA_POINTERS]={ 0 };
    inlinesize[0] = width * 4;
    // bgra 转 yuv 的具体过程
    int h = sws_scale(ctx, indata, inlinesize, 0, height,
                      yuv->data, yuv->linesize);
    if(h <= 0) break;
    
    //编码视频帧
    yuv->pts = p;
    p += 3600:
    ret = avcodec_send_frame(c, yuv);
    AVPacket pkt;
    av_init_packet(&pkt);
    ret = avcodec_receive_packet(c, &pkt);
    
    
    av_interleaved_write_frame(oc, &pkt);
    
}




```

​	这篇[视频流编码流程](https://www.jianshu.com/p/4826c3c34caa)博客比较好，讲的很基础，而且就是图片输出视频的形式.

<img src="images/FFmpeg 学习笔记/image-20230521225644540.png" alt="image-20230521225644540" style="zoom:50%;" />



## 视频解码

![image-20230521195542986](images/FFmpeg 学习笔记/image-20230521195542986.png)



​	B站UP主 ffmpeg学堂 讲解内容，感觉不太明白。

```C++
av_register_all();
avformat_network_init();    // 网络流需要

// 打开视频文件
AVFormatContext* pFormat = NULL;

// // 打开本地文件
// const char* path = "xx.mp4";
       
// 打开网络流
const char* path = "流地址";
AVDictionary* opt = NULL;
av_dict_set(&opt, "rtsp_transport", "tcp", 0);
av_dict_set(&opt, "max_delay", 550, 0);

int ret = avformat_open_input(&pFormat, path, NULL, &opt);
if(ret) {
    printf("failed\n");
    return -1;
}

// 寻找流信息 -> hH264 width height
// 网络流似乎没有下面这部分操作
ret = avformat_find_stream_info(pFormat, NULL);
if(ret) {
    printf("failed\n");
    return -1;
}
int time  = pFormat->duration();
int mbittime = (time / 1000000) / 60;
int mmintime = (time / 1000000) % 60;
av_dump_format(pFormat, NULL, path, 0);     // 显示相应的信息
int VideoStream = -1, AudioStream = -1;
VideoStream = av_find_best_stream(pFormat, AVMEDIA_TYPE_VIDEO, -1, -1, NULL, NULL);

// 寻找解码器
AVCodec* vCodec = avcodec_find_decoder(pFormat->streams[VideoStream]->codec->codec_id);
if(!vCodec) {
    printf("failed\n");
    return -1;
}

// 打开解码器，并初始化解码器的上下文
ret = avcodec_open2(pFormat->streams[VideoStream]->codec->codec_id,
                vCodec, NULL);
if(ret) {
    printf("failed\n");
    return -1;
}

// 解码视频
// 申请原始空间
AVFrame* frame = av_frame_alloc();
AVFrame* frameYUV = av_frame_alloc();
int width = pFormat->streams[VideoStream]->codec->width;
int height = pFormat->streams[VideoStream]->codec->height;
// 分配空间 进行图像转换
int nSize = avpicture_get_size(AV_PIX_FMT_YUV420P, 
                    width, height);
uint8_t* buff = NULL
buff = (uint8_t *)av_malloc(nSize);

// 一帧图像
avpicture_full((AVPicture *)frameYUV, buff, AV_PIX_FMT_YUV420P, 
                    width, height);
AVPacket* packet = (AVPacket *)av_malloc(size(AVPacket));

// 转化上下文
sws_getConText(width, height, AV_PIX_FMT_YUV420P);
// 读帧
while(av_read_frame(pFormat, packet) >= 0){
    if(packet->stream_index == AVMEDIA_TYPE_VIDEO) {
        
    }
}
```

B站UP主 OA工作流 讲解的解码过程

![image-20230521211913095](images/FFmpeg 学习笔记/image-20230521211913095.png)









## 示例

### 推流

​		将图片打包成视频流，存放到 result.h264 中。

```c++
#include <qdebug.h>
#include <iostream>
#include "opencv2/opencv.hpp"
#include "opencv2/core.hpp"

extern "C"
{
#include "libavcodec/avcodec.h"
#include "libavformat/avformat.h"
#include "libswscale/swscale.h"
#include "libavutil/imgutils.h"
}

int width = 4096;
int height = 3072;

int main(int argc, char *argv[])
{
    /// 查找对应解码器
    const AVCodec *codec = avcodec_find_encoder(AV_CODEC_ID_H264);
    if(codec == nullptr) {
        std::cout << "没有找到对应的编码器！" << std::endl;
        return 1;
    }

    /// 申请编码器上下文，设置编码器参数
    AVCodecContext *codecCtx = avcodec_alloc_context3(codec);
    codecCtx->width = width;
    codecCtx->height = height;
    // 编码格式设置
    codecCtx->pix_fmt = AV_PIX_FMT_YUV420P;
    // 帧率设置
    codecCtx->time_base = AVRational{1, 25};
    // 设置码率
    codecCtx->bit_rate = 400000;
    // 设置显示率
    codecCtx->framerate = AVRational{25, 1};
    // 关键帧设置：gop表示多少个帧中存在一个关键帧
    codecCtx->gop_size = 15;
    // 设置h264的量化步长范围
    codecCtx->qmin = 10;
    codecCtx->qmax = 31;
    // B帧设置为0
    codecCtx->max_b_frames = 0;
    // 设置编码器id
    codecCtx->codec_id = AV_CODEC_ID_H264;
    // 设置流
    codecCtx->codec_type = AVMEDIA_TYPE_VIDEO;
    // 设置编码格式
    codecCtx->pix_fmt = AV_PIX_FMT_YUV420P;

    /// 打开编码器
    if (avcodec_open2(codecCtx, codec, NULL) < 0) {
        std::cout << "打开编码器失败！" << std::endl;
        return 1;
    }

    /// 打开需要输出的文件
//    AVFormatContext *ofmtCtx = avformat_alloc_context();
//    ofmtCtx->oformat = av_guess_format("h264", nullptr, nullptr);
    AVFormatContext *ofmtCtx = nullptr;
    // 输出文件名
    const char* outFile = "result.h264";
    if (avformat_alloc_output_context2(&ofmtCtx, NULL, "h264", outFile) < 0) {
        std::cout << "创建输出文件的上下文失败！" << std::endl;
        return 1;
    }


    /// 新建视频流
    AVStream *vStream = avformat_new_stream(ofmtCtx, codec);
    if(vStream == nullptr) {
        std::cout << "创建视频流失败！" << std::endl;
        return 1;
    }
    vStream->time_base.den = 25;
    vStream->time_base.num = 1;
    /// 填充视频结构体
    avcodec_parameters_from_context(vStream->codecpar, codecCtx);
    //打开文件
    if (avio_open(&ofmtCtx->pb, outFile, AVIO_FLAG_READ_WRITE) < 0) {
        std::cout << "output file open failed" << std::endl;
        return 1;
    }

    av_dump_format(ofmtCtx, 0, outFile, 1);
    //写入文件头信息
    if (avformat_write_header(ofmtCtx, NULL) < AVSTREAM_INIT_IN_WRITE_HEADER) {
        std::cout << "Write file header fail" << std::endl;
        return 1;
    }


    AVPacket *pkt = av_packet_alloc();

    AVFrame *rgbFrame = av_frame_alloc();
    rgbFrame->width = codecCtx->width;
    rgbFrame->height = codecCtx->height;
    rgbFrame->format = AV_PIX_FMT_BGR24;
    int size = av_image_get_buffer_size(AV_PIX_FMT_BGR24, codecCtx->width, codecCtx->height, 1);
    uint8_t* pictureBuf = (uint8_t*)av_malloc(size);
    av_image_fill_arrays(rgbFrame->data, rgbFrame->linesize,
        pictureBuf, AV_PIX_FMT_BGR24,
        codecCtx->width, codecCtx->height, 1);

    AVFrame *yuvFrame = av_frame_alloc();
    yuvFrame->format = codecCtx->pix_fmt;
    yuvFrame->width = codecCtx->width;
    yuvFrame->height = codecCtx->height;
    int yuvSize = av_image_get_buffer_size(codecCtx->pix_fmt, codecCtx->width, codecCtx->height, 1);
    uint8_t* yuvBuf = (uint8_t*)av_malloc(yuvSize);
    av_image_fill_arrays(yuvFrame->data, yuvFrame->linesize,
        yuvBuf, codecCtx->pix_fmt,
        codecCtx->width, codecCtx->height, 1);

    //设置BGR数据转换为YUV的SwsContext
    struct SwsContext* imgCtx = sws_getContext(
        codecCtx->width, codecCtx->height, AV_PIX_FMT_BGR24,
        codecCtx->width, codecCtx->height, codecCtx->pix_fmt,
        SWS_BILINEAR, NULL, NULL, NULL);

    av_new_packet(pkt, size);



    //这里使用OpenCV读取图像的数据，当然FFmpeg也可以读取图像数据，会略微麻烦一些之后会举例
    cv::Mat img;
    char imgPath[] = "../image/0.jpg";
    for (int i = 0; i < 3; i++) {
        imgPath[9] = '0' + i % 2;
        img = cv::imread(imgPath, cv::IMREAD_COLOR);
//        cv::imshow("img", img);
//        cv::waitKey(1);
        //----------------- BGR数据填充至图像帧 -------------------
        memcpy(pictureBuf, img.data, size);
        //进行图像格式转换
        sws_scale(imgCtx,
            rgbFrame->data,
            rgbFrame->linesize,
            0,
            codecCtx->height,
            yuvFrame->data,
            yuvFrame->linesize);

        for (int j = 0; j < 25; j++) {
            // 设置 pts 值，用于度量解码后视频帧位置
            yuvFrame->pts = i * 25 + j;
            // 解码时为 avcodec_send_packet ，编码时为 avcodec_send_frame
            if (avcodec_send_frame(codecCtx, yuvFrame) >= 0) {
                // 解码时为 avcodec_receive_frame ，编码时为 avcodec_receive_packet
                while (avcodec_receive_packet(codecCtx, pkt) >= 0) {
                    std::cout << "encoder success:" << pkt->size << std::endl;
                    pkt->stream_index = vStream->index;
                    // 将解码上下文中的时间基同等转换为流中的时间基
                    av_packet_rescale_ts(pkt, codecCtx->time_base, vStream->time_base);
                    // pos为-1表示未知，编码器编码时进行设置
                    pkt->pos = -1;
                    // 将包数据写入文件中
                    int ret = av_interleaved_write_frame(ofmtCtx, pkt);
                    if (ret < 0) {
                        std::cout << "error is:" << ret << std::endl;
                    }
                }
            }
        }
    }
    return 0;
}

```



### 拉流

​		从本地文件中拉取视频流，并进行播放

```c++
#include <qdebug.h>
#include <iostream>
#include "opencv2/opencv.hpp"
#include "opencv2/core.hpp"

extern "C"
{
#include "libavcodec/avcodec.h"
#include "libavformat/avformat.h"
#include "libswscale/swscale.h"
#include "libavutil/imgutils.h"
}


int main(int argc, char *argv[])
{
    int temp_ret = 0;
//    std::string videourl= "udp://192.168.3.22:5004";
//    avformat_network_init();
    std::string videourl= "/home/sdlm/sdlm/my_cpp/build-ffmpeg_push3-Desktop_Qt_5_12_12_GCC_64bit-Debug/result.h264";
    AVFormatContext *pFormatCtx = NULL;
    AVDictionary *options = NULL;
    AVPacket *av_packet = NULL; // AVPacket暂存解码之前的媒体数据
    AVCodecContext *avcodec_context = NULL;
    pFormatCtx = avformat_alloc_context(); //用来申请AVFormatContext类型变量并初始化默认参数,申请的空间
    //打开网络流或文件流
    if (avformat_open_input(&pFormatCtx, videourl.c_str(), NULL, &options) != 0) {
        qDebug() << "Couldn't open input stream.\n";
        return -1;
    }
    //获取视频文件信息
    if (avformat_find_stream_info(pFormatCtx, NULL) < 0)
        qDebug() << "Couldn't find stream information.";
    AVDictionaryEntry *tag = NULL;
    //查找码流中是否有视频流
    int videoindex = -1;
    unsigned i = 0;
    for (i = 0; i < pFormatCtx->nb_streams; i++) {
        if (pFormatCtx->streams[i]->codecpar->codec_type == AVMEDIA_TYPE_VIDEO) {
            videoindex = i;
            break;
        }
    }
    if (videoindex == -1)
        qDebug() << "Didn't find a video stream.\n";
    avcodec_context = avcodec_alloc_context3(NULL);     // 保存视频流对应的AVCodecContext信息
    avcodec_parameters_to_context(avcodec_context, pFormatCtx->streams[videoindex]->codecpar);
    const AVCodec* avcodec = avcodec_find_decoder(avcodec_context->codec_id);
    if(avcodec == nullptr) {
        qDebug() << "没有对应的编码器!";
        return -1;
    }
    if(avcodec_open2(avcodec_context, avcodec, NULL) < 0) {
        qDebug() << "打开解码器失败!";
        return -1;
    }
    av_dump_format(pFormatCtx, 0, videourl.c_str(), 0);
    AVFrame * avFrame = av_frame_alloc();
    av_image_alloc(avFrame->data, avFrame->linesize,
                   avcodec_context->width, avcodec_context->height,
                   AV_PIX_FMT_YUV420P, 1);
    av_packet = (AVPacket *)av_malloc(sizeof(AVPacket));

    AVFrame* rgbFrame = av_frame_alloc();
    AVFrame* yuvFrame = av_frame_alloc();

    SwsContext *swsContext = NULL;
    while (true) {

        // 图片转化成RGB格式
        swsContext = sws_getContext(avcodec_context->width, avcodec_context->height, avcodec_context->pix_fmt,
                                    avcodec_context->width, avcodec_context->height, AV_PIX_FMT_RGB24,
                                    SWS_BICUBIC, NULL, NULL, NULL);  //SWS_BICUBIC
        if (swsContext == NULL) {
            qDebug() << "swsContext is NULL";
           return -1;
        }
        //一帧图像数据大小，会根据图像格式、图像宽高计算所需内存字节大小
        int numBytes = av_image_get_buffer_size(AV_PIX_FMT_BGR24, avcodec_context->width, avcodec_context->height, 1);
        unsigned char* out_buffer = (unsigned char*)av_malloc(numBytes * sizeof(unsigned char));

        // 将rgbFrame中的数据以BGR格式存放在out_buffer中
        av_image_fill_arrays(rgbFrame->data, rgbFrame->linesize, out_buffer,
                             AV_PIX_FMT_BGR24, avcodec_context->width, avcodec_context->height, 1);
        cv::Mat img = cv::Mat(cv::Size(avcodec_context->width, avcodec_context->height), CV_8UC3);
        if (av_read_frame(pFormatCtx, av_packet) >= 0) {
            // 这里就是接收到的未解码之前的数据
            if (!(av_packet->stream_index == videoindex)) {
                qDebug() << "no video data size";
            }
            temp_ret = avcodec_send_packet(avcodec_context, av_packet);
            if (0 != temp_ret) {
                qDebug() << "dencode error\n";

                continue;
            }
            if(avcodec_receive_frame(avcodec_context, yuvFrame) != temp_ret) {
                qDebug() << "dencode to pkt error\n";
                continue;
            }
            //mp4文件中视频流使用的是h.264，帧图像为yuv420格式
            //通过sws_scale将数据转换为BGR格式
            sws_scale(swsContext,
                (const uint8_t* const*)yuvFrame->data,
                yuvFrame->linesize,
                0,
                avcodec_context->height,
                rgbFrame->data,
                rgbFrame->linesize);

            img.data = rgbFrame->data[0];

            cv::imshow("img", img);
            cv::waitKey(100);
            av_packet_unref(av_packet);
        }
        else {
            qDebug() << "no data";
        }
        if (av_packet != NULL)
            av_packet_unref(av_packet);
    }
    sws_freeContext(swsContext);


    avcodec_free_context(&avcodec_context);
    av_packet_free(&av_packet);
    if(avFrame) av_frame_free(&avFrame);
    avformat_close_input(&pFormatCtx);
    return 0;
}

```



