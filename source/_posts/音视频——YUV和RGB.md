---
title: 音视频——YUV和RGB
date: 2016-06-28 19:34:02
tags: [YUV,RGB]
---

### YUV

- 如果想把YUV格式像素数据变成灰度图像，只需要将U、V分量设置成128即可。这是因为U、V是图像中的经过偏置处理的色度分量。色度分量在偏置处理前的取值范围是-128至127，这时候的无色对应的是“0”值。经过偏置后色度分量取值变成了0至255，因而此时的无色对应的就是128了。
- 如果打算将图像的亮度减半，只要将图像的每个像素的Y值取出来分别进行除以2的工作就可以了。
- PSNR,PSNR取值通常情况下都在20-50的范围内，取值越高，代表两张图像越接近，反映出受损图像质量越好。公式如下：
![](http://d.hiphotos.baidu.com/baike/pic/item/b2de9c82d158ccbf4fbf06be19d8bc3eb1354136.jpg)

![](http://img.blog.csdn.net/20160117233543104)

#### YUV420P

- 一帧YUV420P像素数据一共占用w\*h\*3/2 Byte的数据。其中前w\*h Byte存储Y，接着的w\*h\*1/4 Byte存储U，最后w\*h\*1/4 Byte存储V

#### YUV444P

- 如果视频帧的宽和高分别为w和h，那么一帧YUV444P像素数据一共占用w\*h\*3 Byte的数据。其中前w\*h Byte存储Y，接着的w\*h Byte存储U，最后w\*h Byte存储V。


### RGB

#### RGB24

- 与YUV420P三个分量分开存储不同，RGB24格式的每个像素的三个分量是连续存储的。一帧宽高分别为w、h的RGB24图像一共占用w*h*3 Byte的存储空间。RGB24格式规定首先存储第一个像素的R、G、B，然后存储第二个像素的R、G、B…以此类推。类似于YUV420P的存储方式称为Planar方式，而类似于RGB24的存储方式称为Packed方式。

#### 将RGB24格式像素数据封装为BMP图像

BMP storage pixel data in opposite direction of Y-axis (from bottom to top).  

在将RGB24格式像素数据封装成BMP图像主要完成了两个工作：
1. 将RGB数据前面加上文件头。
2. 将RGB数据中每个像素的“B”和“R”的位置互换。
BMP文件是由BITMAPFILEHEADER、BITMAPINFOHEADER、RGB像素数据共3个部分构成，它的结构如下图所示。

|  BITMAPFILEHEADER  |
|  BITMAPINFOHEADER  |
|     RGB像素数据     |

```c
typedef  struct  tagBITMAPFILEHEADER  
{   
unsigned short int  bfType;       //位图文件的类型，必须为BM   
unsigned long       bfSize;       //文件大小，以字节为单位  
unsigned short int  bfReserverd1; //位图文件保留字，必须为0   
unsigned short int  bfReserverd2; //位图文件保留字，必须为0   
unsigned long       bfbfOffBits;  //位图文件头到数据的偏移量，以字节为单位  
}BITMAPFILEHEADER;   
typedef  struct  tagBITMAPINFOHEADER   
{   
long biSize;                        //该结构大小，字节为单位  
long  biWidth;                     //图形宽度以象素为单位  
long  biHeight;                     //图形高度以象素为单位  
short int  biPlanes;               //目标设备的级别，必须为1   
short int  biBitcount;             //颜色深度，每个象素所需要的位数  
short int  biCompression;        //位图的压缩类型  
long  biSizeImage;              //位图的大小，以字节为单位  
long  biXPelsPermeter;       //位图水平分辨率，每米像素数  
long  biYPelsPermeter;       //位图垂直分辨率，每米像素数  
long  biClrUsed;            //位图实际使用的颜色表中的颜色数  
long  biClrImportant;       //位图显示过程中重要的颜色数  
}BITMAPINFOHEADER; 
```

BMP采用的是小端（Little Endian）存储方式。这种存储方式中“RGB24”格式的像素的分量存储的先后顺序为B、G、R。由于RGB24格式存储的顺序是R、G、B，所以需要将“R”和“B”顺序作一个调换再进行存储。
