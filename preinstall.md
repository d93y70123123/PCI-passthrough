# 安裝系統前環境檢視  
* CPU：i7-4790(4C8T)  
* MEM：32G  
* GPU：R7-200*2  
* 硬碟  
  * SSD(固態硬碟)：250G(當系統碟用)  
  * HDD(傳統硬碟)：1T*2(做Raid1，儲存容量用)  
  
# Centos8系統安裝  
使用圖形化介面安裝  
KDUMP關閉  
### 系統目錄分配
系統安裝在HDD上
/ 50G  
/vm_data 剩餘容量  

  
# 預計分配資源  
把實體資源對半切分，分別把一半的資源給虛擬機。  
`*記憶體留8G給實體機使用`  
每一台虛擬機擁有的資源:  
```
* CPU：i7-4790(4C)  
* MEM：12G  
* GPU：R7-200*1  
* 硬碟  
  * C槽：100G(系統碟)  
  * D槽：500G(儲存碟)  
```

# 系統安裝流程
![開始安裝系統](https://github.com/d93y70123123/PCI-passthrough/blob/master/1.%E9%81%B8%E5%8F%96%E5%AE%89%E8%A3%9D.PNG)  
![選語系](https://github.com/d93y70123123/PCI-passthrough/blob/master/2.%E9%81%B8%E8%AA%9E%E7%B3%BB.PNG)  
![選擇GUI](https://github.com/d93y70123123/PCI-passthrough/blob/master/3.%E9%81%B8%E6%93%87GUI%E7%92%B0%E5%A2%83.PNG)  
![關閉KDUMP](https://github.com/d93y70123123/PCI-passthrough/blob/master/4.kdump%E9%97%9C%E9%96%89.PNG)  
![選擇安裝目的地](https://github.com/d93y70123123/PCI-passthrough/blob/master/5.%E9%81%B8%E6%93%87%E5%AE%89%E8%A3%9D%E7%9B%AE%E7%9A%84%E5%9C%B0.PNG)  
![選擇自訂](https://github.com/d93y70123123/PCI-passthrough/blob/master/6.%E9%81%B8%E6%93%87%E8%87%AA%E8%A8%82.PNG)  
![選擇標準分割](https://github.com/d93y70123123/PCI-passthrough/blob/master/7.%E9%81%B8%E6%93%87%E6%A8%99%E6%BA%96%E5%88%86%E5%89%B2.PNG)  
![分割容量](https://github.com/d93y70123123/PCI-passthrough/blob/master/8.%E5%88%86%E5%89%B2%E5%AE%B9%E9%87%8F.PNG)  
![可以安裝了](https://github.com/d93y70123123/PCI-passthrough/blob/master/9.%E9%96%8B%E5%A7%8B%E5%AE%89%E8%A3%9D.PNG)  
![設定帳密](https://github.com/d93y70123123/PCI-passthrough/blob/master/10.%E8%A8%AD%E5%AE%9A%E5%B8%B3%E8%99%9F%E5%AF%86%E7%A2%BC.PNG)  


# 全系統更新
```bash
dnf -y update

[root@120-114-142-50 ~]# dnf -y update
上次中介資料過期檢查：0:01:11 以前，時間點為 西元2019年11月28日 (週四) 17時17分12秒。
依賴關係解析完畢。
===================================================================================================================================================
 軟體包                                         架構            版本                                                      軟體庫              大小
===================================================================================================================================================
Installing:
 kernel                                         x86_64          4.18.0-80.11.2.el8_0                                      BaseOS             424 k
 kernel-core                                    x86_64          4.18.0-80.11.2.el8_0                                      BaseOS              24 M
 kernel-modules                                 x86_64          4.18.0-80.11.2.el8_0                                      BaseOS              20 M
Upgrading:
 anaconda-core                                  x86_64          29.19.0.43-1.el8_0                                        AppStream          2.1 M
...
..
已安裝:
  kernel-4.18.0-80.11.2.el8_0.x86_64            kernel-core-4.18.0-80.11.2.el8_0.x86_64         kernel-modules-4.18.0-80.11.2.el8_0.x86_64
  xorg-x11-drv-fbdev-0.5.0-2.el8.x86_64         xorg-x11-drv-vesa-2.4.0-3.el8.x86_64            grub2-tools-efi-1:2.02-66.el8_0.1.x86_64

完成！

```

