# camera-car
在車體上加裝相機，達到類似遙控無人機的功能

原用意在尋找掉落在床底下、桌下等地方的小東西，比如鑰匙圈之類的，原目標是利用camera-car將其推出來

由於車體有不穩定性，以及一些時間因素等等，我們沒有時間再加裝機械手臂，但基本功能皆已完成。

車體可以遙控行走，也可以藉由wifi傳輸相機拍照後的影像回來。

## 參考網站
1.拍照功能:http://atceiling.blogspot.tw/2014/04/raspberry-pi-webcam.html  --包含軟體的安裝、指令等

2.連線功能:https://sites.google.com/site/raspberypishare0918/home/di-yi-ci-qi-dong/1-6-you-xian-huo-wu-xian-dedhcp --此為DHCP透過網路線有線的連線

3.修正連線問題:https://superuser.com/questions/42460/can-you-explain-how-to-understand-what-the-iwconfig-command-displays-in-ubuntu  --提到密碼的設定必須為13個ascii字元

https://www.ptt.cc/bbs/Linux/M.1318004307.A.D42.html --編輯/etc/network/interfaces內容的部分

http://www.arthurtoday.com/2010/08/ubuntu.html --鍵入指令開啟wifi

4.SSH to raspberrypi:http://yhhuang1966.blogspot.tw/2014/02/ssh.html --putty連線到raspberry pi

5.SSH一些設定的修正:https://www.youtube.com/watch?v=RgUM8ulMfHE  --打開ssh權限(之前有被擋下來，所以特別去查)

6.使用圖形化介面ssh raspberrypi:https://www.youtube.com/watch?v=miGXUFyBvJQ  --Xming的相關設定部分

7.自走車套件組裝:http://goods.ruten.com.tw/item/show?21735010050329 ---參考自走車成品圖

http://yhhuang1966.blogspot.tw/2017/06/arduino_28.html ---底盤組裝參考

8.自走車接線:http://hophd.com/raspberry-pi-remote-car-hardware/ --硬體接線

9.自走車軟體部分:http://hophd.com/raspberry-pi-remote-car-program/  --程式是參考此為底寫的

## 使用器材

###### 1.長度20cm 10pin的母對母排線

50根(因商家要求訂購滿100才出貨)，共NT$140

###### 2.Raspberry Pi USB Camera:

    免驅動迷你USB相機，支持所有版本的Raspberry Pi

    鏡頭焦距：F6.0MM

    對焦範圍：20MM

    視頻分辨率：640 x 480
    
NT$270(含運費)

###### 3.2WD自走車套件:

    小車底盤1片

    過EMC檢測的減速電機1:48 2個

    輪子2個

    高精度20線測速碼盤2個

    萬向輪1個

    四節電池盒1隻
      
###### 4.雙H橋電機驅動模塊(L298N):

    工作模式：H橋驅動（雙路）
    
    主控芯片：L298N

自走車套件整套包含L298N，共NT$340(含運費)

###### 5.電池

包含手電筒用電池以及驅動自走車用的電池，共NT$190

## 遇到的問題與解決方法概述

#### PROBLEM 1: 因最終目標pi需要能獨自行走, 所以得讓pi有自己的ip, 再以電腦ssh進入pi來遠程操控. 

SOLUTION : 我們的想法是以pi連上電腦熱點取得ip後, 再使用ssh連入此ip.

首先使用putty, 並以serial方式進入pi.
  
輸入sudo apt-get install wpasupplicant, 確定連結熱點的功能有安裝在pi裡.
  
再來輸入sudo vim /etc/network/interfaces 編輯網路卡的使用設定.
  
在這檔案底下加上
  
auto wlan0

iface wlan0 inet manual

wpa-ssid [your ssid]

wpa-psk [your-key]
  
ssid即為wifi id, key 為密碼.
  
編輯完成後輸入sudo /etc/init.d/networking restart 重啟網路功能.
  
再來輸入 sudo ifconfig wlan0 up 啟用無線網卡,
  
