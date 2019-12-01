# 建置虛擬機

## create network interface

## select VM maneger  

## create virtual disk  
```bash
[root@KVM ~]# qemu-img create -f qcow2 /original/windows10.img 100G
Formatting '/original/windows10.img', fmt=qcow2 size=107374182400 cluster_size=65536 lazy_refcounts=off refcount_bits=16

[root@KVM ~]# qemu-img create -f qcow2 /vm_data/img/windows10.img 400G
Formatting '/vm_data/img/windows10.img', fmt=qcow2 size=429496729600 cluster_size=65536 lazy_refcounts=off refcount_bits=16
```

## select create new VM  
have 5 step  

## setting of VM  
* remeber press apply  

1. intro  
2. CPU check model type 'host-passthrough'  
3. check mem  
4. select boot list  
5. check disk & CD/DVD HDD select virt-io CD/DVD select SATA
6. select network interface
7. then you can clean not need device like this
8. add pci device graphic and USB device  
9. you can create VM now~
10. install your Windows OS

