---
layout: post
title: "Qt QImage shows weird, distorted, angled image from OpenCV"
date: "2016-11-09 14:17:07 +0200"
permalink: /:title
---

Give the step from OpenCV `Mat` to `QImage` and you should be fine.

Broken

```cpp
QImage plateImage = QImage((const unsigned char*)(mymat.data), mymat.cols, mymat.rows, QImage::Format_RGB888)
```

Fixed

```cpp
QImage plateImage = QImage((const unsigned char*)(mymat.data), mymat.cols, mymat.rows, mymat.step, QImage::Format_RGB888)
```

I found the answer from [this][stackoverflow-answer] StackOverflow thread.

[stackoverflow-answer]: http://stackoverflow.com/questions/39057168/qwidget-draws-distorted-angled-qimage
