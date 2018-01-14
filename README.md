# camera-car
94有相機的車。

用意在尋找掉落在床底下、桌下等地方的小東西，比如鑰匙圈之類的，目標是利用camera-car將其推出來

## 參考網站
1.拍照功能:http://atceiling.blogspot.tw/2014/04/raspberry-pi-webcam.html

2.連線功能:https://sites.google.com/site/raspberypishare0918/home/di-yi-ci-qi-dong/1-6-you-xian-huo-wu-xian-dedhcp

3.修正連線問題:https://superuser.com/questions/42460/can-you-explain-how-to-understand-what-the-iwconfig-command-displays-in-ubuntu    

https://www.ptt.cc/bbs/Linux/M.1318004307.A.D42.html

http://www.arthurtoday.com/2010/08/ubuntu.html

4.SSH to raspberrypi:http://yhhuang1966.blogspot.tw/2014/02/ssh.html

5.SSH一些設定的修正:https://www.youtube.com/watch?v=RgUM8ulMfHE

6.使用圖形化介面ssh raspberrypi:https://www.youtube.com/watch?v=miGXUFyBvJQ


## 使用器材






## 遇到的問題與解決方法概述

Problem 1: 因最終目標pi需要能獨自行走, 所以得讓pi有自己的ip, 再以電腦ssh進入pi來遠程操控. 

Solution : 我們的想法是以pi連上電腦熱點取得ip後, 再使用ssh連入此ip.

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
 
再來用ifconfig獲取pi的ip後, 即可使用ssh登入pi的終端機畫面.
  
p.s. 1: 若wifi不是自己設置的, 可使用 sudo iwlist wlan0 scan 指令尋找附近可連線的基地台.
  
p.s. 2: 因ssh進pi的文字顯示是黑色的, 所以可以在putty設定裡將backgrond的RGB改為255, 這樣底色就會變成白的, 避免進入pi後只看見一片黑.
  
p.s. 3: 在遇到access denied的情況下, 我們這遇到的情況是pi的ssh服務未開啟, 執行 sudo raspi-config, 進入interfacing option 即可將ssh功能啟動.
  
  
 
Problem 2: 因pi外接了攝影機鏡頭, 我們必須要有方法能檢視鏡頭錄到的東西.
  
Soluton : 我們的想法是使用圖形化介面ssh pi, 這樣就能直接檢視拍到(或攝影到的)畫面了.

首先先下載xming並安裝, 之後再xming.exe右鍵進入內容, 並在目標後的內容 clipbord 後面的文字改為 -rootless, 儲存.

在要ssh進pi前先啟動xming, 這時並不會有任何畫面.
  
然後在使用putty ssh時將設定裡的 ssh/x11 裡的 enable x11 forwarding 選項打勾.
  
之後正常的以終端機介面登入後, 輸入startlxde執行, 過了一下之後就能看見xming出現了圖形化介面.
  
  
  
  