最後打上 sudo dhclient -v wlan0 即連線成功.

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/wlan0.PNG?raw=true)

可看到圖中的 wlan0(無線網卡) 已有自己的ip.
 
再來用ifconfig獲取pi的ip後, 即可使用ssh登入pi的終端機畫面.
  
p.s. 1: 若wifi不是自己設置的, 可使用 sudo iwlist wlan0 scan 指令尋找附近可連線的基地台.
  
p.s. 2: 因ssh進pi的文字顯示是黑色的, 所以可以在putty設定裡將backgrond的RGB改為255, 這樣底色就會變成白的, 避免進入pi後只看見一片黑.
  
p.s. 3: 在遇到access denied的情況下, 我們這遇到的情況是pi的ssh服務未開啟, 執行 sudo raspi-config, 進入interfacing option 即可將ssh功能啟動.

p.s. 4: 後來發現只要更改interface檔，就可以更簡單的開啟wifi

但是其中密碼的部分限定一定要13字元，似乎是格式轉換的時候，如果不是13字元會導致密碼有誤

下圖是設定檔的部分

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/wifi.PNG?raw=true)

#### PROBLEM 2: 因pi外接了攝影機鏡頭, 我們必須要有方法能檢視鏡頭錄到的東西.

SOLUTION : 我們的想法是使用圖形化介面ssh pi, 這樣就能直接檢視拍到(或攝影到的)畫面了.

首先先下載xming並安裝, 之後再xming.exe右鍵進入內容, 並在目標後的內容 clipbord 後面的文字改為 -rootless, 儲存.

在要ssh進pi前先啟動xming, 這時並不會有任何畫面.
  
然後在使用putty ssh時將設定裡的 ssh/x11 裡的 enable x11 forwarding 選項打勾.
  
之後正常的以終端機介面登入後, 輸入startlxde執行, 過了一下之後就能看見xming出現了圖形化介面.
  

p.s.: 完成後補述

雖然我們原本的目標是使用圖形化介面來開啟攝影機儲存的照片，但我們後來發現只要讓電腦上的瀏覽器連線到raspberry pi也可以達到同樣的功能

因此，圖形化介面的部分就變得不是很必要，但在此還是將其補上。

以下兩張圖分別是ssh到pi後開啟圖形化介面，以及圖形化介面的展示

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/ssh.jpg?raw=true)

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/ssh1.jpg?raw=true)  
  
#### PROBLEM 3: 實現攝影功能

SOLUTION : 首先須有一台 Web camera, 並輸入sudo apt-get install fswebcam 和 sudo apt-get install motion 這兩項指令.

分別為安裝拍照功能與攝影功能的軟體.

單純的拍照, 只需輸入fswebcam [filename].[type] , 譬如像是 test.jpg這樣的指令, camera就會進行拍照, 並存在home目錄底下.

而錄影部分, 因錄影即為連續拍照, 所以必須一直更新相機所拍到的照片, 這時就得將照片一直傳至指定的port, 再至此ip觀看才行.

首先輸入 sudo vim /etc/motion/motion.conf 修改相關設定.

將 daemon 改為 on, stream_localhost 改為 off, 這樣才能從外部連線至 webcam.

再來將 /etc/default/motion 裡的 start_motion_daemon=no 改為 yes.

然後 sudo service motion start, 重啟motion功能.

最後只要在瀏覽器上網址打上 [pi的localhost ip] : 8081, 就能成功地看到webcam的攝影畫面.

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/C1.PNG?raw=true)

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/C2.PNG?raw=true)

在左上角可以看見,  瀏覽器是以 [pi的localhost ip] : 8081 的形式成功地觀看了攝影機所錄到的影像,

並在 13:46:32 至 13:47:32 (攝影機畫面的右下角), 也就是在1秒後, 更新了在鏡頭前所擺放的物體.

