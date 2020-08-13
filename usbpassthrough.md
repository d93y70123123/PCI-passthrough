# USB分配虛擬機  
## 環境介紹 
處理器：i7-4790  
主機板：ASUS Z97-PRO GAMER  
實體機作業系統：CentOS8  
虛擬機環境：Windows10*2  

## 先說說為什麼要做這個  
  以往在做虛擬機時，我們會分配顯卡以及USB給虛擬機使用，USB分配一直困擾著我們，所以通常都使用一張擴充USB(PCI介面)來解決。
但是總覺得能不花錢解決才是王道，可也因為這樣累到自己，所以稍微比較一下差別。
| 使用方式 | USB孔分配 | PCI擴充卡分配 | USB控制器 |
|:-:|:-:|:-:|:-:|
|費用|不用錢|要花錢|**不用錢**|
|方便度|不方便|第二方便|**最方便**|
|難易度|**簡單**|中度|困難|

解釋一下PCI擴充卡，因為一般主機不會附贈擴充卡，所以需要自己花錢購買，而且在安裝上還需要拆開主機，確認有沒有插好。  

## 原理
其實說是分配USB，但背後的原理還是分配PCI上的USB Controller，也因為這樣很大部分被主機板限制住。  

## 第一步，理清架構  
可以從bisoc或是作業系統查看USB的狀況
```bash
# lspci |grep -i usb
00:14.0 USB controller: Intel Corporation 9 Series Chipset Family USB xHCI Controller
00:1a.0 USB controller: Intel Corporation 9 Series Chipset Family USB EHCI Controller #2
00:1d.0 USB controller: Intel Corporation 9 Series Chipset Family USB EHCI Controller #1
```  
`小科普：xHCI是USB3.0、EHCI是USB2.0`  

```bash
# readlink -f /sys/bus/usb/devices/usb*
/sys/devices/pci0000:00/0000:00:1a.0/usb1
/sys/devices/pci0000:00/0000:00:1d.0/usb2
/sys/devices/pci0000:00/0000:00:14.0/usb3
/sys/devices/pci0000:00/0000:00:14.0/usb4
```

根據參考文獻的敘述，他會先將BIOS的USB3.0都關閉，這樣會比較好找出分別是屬於哪個控制器。  
但是我實測下來，發現用一般的USB隨身碟跟USB鍵盤滑鼠裝置，會出現在不同的控制上。  
舉例來說：  
  我插了一顆隨身碟，他會出現在BUS1  
  但是插鍵盤滑鼠，則會出現在BUS2  
**所以我統一用滑鼠鍵盤來測試，並且沒有關閉USB3.0**  

下面簡稱控制器分別為:1a、1d、14  

## 第二步，認識USB Controller  
本測試的主機，前面有3個USB，背面有6個USB  
前面面板
|1|2|3|
|:-:|-|-|

後面面板
|1|2|
|:-:|:-:|
|3|4|
|5|6|

預計分配
|HOST|HOST|VM1|
|:-:|-|-|

|VM1|VM1|
|:-:|:-:|
|VM2|VM2|
|VM2|VM2|

```bash
   00  00 0000  1111 1111
    \/  \_____/  \_______/
Always  EHCI #2   EHCI #1
Zero    65 4321  8765 4321
```
資料來自文獻[1]

電腦只認識0、1，所以我們要稍微代位思考一下，並轉換成16進位，為了搭配指令。
在BIOS上面查看有14組USB，但實際上在機殼出現的只有9Port，要找出每個USB分別對應的是哪一個數字就是最重要的任務。

首先我會建議把在14(3.0)上的全部變為0，這時候會發現不管插哪個孔洞，都會出現在1a跟1d。  
```bash
# setpci -s0:14.0 0xd0.W=0x0000
```

[0000]是轉換成16進位後的數值，下面會根據這4個數字分配孔位。

搭配lsusb -t，可以清楚查看該孔洞是屬於哪個BUS。 
建議搭配watch，更方便
```bash
# watch lsusb -t
Every 2.0s: lsusb -t           ab-smallsix: Thu Aug 13 21:26:28 2020

/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
```

到目前為止我們可以很清楚地發現1a在前面1、2兩個孔洞上，其餘都在1d控制器上。
所以目前為止前面兩個16進位為 0000 0000，所以前面兩個數字為00  

接下來就是找出後面兩個數字，1d控制器該如何分配  
這邊建議後面兩個數字從f0開始先一個一個數字確定，接著就是慢慢找，從0-f一個一個嘗試，其實就在排列組合而已  
把每個USB想成0000 0000 0000 0000，所以總共有16個USB孔洞  
如果我想要讓最後一組都分配到14上就會變成這樣 0000 0000 0000 1111，以此類推。  

如果怕弄壞，可以怕USB弄回3.0  
```bash
# setpci -s0:14.0 0xd0.W=0x3fff
```

## 第三步，分配USB孔到不同的控制器上
總而言之，我們有三組控制器：1a、1d、14，所以要考慮的是如何把14上的開關控制好。  
最後，設計出我們要控制的USB Controller需要用到下面的指令
```bash
# setpci -s0:14.0 0xd0.W=0x003e
```

**若有出現關於USB錯誤訊息可以嘗試：[ echo 0 > /proc/sys/kernel/printk ]，無視她



## 參考文獻
[1]HOWTO Pass Onboard USB Ports to a KVM Virtual Machine : https://sholland.org/2016/howto-pass-usb-ports-to-kvm/
