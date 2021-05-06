---
title: Java后端生成二维码
date: 2020-11-20
tags: [java]
categories: Java
---

搬运博客:

[生成二维码(java后端)](https://blog.csdn.net/zhang_z_ming/article/details/84027480)

> 有一个需求
>
> 就是实现 放票 抢票 检票 的功能

### 生成二维码

> 先引入两个jar包

```java
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>javase</artifactId>
    <version>3.3.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.google.zxing/core -->
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>core</artifactId>
    <version>3.3.1</version>
</dependency>
```

> 生成方法如下

```java
package com.payplatform.util;

import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import javax.imageio.ImageIO;
import com.google.zxing.BarcodeFormat;
import com.google.zxing.Binarizer;
import com.google.zxing.BinaryBitmap;
import com.google.zxing.EncodeHintType;
import com.google.zxing.LuminanceSource;
import com.google.zxing.MultiFormatReader;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.NotFoundException;
import com.google.zxing.Result;
import com.google.zxing.WriterException;
import com.google.zxing.client.j2se.BufferedImageLuminanceSource;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.common.HybridBinarizer;

public class QRCodeUtil {
	//二维码颜色  
    private static final int BLACK = 0xFF000000;  
    //二维码颜色  
    private static final int WHITE = 0xFFFFFFFF;  
  
    public static void main(String[] args) throws Exception {
		zxingCodeCreate("http://www.baidu.com", 300, 300, "D:/qrcode.jpg", "jpg");
//    	zxingCodeAnalyze("D:/qrcode.jpg");
	}
    /** 
     * 生成二维码
     * @param text    二维码内容
     * @param width    二维码宽 
     * @param height    二维码高 
     * @param outPutPath    二维码生成保存路径 
     * @param imageType     二维码生成格式 
     */  
    public static void zxingCodeCreate(String text, int width, int height, String outPutPath, String imageType){  
        Map<EncodeHintType, String> his = new HashMap<EncodeHintType, String>();  
        //设置编码字符集  
        his.put(EncodeHintType.CHARACTER_SET, "utf-8");  
        try {  
            //1、生成二维码  
            BitMatrix encode = new MultiFormatWriter().encode(text, BarcodeFormat.QR_CODE, width, height, his);  
              
            //2、获取二维码宽高  
            int codeWidth = encode.getWidth();  
            int codeHeight = encode.getHeight();  
              
            //3、将二维码放入缓冲流  
            BufferedImage image = new BufferedImage(codeWidth, codeHeight, BufferedImage.TYPE_INT_RGB);  
            for (int i = 0; i < codeWidth; i++) {  
                for (int j = 0; j < codeHeight; j++) {  
                    //4、循环将二维码内容定入图片  
                    image.setRGB(i, j, encode.get(i, j) ? BLACK : WHITE);  
                }  
            }  
            File outPutImage = new File(outPutPath);  
            //如果图片不存在创建图片  
            if(!outPutImage.exists())  
                outPutImage.createNewFile();  
            //5、将二维码写入图片  
            ImageIO.write(image, imageType, outPutImage);  
        } catch (WriterException e) {  
            e.printStackTrace();  
            System.out.println("二维码生成失败");  
        } catch (IOException e) {  
            e.printStackTrace();  
            System.out.println("生成二维码图片失败");  
        }  
    }  
      
    /** 
     * 二维码解析 
     * @param analyzePath    二维码路径 
     * @return 
     * @throws IOException 
     */  
    @SuppressWarnings({ "rawtypes", "unchecked" })  
    public static String zxingCodeAnalyze(String analyzePath) throws Exception{  
        MultiFormatReader formatReader = new MultiFormatReader();  
        String resultStr = null;  
        try {  
            File file = new File(analyzePath);  
            if (!file.exists())  
            {  
                return "二维码不存在";  
            }  
            BufferedImage image = ImageIO.read(file);  
            LuminanceSource source = new BufferedImageLuminanceSource(image);  
            Binarizer binarizer = new HybridBinarizer(source);    
            BinaryBitmap binaryBitmap = new BinaryBitmap(binarizer);  
            Map hints = new HashMap();  
            hints.put(EncodeHintType.CHARACTER_SET, "UTF-8");  
            Result result = formatReader.decode(binaryBitmap, hints); 
            resultStr = result.getText();
        } catch (NotFoundException e) {  
            e.printStackTrace();  
        }    
        System.out.println(resultStr);
        return resultStr;  
    }
}


```