p.s : 一開始在進入此ip時曾經發生 access denied 的情形, 我們解決的方式和 problem 1 一樣, 進入 interfacing option 畫面, 將 camera 功能 enable 後即可成功連線.

#### PROBLEM 4: 自走車套件
SOLUTION : 我們訂購的自走車套件，現貨到的時候我們沒有仔細檢查，在開始做之後才發現馬達上有缺陷，並沒有接出正負兩極的線路讓我們能連到自走車套件上，

所以我們必須自己將兩顆馬達的正副兩極焊接上，所幸的是我們有過焊接經驗，在這方面影響並不會太大。

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/car2.jpg?raw=true)

上圖為未焊接前的自走車示意圖，是最初組裝起來的版本，在這個版本中自走車的馬達是沒有線可以接到L298N的

如下圖所示：

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/car.jpg?raw=true)

圖片中, 兩個馬達的兩個正負極, 皆沒有可以連至 L298N 的線路.

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/car3.jpg?raw=true)

上圖為焊接後馬達的線路，可以清楚看到馬達正負極已經焊好線路了。

#### PROBLEM 5: 實作自走車

SOLUTION :

![image](https://github.com/NCNU-OpenSource/camera-car/blob/master/car4.jpg?raw=true)

上圖為自走車完成圖，相機掛在前方。實際運作觀看影片展示

影片: https://drive.google.com/file/d/1fLWBfmpYYZmxTsMdCZoKF4_ddP6DUITG/view

上述網址為實作後的結果, 最後的終端機所顯示的f, r, l, b, s 分別為前進, 右轉, 左轉, 後退, 停留的指令, 這部分可由輪子的轉向可以看出.

要控制自走車, 首先要先在 /usr/bin 底下建立一個操控 L298N 的檔案, 在這取名為car2.py.

程式碼內容為:

#! /usr/bin/python
 
import RPi.GPIO as gpio

import sys
 
gpio.setwarnings(False)

gpio.setmode(gpio.BOARD)

gpio.setup(7, gpio.OUT)

gpio.setup(11, gpio.OUT)

gpio.setup(13, gpio.OUT)

gpio.setup(15, gpio.OUT)
 
if len(sys.argv)<=1:

　cmd = 'stop'

else:

　cmd = sys.argv[1]
 
print(cmd)
 
if cmd == 'B' or cmd =='b':

　gpio.output(7, True)

　gpio.output(11, False)

　gpio.output(13, True)

　gpio.output(15, False)

elif cmd == 'S' or cmd =='s':

　gpio.output(7, False)

　gpio.output(11, False)

　gpio.output(13, False)

　gpio.output(15, False)

elif cmd == 'F' or cmd =='f':

　gpio.output(7, False)

　gpio.output(11, True)

　gpio.output(13, False)

　gpio.output(15, True)

elif cmd == 'L' or cmd =='l':

　gpio.output(7, True)

　gpio.output(11, False)

　gpio.output(13, False)

　gpio.output(15, True)

elif cmd == 'R' or cmd =='r':

　gpio.output(7, False)

　gpio.output(11, True)

　gpio.output(13, True)

　gpio.output(15, False)

else:

　gpio.output(7, False)

　gpio.output(11, False)

　gpio.output(13, False)

　gpio.output(15, False)

程式內容大意為 :

根據收到的指令, 給不同的 pin 角不同的高低電位, 使得左右輪子有自己的旋轉方向, 這樣就能實現行走的功能了.
## 結論
雖然在無線遙控的時候不是很直觀，但原先預計的功能已經都完成了:

1.具有無線遙控的功能

2.車體組裝完成可以行走

3.能夠回傳拍照的影像

#### 最後, 只要將上述實現的各項功能, 各自在自走車上組合, 並打開瀏覽器監控, 就能完成我們的 camera car 了.


## 工作分配
103321023 許家愷 --負責主導工作，網路，相機，實機測試

103321036 李明皓 --負責當助手，從旁輔助排解問題，實體部分，實機測試

103321041 何承燁 --實體部分，查詢資料
