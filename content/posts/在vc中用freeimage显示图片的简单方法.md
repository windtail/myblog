---
title: "在VC中用FreeImage显示图片的简单方法"
date: 2010-11-27T22:37:00+08:00
draft: false
---

 


没什么想法，就是想找个偷懒的方法显示个图片，至少要支持bmp，最好顺带多支持几种格式类型，


本以为上网搜下就有了，结果还搞了半天。


 


没时间多写，贴个代码：


 


void test(CDC &dc)  

{  

    const char imgName[64] = ".//1252.png";  

  

    int width;  

    int height;  

  

    // 获得图像文件的类型  

    FREE\_IMAGE\_FORMAT fifmt = FreeImage\_GetFileType(imgName, 0);  

  

    // 加载此文件  

    FIBITMAP \*dib = FreeImage\_Load(fifmt, imgName,0);  

  

    if(dib == NULL) return;  

  

    // 对于不是24位的图片，强制转换成24位 , RGB


    // 为了省事  

    dib = FreeImage\_ConvertTo24Bits(dib);  

    BYTE \*pixels;  

    width = FreeImage\_GetWidth(dib);  

    height = FreeImage\_GetHeight(dib);  

      

    pixels = FreeImage\_GetBits(dib);  

  

    BITMAPINFO info;  

    memset(&info, 0, sizeof(BITMAPINFO));  

    info.bmiHeader.biSize = sizeof(BITMAPINFOHEADER);  

    info.bmiHeader.biBitCount = 24;  

    info.bmiHeader.biPlanes = 1;  

    info.bmiHeader.biWidth = width;  

    info.bmiHeader.biHeight = height;      

  

    StretchDIBits(dc.GetSafeHdc(), 0, 0, width, height, 0, 0, width, height, pixels, &info, DIB\_RGB\_COLORS, SRCCOPY);  

    FreeImage\_Unload(dib);  

}


 


注：freeimage的库参见
 http://freeimage.sourceforge.net/




