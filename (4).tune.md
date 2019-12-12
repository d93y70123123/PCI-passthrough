# 改善消耗資源  
由於我們的系統碟是打算直接放置在SSD上，所以必須要想辦法減少系統碟在SSD上耗費的容量以及減少讀寫的動作。  
所以這邊直接把系統裡面使用者的部分切割開來，放置在傳統硬碟，這樣有什麼好處：
1. SSD耗費容量較少  
2. 減少SSD讀寫情況  
3. 可延長SSD壽命  
當然，如果要更保險可以加上磁碟陣列。  

## 移動使用者
```diff
- 這邊用虛擬機做示範
```

#### 1. 先建立儲存用的虛擬磁碟
```bash
[root@KVM ~]# cd /vm_data/img/
[root@KVM img]# ll
總計 98289232
-rw-r--r--. 1 qemu qemu 100648026112 12月  2 22:31 win10-storage.img
-rw-r--r--. 1 root root       203008 12月  1 17:59 windows10.img
[root@KVM img]# qemu-img create -f qcow2 windows10-storage.img 400G
Formatting 'windows10-storage.img', fmt=qcow2 size=429496729600 cluster_size=65536 lazy_refcounts=off refcount_bits=16
[root@KVM img]#
```

#### 2. 加入儲存碟
可以選擇用Virt Manage或是用指令加入，這邊使用Virt Manage  
**加入硬碟**  
![addstorage](http://120.114.142.50/img2/1.addstorage.png)  
**選擇硬碟**  
![addstorage](http://120.114.142.50/img2/2.addstorage2.png)  
![addstorage](http://120.114.142.50/img2/3.addstorage3.png)  
**完成後**  
![addstorage](http://120.114.142.50/img2/4.virtio.png)  

#### 3. 磁碟管理
先在Windows內新增磁區  
**磁碟管理**  
![adddisk](http://120.114.142.50/img2/5.disk.png)  
**分配磁碟**  
![adddisk](http://120.114.142.50/img2/6.disk2.png)  
**新增磁碟管理**  
因為不會分割所以基本上一直下一步就好了  
![adddisk](http://120.114.142.50/img2/7.disk3.png)  
![adddisk](http://120.114.142.50/img2/8.disk4.png)  
![adddisk](http://120.114.142.50/img2/9.disk5.png)  
**選擇磁區代號**  
這邊設置在 [D] ，若這時磁碟代號已經被使用的話，可以透過更換其磁區代號來解決  
![adddisk](http://120.114.142.50/img2/10.disk6.png)  
下一步  
![adddisk](http://120.114.142.50/img2/11.disk7.png)
**更換磁碟代號**  
將CD的磁碟代號改為[E]
![adddisk](http://120.114.142.50/img2/12.changecd.png)  
![adddisk](http://120.114.142.50/img2/13.changecd2.png)  
**安裝驅動**  
![adddisk](http://120.114.142.50/img2/14.driver.png)  

#### 4. 進入救援模式  
要移動使用者之前必須要先進入到類似救援模式的環境，最後使用命令提示字元操作，照著下面標題點選：  
1. 設定  
![set](http://120.114.142.50/img2/15.tocmd.png)  
2. 更新與安全性  
![update&security](http://120.114.142.50/img2/16.tocmd2.png)  
3. 復原  
![restore](http://120.114.142.50/img2/17.tocmd3.png)  
4. 立即重新啟動  
![reboot](http://120.114.142.50/img2/18.tocmd4.png)  
5. 疑難排解  
![question](http://120.114.142.50/img2/19.tocmd5.png)  
6.進階選項
![adv](http://120.114.142.50/img2/20.tocmd6.png)
7. 命令提示字元  
![cmd](http://120.114.142.50/img2/21.tocmd7.png)  
8. 選擇語系  
![language](http://120.114.142.50/img2/22.tocmd8.png)  
9. 疑難排解  
![question](http://120.114.142.50/img2/22.tocmd8.png)  
10. 命令提示字元  
![cmd](http://120.114.142.50/img2/23.tocmd9.png)  

#### 5. 安裝磁碟驅動~  
由於是虛擬機的問題，所以必須要載入硬碟的驅動，所以第一個步驟是先找到光碟機並安裝  
*這邊如果光碟機內是Windows10的iso，可以利用virsh attach將iso替換成Virtio  

**找到光碟機**  
指令步驟：diskpart  >>  list volume  >> exit  
如下圖可以
![diskpart](http://120.114.142.50/img2/25.diskpart.png)  

**安裝驅動**  
上面可以看到光碟機的磁碟代號是 [D]  
驅動的位置：D:\amd64\w10\viostor.inf  
![drive](http://120.114.142.50/img2/26.drive.png)  
要安裝驅動時，必須要在 [X:] 這個目錄下，因為安裝的程式放在這  
指令為：drvload.exe D:\amd64\w10\viostor.inf  
![installdriver](http://120.114.142.50/img2/27.installdrive.png)  

**重新檢查磁碟**  
這時候可以發現跟我們剛剛設置的磁碟代號稍微不同，需要小心並記得個別的代號  
C槽的代號：  C => C  
光碟機的代號：E => D  
儲存碟的代號：D => E  
![diskcheck](http://120.114.142.50/img2/28.disklist.png)

#### 6. 移動使用者囉~~  
接下來要把 [C:] 的Users移動到D:去，剛剛說 [D:] 在救援模式內的代號變為[E]，所以這邊要把Users複製到E:  
  
**C:\Users**  
![Users](http://120.114.142.50/img2/29.Users.png)  

**複製Users**  
指令使用：xcopy C:\Users E:\Users /e /v /i /g /h /k /o /x /b /c  
![copy](http://120.114.142.50/img2/30.xcopy.png)  
看一下有沒有成功複製  
![copy](http://120.114.142.50/img2/31.xcopy2.png)  

**刪除Users**  
成功複製過去就可以刪除原有的Users，若是想留條後路的話，可以將Users改名就好，確認可以正常關機再刪除  
指令使用：rmdir /s /q C:\Users
![delUsers](http://120.114.142.50/img2/33.deleteUsers2.png)  

**連結D槽的Users**  
接下來要做個假連結連接到D槽的Users，讓系統不會發現這個目錄有問題  
```diff
- 這邊需要非常小心，做錯就不能開機了
```
剛剛上面確認的是 D=>E ，但是這邊要連結的是 "實際上" 的D槽，所以在指令的部分是連接到D，而不是E。  
使用指令：mklink /J Users D:\Users
![link](http://120.114.142.50/img2/34.link.png)

**關閉視窗&重開機**  
![reboot](http://120.114.142.50/img2/35.reboot.png)

#### 7. 開機檢查囉~  
![boot](http://120.114.142.50/img2/36.boot.png)  
![boot](http://120.114.142.50/img2/37.boot2.png)  
![boot](http://120.114.142.50/img2/38.boot3.png)  

##  Windows磁碟優化  
**關閉磁碟的索引，減少容量使用**  
![index](http://120.114.142.50/img3/1.index.png)  
![index](http://120.114.142.50/img3/2.index2.png)  
![index](http://120.114.142.50/img3/3.index3.png)  
![index](http://120.114.142.50/img3/4.index4.png)  
![skip](http://120.114.142.50/img3/5.index5.png)  
`D槽也可以這樣做~`  

**服務關閉**  
```diff
- 記得都要按右下角的套用喔~ 確保有調整到，不要做白工  
```  
1. 先搜尋'服務'  
![search](http://120.114.142.50/img3/6.service.png)  
2. 關閉使用者體驗回報  
![Users](http://120.114.142.50/img3/7.Users.png)  
![Users](http://120.114.142.50/img3/8.Users2.png)  
3. 關閉Superfetch，現在改名為SysMain  
![SysMain](http://120.114.142.50/img3/9.SysMain.png)  
![SysMain](http://120.114.142.50/img3/10.SysMain2.png)  
4. 關閉Windows Search  
![search](http://120.114.142.50/img3/11.Search.png)  
![search](http://120.114.142.50/img3/12.Search2.png)  

## 大功告成~~~
