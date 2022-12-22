---
layout: post
title: MediaPipe Hands poke bubbles
author: [蘇言文、陳柏帆]
category: [Lecture]
tags: [ai]
---



### MediaPipe Hands poke bubbles(戳泡泡遊戲)


**期末報告描述**

一個可以透過鏡頭遊玩的戳泡泡遊戲，透過**MediaPipe**模型辨識食指的位置，在30秒內盡可能的戳破泡泡來得分。

---

**開發過程:**
1. 手掌辨識<br>
2. 繪製泡泡<br>
3. 計分系統<br>
4. 計時系統<br>
5. 開始遊戲功能 <br>

---

**執行環境:**

這次的模型框架有特別強調對硬體的要求不高，應該大部分的電腦都能運行，唯一要注意的是需要**鏡頭**才能遊玩這個遊戲。
![](https://i.imgur.com/zdY9HZd.png)

以下是使用版本供參考

* python 3.9.13
* mediapipe 0.9.01


---

### MediaPipe介紹

GitHub: [mediapipe ](https://google.github.io/mediapipe/)

Kaggle: [rkuo2000/mediapipe-pose](https://www.kaggle.com/code/rkuo2000/mediapipe-pose)

**MediaPipe** 是 Google Research 所開發的機器學習模型應用框架，支援 JavaScript、Python、C++ 等程式語言，也可以放在嵌入式平臺 (例如樹莓派等)、移動設備 ( iOS 或 Android ) 或後端伺服器

如果使用 Python 語言進行開發，MediaPipe 支援下列幾種辨識功能

1. MediaPipe Face Detection ( 人臉追蹤 )
1. MediaPipe Face Mesh ( 人臉網格 )
1. MediaPipe Hands ( 手掌偵測 )
1. MediaPipe Holistic ( 全身偵測 )
1. MediaPipe Pose ( 姿勢偵測 )
1. MediaPipe Objectron ( 物體偵測 )
1. MediaPipe Selfie Segmentation ( 人物去背 )


其中我們要用到的是**MediaPipe Hands ( 手掌偵測 )**，官網的示意圖如下。
![](https://i.imgur.com/dtPZI4v.png)


其中各個不同的手掌位置都有預設不同的代號，因為我們要判定的是食指，所以用的是8號。
![](https://i.imgur.com/MVqCJEP.png)

---

## 程式說明


程式的部分主要分成**辨識手掌**和**產生泡泡**還有**計分計時**三個部分，其中辨識手掌的部分主要參考自MediaPipe的官方文檔如下

---
### 官方文檔

```
import cv2
import mediapipe as mp
mp_drawing = mp.solutions.drawing_utils
mp_drawing_styles = mp.solutions.drawing_styles
mp_hands = mp.solutions.hands


# For webcam input:
cap = cv2.VideoCapture(0)
with mp_hands.Hands(
    model_complexity=0,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5) as hands:
  while cap.isOpened():
    success, image = cap.read()
    if not success:
      print("Ignoring empty camera frame.")
      # If loading a video, use 'break' instead of 'continue'.
      continue

    # To improve performance, optionally mark the image as not writeable to
    # pass by reference.
    image.flags.writeable = False
    image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    results = hands.process(image)

    # Draw the hand annotations on the image.
    image.flags.writeable = True
    image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)
    if results.multi_hand_landmarks:
      for hand_landmarks in results.multi_hand_landmarks:
        mp_drawing.draw_landmarks(
            image,
            hand_landmarks,
            mp_hands.HAND_CONNECTIONS,
            mp_drawing_styles.get_default_hand_landmarks_style(),
            mp_drawing_styles.get_default_hand_connections_style())
    # Flip the image horizontally for a selfie-view display.
    cv2.imshow('MediaPipe Hands', cv2.flip(image, 1))
    if cv2.waitKey(5) & 0xFF == 27:
      break
cap.release()
```


### 辨識手掌
```
import cv2
import mediapipe as mp
import random
import numpy as np
import time


mp_drawing = mp.solutions.drawing_utils
mp_drawing_styles = mp.solutions.drawing_styles
mp_hands = mp.solutions.hands

cap = cv2.VideoCapture(0)

with mp_hands.Hands(
    model_complexity=0,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5) as hands:
  while cap.isOpened():
    success, image = cap.read()
    image = cv2.flip(image,1)

    if not success:
      print("Ignoring empty camera frame.")
      continue

    image =cv2.resize(image,(640,480))
    size = image.shape
    w = size[1]
    h = size[0]

    if run :
        run = False
        rx = random.randint(50,w-50)
        ry = random.randint(50,h-100)

    image.flags.writeable = False
    image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    results = hands.process(image)
    image.flags.writeable = True
    image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)
    if results.multi_hand_landmarks:
      for hand_landmarks in results.multi_hand_landmarks:
        x = hand_landmarks.landmark[8].x * w
        y = hand_landmarks.landmark[8].y * h
        if x>rx and x<(rx+25) and y>ry and y<(ry+25):
            run = True
        mp_drawing.draw_landmarks(
            image,
            hand_landmarks,
            mp_hands.HAND_CONNECTIONS,
            mp_drawing_styles.get_default_hand_landmarks_style(),
            mp_drawing_styles.get_default_hand_connections_style())
```
---

### 產生泡泡

將泡泡的中心點設在`(rx,ry)`，當食指的位置`(x,y)`距離泡泡中心夠近時(小於泡泡半徑)，辨識為戳到泡泡。

```
if x>rx and x<(rx+25) and y>ry and y<(ry+25):
    run = True
```
當戳到泡泡時，將泡泡的中心重新設置在隨機的位置。
```
rx = random.randint(50,w-50)
ry = random.randint(50,h-100)
```
下面繪製出泡泡的圖形，用四個圓圈繪製出泡泡的模樣。

```
cv2.circle(image, (rx, ry), 25, (255, 230, 190), -1)
cv2.circle(image, (rx-10, ry-10), 3, (250, 250, 250), 4)
cv2.circle(image, (rx + 10, ry + 10), 2, (250, 250, 250), 2)
cv2.circle(image, (rx, ry), 23, (230, 224, 176), 2)
```

---


### 計分計時
按下w後，`i`變數由`False`改為`True`，代表遊戲開始。
```
i = False

if cv2.waitKey(5) == ord('w'):
    i = True
```
設定倒數計時30秒。
```
total_time = 30
```
判斷i為`True`後，開始倒數計時並執行計分。
```
if i:
    time_now = time.time()
    time_li = str(time_on - time_now)
if i and run:
    fra = fra + 1
```
顯示計分與倒數計時文字。
```
cv2.rectangle(image, (270,50),(400,10),(192, 192, 192),-1)
cv2.rectangle(image, (250,460),(400,415),(192, 192, 192),-1)
cv2.putText(image,"Point:" + str(fra) , (270, 40), cv2.FONT_HERSHEY_DUPLEX, 1, (255, 0, 0), 2, cv2.LINE_AA)
cv2.putText(image,"Time:" + time_li[:4], (250, 450), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 139), 2, cv2.LINE_AA)
```
倒數計時時間為0時，顯示最終分數
```
if time_on - time_now <= 0:
    cv2.rectangle(image, (130, 200),(550, 270),(0, 0, 0),-1)
    cv2.putText(image,"Your point is:" + str(fra) , (150,250), cv2.FONT_HERSHEY_DUPLEX, 1.5, (10, 215, 255), 2, cv2.LINE_AA)
```

---

### **完整MediaPipe Hands poke bubbles(戳泡泡遊戲)程式** <br>
程式連結: [MediaPipe_Hands_poke_bubbles.py](https://github.com/hahakevin45/AI/blob/gh-pages/code/MediaPipe_Hands_poke_bubbles.py)

```
import cv2
import mediapipe as mp
import random
import numpy as np
import time


mp_drawing = mp.solutions.drawing_utils
mp_drawing_styles = mp.solutions.drawing_styles
mp_hands = mp.solutions.hands

run=True
fra = 0
i = False
total_time =30
time_on = 1
time_now = 0
time_li = str(total_time)
cap = cv2.VideoCapture(0)

# For webcam input:
with mp_hands.Hands(
    model_complexity=0,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5) as hands:
  while cap.isOpened():
    success, image = cap.read()
    image = cv2.flip(image,1)

    if not success:
      print("Ignoring empty camera frame.")
      continue

    image =cv2.resize(image,(640,480))
    size = image.shape
    w = size[1]
    h = size[0]

    if run :
        run = False
        rx = random.randint(50,w-50)
        ry = random.randint(50,h-100)

    image.flags.writeable = False
    image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    results = hands.process(image)

    # Draw the hand annotations on the image.
    image.flags.writeable = True
    image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)
    if results.multi_hand_landmarks:
      for hand_landmarks in results.multi_hand_landmarks:
        x = hand_landmarks.landmark[8].x * w
        y = hand_landmarks.landmark[8].y * h
        if x>rx and x<(rx+25) and y>ry and y<(ry+25):
            run = True
        mp_drawing.draw_landmarks(
            image,
            hand_landmarks,
            mp_hands.HAND_CONNECTIONS,
            mp_drawing_styles.get_default_hand_landmarks_style(),
            mp_drawing_styles.get_default_hand_connections_style())

    image2 = np.zeros((400, 400, 3), np.uint8)
    image2.fill(90)
    if not i:
      cv2.rectangle(image, (200, 260),(470, 220),(192, 192, 192),-1)
      cv2.putText(image,"Press w to start" , (200, 250), cv2.FONT_HERSHEY_DUPLEX, 1, (255, 0, 0), 2, cv2.LINE_AA)
    if cv2.waitKey(5) == ord('w'):
      i = True
      time_on = time.time()+total_time
    if i:
      time_now = time.time()
      time_li = str(time_on - time_now)
    if i and run:
      fra = fra + 1

    cv2.rectangle(image, (270,50),(400,10),(192, 192, 192),-1)
    cv2.rectangle(image, (250,460),(400,415),(192, 192, 192),-1)
    cv2.putText(image,"Point:" + str(fra) , (270, 40), cv2.FONT_HERSHEY_DUPLEX, 1, (255, 0, 0), 2, cv2.LINE_AA)
    cv2.putText(image,"Time:" + time_li[:4], (250, 450), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 139), 2, cv2.LINE_AA)

    cv2.circle(image, (rx, ry), 25, (255, 230, 190), -1)
    cv2.circle(image, (rx-10, ry-10), 3, (250, 250, 250), 4)
    cv2.circle(image, (rx + 10, ry + 10), 2, (250, 250, 250), 2)
    cv2.circle(image, (rx, ry), 23, (230, 224, 176), 2)
    if time_on - time_now <= 0:
          cv2.rectangle(image, (130, 200),(550, 270),(0, 0, 0),-1)
          cv2.putText(image,"Your point is:" + str(fra) , (150,250), cv2.FONT_HERSHEY_DUPLEX, 1.5, (10, 215, 255), 2, cv2.LINE_AA)
    cv2.imshow('MediaPipe Hands', image)
    if cv2.waitKey(5) & 0xFF == 27 :
      break
    if time_on - time_now <= 0:
      print("Your point is " + str(fra) )
      time.sleep(2)
      break

cap.release()
```

---


### 成果說明


#### 影片展示: 
<iframe width=854 height=480  src="https://www.youtube.com/embed/YJ_JCDBOgiE?enablejsapi=1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen onload="onYouTubeIframeAPIReady()"></iframe>


#### 照片展示:

開始遊戲時，按下W才會開始遊戲，可以先暖身一下

<img src="https://i.imgur.com/AtdW7us.jpg" width="640" height="360" alt="遊戲結束"/><br/>

遊戲過程，辨識手部，這裡可以一次辨識兩隻手，精確度跟速度都不錯

<img src="https://i.imgur.com/SNbTB9S.jpg" width="640" height="360" alt="遊戲結束"/><br/>

遊戲結束會顯示分數

<img src="https://i.imgur.com/8mhVSrE.jpg" width="640" height="360" alt="遊戲結束"/><br/>

---
### 參考資料

[Python 與 OpenCV 加入線條圖案與文字教學](https://blog.gtwang.org/programming/opencv-drawing-functions-tutorial/)



<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

