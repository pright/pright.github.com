---
layout: post
title: "RGB和YUV/YCbCr的转换"
description: ""
category: ""
tags: [DTV]
---
{% include JB/setup %}

YUV和YCbCr的差异和关系，摘自[YUV Wiki](http://en.wikipedia.org/wiki/YUV)和[YCbCr Wiki](http://en.wikipedia.org/wiki/YCbCr)。
> The scope of the terms Y'UV, YUV, YCbCr, YPbPr, etc., is sometimes ambiguous and overlapping. Historically, the terms YUV and Y'UV were used for a specific analog encoding of color information in television systems, while YCbCr was used for digital encoding of color information suited for video and still-image compression and transmission such as MPEG  and JPEG . Today, the term YUV is commonly used in the computer industry to describe file-formats that are encoded using YCbCr. 
Y'UV is often and mistakenly used as the term for YCbCr . However, they are different formats. Y'UV is an analog system with scale factors different than the digital Y'CbCr system. 
In digital video/image systems, Y'CbCr is the most common way to express color in a way suitable for compression/transmission. The confusion stems from computer implementations and text-books erroneously using the term YUV where Y'CbCr would be correct. 
Y'CbCr is often confused with the YUV  color space, and typically the terms YCbCr and YUV are used interchangeably, leading to some confusion; when referring to signals in video or digital form, the term "YUV" mostly means "Y'CbCr". 

关键的一句： 
> These formulae use the more traditional model of Y'UV, which is used for analog PAL  equipment; digital PAL  and digital NTSC  HDTV  do not use YUV but YCbCr. 

转换公式： 

```
YUV<-->RGB 
R, G, B in [0..1] 
Y in [0..1] 
U in [-0.436..0.436] 
V in [-0.615..0.615] 
Y'= 0.299*R' + 0.587*G' + 0.114*B' 
U'= -0.147*R' - 0.289*G' + 0.436*B' = 0.492*(B'- Y') 
V'= 0.615*R' - 0.515*G' - 0.100*B' = 0.877*(R'- Y') 
R' = Y' + 1.140*V' 
G' = Y' - 0.394*U' - 0.581*V' 
B' = Y' + 2.032*U' 
```

```
YCbCr<-->RGB （ITU-R BT.601数字视频标准，用于SDTV。HDTV使用ITU-R BT.709标准，采取不同的参数） 
R, G, B in [0..255] 
Y' in [16..235] 
Cb', Cr' in [16..240] 
Y’ = 0.257*R' + 0.504*G' + 0.098*B' + 16 
Cb' = -0.148*R' - 0.291*G' + 0.439*B' + 128 
Cr' = 0.439*R' - 0.368*G' - 0.071*B' + 128 
R' = 1.164*(Y’-16) + 1.596*(Cr'-128) 
G' = 1.164*(Y’-16) - 0.813*(Cr'-128) - 0.392*(Cb'-128) 
B' = 1.164*(Y’-16) + 2.017*(Cb'-128) 
```

由于转换公式里都是浮点计算，为了提高转换效率，通常会使用整型值，尤其是定点数的近似值： 

```
Basic Transform 
 Y' = 66 * R + 129 * G + 25 *B 
 U = -38 * R - 74 * G + 112 * B 
 V = 112 * R - 94 * G - 18 * B 
Scale down to 8 bits with rounding 
Y' = (Y' + 128) > > 8 
U = (U + 128) > > 8 
V = (V + 128) > > 8 
Shift values 
Y' + = 16 
U + = 128 
V + = 128 
```

这样获得的实际上是YCbCr的值，RGB均为8-bit 
源码参考：（来自Nagareshwar Talekar的VideoNet源码，实际上是rgb24和ycrcb420的转换） 

```
////////////////////////////////////////////////////////////////////////////
//
//
// Project : VideoNet version 1.1.
// Description : Peer to Peer Video Conferencing over the LAN.
// Author :Nagareshwar Y Talekar ( nsry2002@yahoo.co.in)
// Date : 15-6-2004.
//
//
// File description : 
// Name : convert.cpp
// Details : Conversion routine from RGB24 to YUV420 & YUV420 to RGB24.
//
///////////////////////////////////////////////////////////////////////////// 
#include "convert.h" 
// Conversion from RGB to YUV420
int RGB2YUV_YR[256], RGB2YUV_YG[256], RGB2YUV_YB[256];
int RGB2YUV_UR[256], RGB2YUV_UG[256], RGB2YUV_UBVR[256];
int RGB2YUV_VG[256], RGB2YUV_VB[256]; 
// Conversion from YUV420 to RGB24
static long int crv_tab[256];
static long int cbu_tab[256];
static long int cgu_tab[256];
static long int cgv_tab[256];
static long int tab_76309[256];
static unsigned char clp[1024];//for clip in CCIR601 
//
// Table used for RGB to YUV420 conversion
//
void InitLookupTable()
{
    int i; 
    for (i = 0; i < 256; i++) RGB2YUV_YR = (float)65.481 * (i<<8);
    for (i = 0; i < 256; i++) RGB2YUV_YG = (float)128.553 * (i<<8);
    for (i = 0; i < 256; i++) RGB2YUV_YB = (float)24.966 * (i<<8);
    for (i = 0; i < 256; i++) RGB2YUV_UR = (float)37.797 * (i<<8);
    for (i = 0; i < 256; i++) RGB2YUV_UG = (float)74.203 * (i<<8);
    for (i = 0; i < 256; i++) RGB2YUV_VG = (float)93.786 * (i<<8);
    for (i = 0; i < 256; i++) RGB2YUV_VB = (float)18.214 * (i<<8);
    for (i = 0; i < 256; i++) RGB2YUV_UBVR = (float)112 * (i<<8);
} 
//
// Convert from RGB24 to YUV420
//
int ConvertRGB2YUV(int w,int h,unsigned char *bmp,unsigned int *yuv)
{ 
    unsigned int *u,*v,*y,*uu,*vv;
    unsigned int *pu1,*pu2,*pu3,*pu4;
    unsigned int *pv1,*pv2,*pv3,*pv4;
    unsigned char *r,*g,*b;
    int i,j; 
    uu=new unsigned int[w*h];
    vv=new unsigned int[w*h]; 
    if(uu==NULL || vv==NULL)
        return 0; 
    y=yuv;
    u=uu;
    v=vv; 
    // Get r,g,b pointers from bmp image data....
    r=bmp;
    g=bmp+1;
    b=bmp+2; 
    //Get YUV values for rgb values... 
    for(i=0;i<h;i++)
    {
        for(j=0;j<w;j++)
        {
            *y++=( RGB2YUV_YR[*r] +RGB2YUV_YG[*g]+RGB2YUV_YB[*b]+1048576)>>16;
            *u++=(-RGB2YUV_UR[*r] -RGB2YUV_UG[*g]+RGB2YUV_UBVR[*b]+8388608)>>16;
            *v++=( RGB2YUV_UBVR[*r]-RGB2YUV_VG[*g]-RGB2YUV_VB[*b]+8388608)>>16; 
            r+=3;
            g+=3;
            b+=3;
        } 
    } 

    // Now sample the U & V to obtain YUV 4:2:0 format 
    // Sampling mechanism...
    /* @ -> Y
# -> U or V

@ @ @ @
# #
@ @ @ @
@ @ @ @
# #
@ @ @ @ 
     */ 
    // Get the right pointers...
    u=yuv+w*h;
    v=u+(w*h)/4; 
    // For U
    pu1=uu;
    pu2=pu1+1;
    pu3=pu1+w;
    pu4=pu3+1; 
    // For V
    pv1=vv;
    pv2=pv1+1;
    pv3=pv1+w;
    pv4=pv3+1; 
    // Do sampling....
    for(i=0;i<h;i+=2)
    {
        for(j=0;j<w;j+=2)
        {
            *u++=(*pu1+*pu2+*pu3+*pu4)>>2;
            *v++=(*pv1+*pv2+*pv3+*pv4)>>2; 
            pu1+=2;
            pu2+=2;
            pu3+=2;
            pu4+=2; 
            pv1+=2;
            pv2+=2;
            pv3+=2;
            pv4+=2;
        }
        pu1+=w;
        pu2+=w;
        pu3+=w;
        pu4+=w; 
        pv1+=w;
        pv2+=w;
        pv3+=w;
        pv4+=w;
    } 
    delete uu;
    delete vv; 
    return 1;
} 



//
//Initialize conversion table for YUV420 to RGB
//
void InitConvertTable()
{
    long int crv,cbu,cgu,cgv;
    int i,ind; 

    crv = 104597; cbu = 132201; /* fra matrise i global.h */
    cgu = 25675; cgv = 53279;

    for (i = 0; i < 256; i++) {
        crv_tab = (i-128) * crv;
        cbu_tab = (i-128) * cbu;
        cgu_tab = (i-128) * cgu;
        cgv_tab = (i-128) * cgv;
        tab_76309 = 76309*(i-16);
    }

    for (i=0; i<384; i++)
        clp =0;
    ind=384;
    for (i=0;i<256; i++)
        clp[ind++]=i;
    ind=640;
    for (i=0;i<384;i++)
        clp[ind++]=255;
} 
//
// Convert from YUV420 to RGB24
//
void ConvertYUV2RGB(unsigned char *src0,unsigned char *src1,unsigned char *src2,unsigned char *dst_ori,
        int width,int height)
{
    int y1,y2,u,v; 
    unsigned char *py1,*py2;
    int i,j, c1, c2, c3, c4;
    unsigned char *d1, *d2; 
    py1=src0;
    py2=py1+width;
    d1=dst_ori;
    d2=d1+3*width;
    for (j = 0; j < height; j += 2) { 
        for (i = 0; i < width; i += 2) { 
            u = *src1++;
            v = *src2++; 
            c1 = crv_tab[v];
            c2 = cgu_tab;
            c3 = cgv_tab[v];
            c4 = cbu_tab; 
            //up-left
            y1 = tab_76309[*py1++];
            *d1++ = clp[384+((y1 + c1)>>16)]; 
            *d1++ = clp[384+((y1 - c2 - c3)>>16)];
            *d1++ = clp[384+((y1 + c4)>>16)]; 
            //down-left
            y2 = tab_76309[*py2++];
            *d2++ = clp[384+((y2 + c1)>>16)]; 
            *d2++ = clp[384+((y2 - c2 - c3)>>16)];
            *d2++ = clp[384+((y2 + c4)>>16)]; 
            //up-right
            y1 = tab_76309[*py1++];
            *d1++ = clp[384+((y1 + c1)>>16)]; 
            *d1++ = clp[384+((y1 - c2 - c3)>>16)];
            *d1++ = clp[384+((y1 + c4)>>16)]; 
            //down-right
            y2 = tab_76309[*py2++];
            *d2++ = clp[384+((y2 + c1)>>16)]; 
            *d2++ = clp[384+((y2 - c2 - c3)>>16)];
            *d2++ = clp[384+((y2 + c4)>>16)];
        }
        d1 += 3*width;
        d2 += 3*width;
        py1+= width;
        py2+= width;
    } 
} 
```

