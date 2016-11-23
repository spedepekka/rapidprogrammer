---
layout: post
title: "How to access camera with OpenCV and Java"
date: "2016-11-23 09:56:40 +0200"
permalink: /:title
---

**This blog post is about plain Java, not Android**

**Complete Gradle example project at [Github][github]**

I had hard time to test my OpenCV algorithm on desktop, because the existing tutorials were not good enough and I wasn't able to see under the hood of my algorithm. So here is how I were able to show the frames on my Mac desktop.

## Transform Mat to BufferedImage

This class can transfor OpenCV Mat to BufferedImage

```java
package com.rapidprogrammer.javaopencv3swing;

import org.opencv.core.Core;
import org.opencv.core.Mat;

import java.awt.image.BufferedImage;
import java.awt.image.DataBufferByte;
import java.awt.image.WritableRaster;

public class Mat2Image {

    static{
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
    }

    Mat mat = new Mat();
    BufferedImage img;

    public Mat2Image() {
    }

    public Mat2Image(Mat mat) {
        getSpace(mat);
    }

    public void getSpace(Mat mat) {
        int type = 0;
        if (mat.channels() == 1) {
            type = BufferedImage.TYPE_BYTE_GRAY;
        } else if (mat.channels() == 3) {
            type = BufferedImage.TYPE_3BYTE_BGR;
        }
        this.mat = mat;
        int w = mat.cols();
        int h = mat.rows();
        if (img == null || img.getWidth() != w || img.getHeight() != h || img.getType() != type)
            img = new BufferedImage(w, h, type);
    }

    BufferedImage getImage(Mat mat){
        getSpace(mat);
        WritableRaster raster = img.getRaster();
        DataBufferByte dataBuffer = (DataBufferByte) raster.getDataBuffer();
        byte[] data = dataBuffer.getData();
        mat.get(0, 0, data);
        return img;
    }
}
```

## JFrame to show the camera feed with OpenCV

This class contains the main for the application.

```java
package com.rapidprogrammer.javaopencv3swing;

import java.awt.EventQueue;
import java.awt.Graphics;
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.border.EmptyBorder;

public class MyFrame extends JFrame {
    private JPanel contentPane;

    public static void main(String[] args) {
        EventQueue.invokeLater(new Runnable() {
            public void run() {
                try {
                    MyFrame frame = new MyFrame();
                    frame.setVisible(true);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
    }

    public MyFrame() {
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setBounds(0, 0, 1280, 720);
        contentPane = new JPanel();
        contentPane.setBorder(new EmptyBorder(0, 0, 0, 0));
        setContentPane(contentPane);
        contentPane.setLayout(null);

        new MyThread().start();
    }

    VideoCap videoCap = new VideoCap();

    public void paint(Graphics g){
        g = contentPane.getGraphics();
        g.drawImage(videoCap.getOneFrame(), 0, 0, this);
    }

    class MyThread extends Thread{
        @Override
        public void run() {
            for (;;){
                repaint();
                try { Thread.sleep(30);
                } catch (InterruptedException e) {    }
            }
        }
    }
}
```

## Simple Java class to open OpenCV VideoCapture

```java
package com.rapidprogrammer.javaopencv3swing;

import org.opencv.core.Core;
import org.opencv.videoio.VideoCapture;

import java.awt.image.BufferedImage;

public class VideoCap {

    static{
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
    }

    VideoCapture cap;
    Mat2Image mat2Img = new Mat2Image();

    VideoCap(){
        cap = new VideoCapture();
        cap.open(0);
    }

    BufferedImage getOneFrame() {
        cap.read(mat2Img.mat);
        return mat2Img.getImage(mat2Img.mat);
    }
}
```

## Gradle configuration

```
apply plugin: 'java'
apply plugin: 'application'

// Modify the OpenCV folder is needed
def myOpenCV = '/usr/local/opt/opencv3/share/OpenCV/java/'

dependencies {
    // OpenCV
    // If you don't have OpenCV jar in the same folder, set the folder here
    compile fileTree(dir: myOpenCV, include: ['*.jar'])
}

sourceCompatibility = "1.7"
targetCompatibility = "1.7"

def myMainClass = 'com.rapidprogrammer.javaopencv3swing.MyFrame'
// Set the main class to gradle run
mainClassName = myMainClass

// Pass the path to gradle run
run {
    systemProperty "java.library.path", myOpenCV
}
```

## How to run this OpenCV Gradle project

`./gradlew run`

Run tasks comes from application-plugin and it requires the main class is defined `mainClassName = 'com.example.main'`.

Remember to set the OpenCV library paths to match your OpenCV installation.

**Complete Gradle example project at [Github][github]**

[github]: https://github.com/spedepekka/java-opencv3-swing

