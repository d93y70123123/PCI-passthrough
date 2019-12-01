# 建置虛擬機

## create network interface  
bridge or dhcp  

## 開啟虛擬機管理員  
![vmaneger](http://120.114.142.50/img/1.%e9%96%8b%e5%95%9fVM%e7%ae%a1%e7%90%86%e5%93%a1.png)

## 新建虛擬硬碟  
需要系統碟跟儲存碟：  
系統碟 => 100G  
儲存碟 => 400G  
```bash
[root@KVM ~]# qemu-img create -f qcow2 /original/windows10.img 100G
Formatting '/original/windows10.img', fmt=qcow2 size=107374182400 cluster_size=65536 lazy_refcounts=off refcount_bits=16

[root@KVM ~]# qemu-img create -f qcow2 /vm_data/img/windows10.img 400G
Formatting '/vm_data/img/windows10.img', fmt=qcow2 size=429496729600 cluster_size=65536 lazy_refcounts=off refcount_bits=16
```

## 選擇建立新的虛擬機  
總共有五個步驟  
1. 選擇匯入的方法  
![createvm](http://120.114.142.50/img/2.%e5%bb%ba%e7%ab%8b%e6%96%b0%e7%9a%84%e8%99%9b%e6%93%ac%e6%a9%9f.png)  
2. 選擇作業系統的ISO，並輸入下方的作業系統名稱  
![chooseiso](http://120.114.142.50/img/4.%e4%b8%8b%e6%96%b9%e8%bc%b8%e5%85%a5%e4%bd%9c%e6%a5%ad%e7%b3%bb%e7%b5%b1%e5%90%8d%e7%a8%b1.png)  
3.設定記憶體和CPU  
![setmemcpu](http://120.114.142.50/img/5.%e8%a8%ad%e5%ae%9a%e8%a8%98%e6%86%b6%e9%ab%94CPU.png)  
4. 選擇使用自己建立的虛擬硬碟  
![virtualHDD](http://120.114.142.50/img/6.%e9%81%b8%e6%93%87%e8%87%aa%e5%b7%b1%e5%bb%ba%e7%ab%8b%e7%9a%84%e8%99%9b%e6%93%ac%e7%a1%ac%e7%a2%9f.png) 
![virtualDDD2](http://120.114.142.50/img/7.%e9%81%b8%e6%93%87%e8%99%9b%e6%93%ac%e7%a1%ac%e7%a2%9f.png)  
5. 輸入虛擬機名稱並選擇自訂組態  
![new name](http://120.114.142.50/img/8.%e8%bc%b8%e5%85%a5%e5%90%8d%e7%a8%b1%e4%b8%a6%e9%81%b8%e6%93%87%e7%b5%84%e8%87%aa%e8%a8%82%e6%85%8b.png)



## 設定虛擬機資源  
讓我們一個功能一個功能分別設定~  
```diff
- 調整完後記得按下套用阿~~
```

1. 先設定晶片組和韌體  
要選擇i440FX和UEFI  
![setuefi](http://120.114.142.50/img/9.%e9%81%b8%e6%93%87%e6%9e%b6%e6%a7%8b%e8%b7%9fUEFI.png)  
2. CPU使用實體核心  
CPU的model使用host-passthrough  
3. 檢查記憶體  
![checkmem](http://120.114.142.50/img/10.%e7%a2%ba%e8%aa%8d%e8%a8%98%e6%86%b6%e9%ab%94.png)  
4. 調整開機順序  
因等等要安裝作業系統，所以這邊可以先將CD調為第一順位  
![boot](http://120.114.142.50/img/11.%e7%a2%ba%e8%aa%8d%e9%96%8b%e6%a9%9f%e9%a0%86%e5%ba%8f.png)
5. check disk & CD/DVD HDD select virt-io CD/DVD select SATA
硬碟設定為使用virtio介面  
CD則使用SATA  
6. 選擇網路介面  
選擇bridge(通常預設就會是bridge了)  
7. 刪除不會用到的裝置  
除了上述設定到的裝置，其餘都可以刪除  
8. 增加顯示卡以及USB等PCI裝置  
![addcard](http://120.114.142.50/img/12.%e9%81%b8%e6%93%87%e9%a1%af%e7%a4%ba%e5%8d%a1.png)
![addpci](http://120.114.142.50/img/14.%e5%88%aa%e9%99%a4%e4%b8%8d%e5%bf%85%e8%a6%81%e7%9a%84%e8%a3%9d%e7%bd%ae.png)
9. 接下來就可以按下左上角的[開始安裝了]~  


## install WINDOWS10  
1. 按下立即安裝  
![install](http://120.114.142.50/img/15.%e9%96%8b%e5%a7%8b%e5%ae%89%e8%a3%9dwin10.png)
2. 選擇專業版  
![version](http://120.114.142.50/img/16.%e9%81%b8%e6%93%87%e5%b0%88%e6%a5%ad%e7%89%88.png)
3. 抽換ISO檔  
現在的狀態下我們使用的硬碟是Windows10的ISO檔，但是這樣會沒有驅動，所以沒辦法安裝  
```bash
[root@KVM ~]# virsh attach-disk windows10 /vm_data/iso/virtio-win.iso sdb --targetbus sata --type cdrom --mode readonly
Disk attached successfully

```
按下載入驅動然後瀏覽  
![load drive](http://120.114.142.50/img/17.%e8%bc%89%e5%85%a5%e9%a9%85%e5%8b%95.png)
選擇virt-io  
![virt-io1](http://120.114.142.50/img/18.%e9%81%b8%e6%93%87virtio.png)
then chosse amd64 >> w10 >> ok >> next  
![virt-io2](http://120.114.142.50/img/20.%e9%81%b8%e6%93%87virtio3.png)
![install-virtio](http://120.114.142.50/img/21.%e9%81%b8%e6%93%87virtio3.png)  
安裝完後會發現出現了一顆硬碟，但是這時候我們的ISO檔是virt-io的ISO，所以要換回來Windows10的ISO檔才能繼續安裝  
change CD/DVD again  
```bash
[root@KVM ~]# virsh attach-disk windows10 /vm_data/iso/SW_DVD9_Win_Pro_10_1903_64BIT_ChnTrad_Pro_Ent_EDU_N_MLF_X22-02880.ISO --targetbus sata --type cdrom --mode readonly
Disk attached successfully
```  
![changeISO](http://120.114.142.50/img/22.%e6%9b%b4%e6%8f%9bISO.png)  
按下重新整理，重新讀取  
![changeISO2](http://120.114.142.50/img/23.%e6%9b%b4%e6%8f%9bISO2.png)  
按下新增然後使用所有的容量來安裝作業系統
![assign space](http://120.114.142.50/img/24.%e5%88%86%e9%85%8d%e7%a1%ac%e7%a2%9f%e5%ae%b9%e9%87%8f.png)  
![assign space2](http://120.114.142.50/img/25.%e5%88%86%e9%85%8d%e7%a1%ac%e7%a2%9f%e5%ae%b9%e9%87%8f2.png)
選擇類型為主要的那個partition安裝，然後安裝  
![partition](http://120.114.142.50/img/26.%e9%81%b8%e6%93%87%e4%b8%bb%e8%a6%81partition%e5%ae%89%e8%a3%9d.png)
安裝完後會重新啟動  
![reboot](http://120.114.142.50/img/27.%e5%ae%89%e8%a3%9d%e4%b8%ad.png)  


4. 設定語系等等..  
![language](http://120.114.142.50/img/29.%e9%81%b8%e6%93%87%e8%aa%9e%e7%b3%bb%e7%ad%89....png)  
![keyboard](http://120.114.142.50/img/30.%e9%81%b8%e6%93%87%e8%aa%9e%e7%b3%bb%e7%ad%89....png)  
![net](http://120.114.142.50/img/31.%e9%81%b8%e6%93%87%e8%aa%9e%e7%b3%bb%e7%ad%89....png)  
![user](http://120.114.142.50/img/32.%e9%81%b8%e6%93%87%e8%aa%9e%e7%b3%bb%e7%ad%89....png)  
![record](http://120.114.142.50/img/33.%e9%81%b8%e6%93%87%e8%aa%9e%e7%b3%bb%e7%ad%89....png)  
等待設定  
![setting](http://120.114.142.50/img/34.%e9%81%b8%e6%93%87%e8%aa%9e%e7%b3%bb%e7%ad%89....png)  

5. 安裝完成囉~~
![complete](http://120.114.142.50/img/35.%e5%ae%89%e8%a3%9d%e5%ae%8c%e6%88%90.png)
