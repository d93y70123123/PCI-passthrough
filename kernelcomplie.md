# 核心編譯
底下使用ELRepo網站提供的SRPM檔案進行核心編譯(以kernel-5.4.0核心版本為例)，整體流程如下:  

1. 先到ELRepo網站下載SRPM [連結網址](https://elrepo.org/linux/kernel/el8/SRPMS/)  
2. 透過 rpm 指令安裝下載的SRPM檔案  
3. 下載linux核心 [連結網址](http://cdn.kernel.org/pub/linux/kernel/v5.x/)  
4. 解壓縮linux核心，並修改核心 [連結網址](https://lkml.org/lkml/2013/5/30/513)  
5. 最後使用SRPM重新包裝核心rpm檔案  


！！！！！(⊙ˍ⊙) ！！注意！！ (⊙ˍ⊙)！！！！！  

下載/編譯 的核心檔案（linux-5.4.0.tar.xz）一定要放在SRPM解壓縮後再ROOT家目錄底下rpmbuild/SOURCES資料夾才行!  
修改完核心參數後，打包成原來的壓縮檔前，最好刪除原本的壓縮檔比較好!  
如果是CentOS 7.2的版本以上，需要額外安裝打包指令 dnf install rpm-build  


## 安裝步驟:  
下載SRPM → wget https://elrepo.org/linux/kernel/el8/SRPMS/kernel-ml-5.4.0-1.el8.elrepo.nosrc.rpm  

安裝SRPM → rpm -ivh kernel-ml-5.4.0-1.el8.elrepo.nosrc.rpm
```bash
[root@KVM ~]# ll
總計 106964
-rw-------. 1 root root      1400 11月 28 17:04 anaconda-ks.cfg
-rw-r--r--. 1 root root      1555 11月 28 17:07 initial-setup-ks.cfg
-rw-r--r--. 1 root root     77616 11月 25 10:15 kernel-ml-5.4.0-1.el8.elrepo.nosrc.rpm
[root@KVM ~]# rpm -ivh kernel-ml-5.4.0-1.el8.elrepo.nosrc.rpm
警告：kernel-ml-5.4.0-1.el8.elrepo.nosrc.rpm: 表頭 V4 DSA/SHA1 Signature, key ID baadae52: NOKEY
Updating / installing...
   1:kernel-ml-5.4.0-1.el8.elrepo     警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
################################# [100%]
警告：user ajb does not exist - using root
警告：group ajb does not exist - using root
[root@KVM ~]# ll
總計 84
-rw-------. 1 root root  1400 11月 28 17:04 anaconda-ks.cfg
-rw-r--r--. 1 root root  1555 11月 28 17:07 initial-setup-ks.cfg
-rw-r--r--. 1 root root 77616 11月 25 10:15 kernel-ml-5.4.0-1.el8.elrepo.nosrc.rpm
drwxr-xr-x. 4 root root    34 11月 28 17:32 rpmbuild
[root@KVM ~]# 
```


下載核心 → wget http://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.4.tar.xz  

移動核心檔案至指定目錄 → cp linux-5.4.tar.xz rpmbuild/SOURCES/  

解壓縮核心檔案 → tar -Jxf linux-5.4.tar.xz  
```bash
[root@KVM ~]# ll
總計 106964
-rw-------. 1 root root      1400 11月 28 17:04 anaconda-ks.cfg
-rw-r--r--. 1 root root      1555 11月 28 17:07 initial-setup-ks.cfg
-rw-r--r--. 1 root root     77616 11月 25 10:15 kernel-ml-5.4.0-1.el8.elrepo.nosrc.rpm
-rw-r--r--. 1 root root 109441440 11月 25 05:32 linux-5.4.tar.xz
drwxr-xr-x. 4 root root        34 11月 28 17:32 rpmbuild
[root@KVM ~]# cp linux-5.4.tar.xz rpmbuild/SOURCES/
[root@KVM ~]# cd rpmbuild/SOURCES/
[root@KVM SOURCES]# ll
總計 107096
-rw-rw-r--. 1 root root    187945 11月 25 09:20 config-5.4.0-x86_64
-rw-rw-r--. 1 root root       150 11月 24 19:38 cpupower.config
-rw-rw-r--. 1 root root       294 11月 24 19:38 cpupower.service
-rwxrwxr-x. 1 root root      3962 11月 24 19:38 filter-modules.sh
-rwxrwxr-x. 1 root root       561 11月 24 19:38 filter-x86_64.sh
-rwxrwxr-x. 1 root root       652 11月 24 19:38 generate_bls_conf.sh
-rw-r--r--. 1 root root 109441440 11月 28 17:36 linux-5.4.tar.xz
-rwxrwxr-x. 1 root root      1386 11月 24 19:38 mod-extra-blacklist.sh
-rw-rw-r--. 1 root root      2251 11月 24 19:38 mod-extra.list
-rwxrwxr-x. 1 root root      1484 11月 24 19:38 mod-extra.sh
[root@KVM SOURCES]# tar -Jxf linux-5.4.tar.xz
[root@KVM SOURCES]# ll
總計 107100
-rw-rw-r--.  1 root root    187945 11月 25 09:20 config-5.4.0-x86_64
-rw-rw-r--.  1 root root       150 11月 24 19:38 cpupower.config
-rw-rw-r--.  1 root root       294 11月 24 19:38 cpupower.service
-rwxrwxr-x.  1 root root      3962 11月 24 19:38 filter-modules.sh
-rwxrwxr-x.  1 root root       561 11月 24 19:38 filter-x86_64.sh
-rwxrwxr-x.  1 root root       652 11月 24 19:38 generate_bls_conf.sh
drwxrwxr-x. 24 root root      4096 11月 24 19:32 linux-5.4
-rw-r--r--.  1 root root 109441440 11月 28 17:36 linux-5.4.tar.xz
-rwxrwxr-x.  1 root root      1386 11月 24 19:38 mod-extra-blacklist.sh
-rw-rw-r--.  1 root root      2251 11月 24 19:38 mod-extra.list
-rwxrwxr-x.  1 root root      1484 11月 24 19:38 mod-extra.sh
[root@KVM SOURCES]#
```


(σ･ω･)σ 修改核心檔案quirks.c,大約在3910行與3984行兩部份新增橘色字體程式碼 :
```bash
[root@KVM SOURCES]# vim linux-5.4/drivers/pci/quirks.c
```

### quirks.c的內容
```diff
static int pci_quirk_mf_endpoint_acs(struct pci_dev *dev, u16 acs_flags)
{
        /*
         * SV, TB, and UF are not relevant to multifunction endpoints.
         *
         * Multifunction devices are only required to implement RR, CR, and DT
         * in their ACS capability if they support peer-to-peer transactions.
         * Devices matching this quirk have been verified by the vendor to not
         * perform peer-to-peer with other functions, allowing us to mask out
         * these bits as if they were unimplemented in the ACS capability.
         */
        acs_flags &= ~(PCI_ACS_SV | PCI_ACS_TB | PCI_ACS_RR |
                       PCI_ACS_CR | PCI_ACS_UF | PCI_ACS_DT);

        return acs_flags ? 0 : 1;
}

+ static bool acs_on_downstream;
+ static bool acs_on_multifunction;

+ #define NUM_ACS_IDS 16
+ struct acs_on_id {
+	unsigned short vendor;
+	unsigned short device;
+  };
+ static struct acs_on_id acs_on_ids[NUM_ACS_IDS];
+ static u8 max_acs_id;

+ static __init int pcie_acs_override_setup(char *p)
+ {
+ 	if (!p)
+ 		return -EINVAL;
+ 
+ 	while (*p) {
+ 		if (!strncmp(p, "downstream", 10))
+ 			acs_on_downstream = true;
+ 		if (!strncmp(p, "multifunction", 13))
+ 			acs_on_multifunction = true;
+ 		if (!strncmp(p, "id:", 3)) {
+ 			char opt[5];
+ 			int ret;
+ 			long val;
+ 
+ 			if (max_acs_id >= NUM_ACS_IDS - 1) {
+ 				pr_warn("Out of PCIe ACS override slots (%d)\n",
+ 					NUM_ACS_IDS);
+ 				goto next;
+ 			}
+ 
+ 			p += 3;
+ 			snprintf(opt, 5, "%s", p);
+ 			ret = kstrtol(opt, 16, &val);
+ 			if (ret) {
+ 				pr_warn("PCIe ACS ID parse error %d\n", ret);
+ 				goto next;
+ 			}
+ 			acs_on_ids[max_acs_id].vendor = val;
+ 
+ 			p += strcspn(p, ":");
+ 			if (*p != ':') {
+ 				pr_warn("PCIe ACS invalid ID\n");
+ 				goto next;
+ 			}
+ 
+ 			p++;
+ 			snprintf(opt, 5, "%s", p);
+ 			ret = kstrtol(opt, 16, &val);
+ 			if (ret) {
+ 				pr_warn("PCIe ACS ID parse error %d\n", ret);
+ 				goto next;
+ 			}
+ 			acs_on_ids[max_acs_id].device = val;
+ 			max_acs_id++;
+ 		}
+ next:
+ 		p += strcspn(p, ",");
+ 		if (*p == ',')
+ 			p++;
+ 	}
+ 
+ 	if (acs_on_downstream || acs_on_multifunction || max_acs_id)
+ 		pr_warn("Warning: PCIe ACS overrides enabled; This may allow non-IOMMU protected peer-to-peer DMA\n");
+ 
+ 	return 0;
+ }
+ early_param("pcie_acs_override", pcie_acs_override_setup);
+ 
+ static int pcie_acs_overrides(struct pci_dev *dev, u16 acs_flags)
+ {
+ 	int i;
+ 
+ 	/* Never override ACS for legacy devices or devices with ACS caps */
+ 	if (!pci_is_pcie(dev) ||
+ 	    pci_find_ext_capability(dev, PCI_EXT_CAP_ID_ACS))
+ 		return -ENOTTY;
+ 
+ 	for (i = 0; i < max_acs_id; i++)
+ 		if (acs_on_ids[i].vendor == dev->vendor &&
+ 		    acs_on_ids[i].device == dev->device)
+ 			return 1;
+ 
+ 	switch (pci_pcie_type(dev)) {
+ 	case PCI_EXP_TYPE_DOWNSTREAM:
+ 	case PCI_EXP_TYPE_ROOT_PORT:
+ 		if (acs_on_downstream)
+ 			return 1;
+ 		break;
+ 	case PCI_EXP_TYPE_ENDPOINT:
+ 	case PCI_EXP_TYPE_UPSTREAM:
+ 	case PCI_EXP_TYPE_LEG_END:
+ 	case PCI_EXP_TYPE_RC_END:
+ 		if (acs_on_multifunction && dev->multifunction)
+ 			return 1;
+ 	}
+ 
+ 	return -ENOTTY;
+ }

static const struct pci_dev_acs_enabled {
        u16 vendor;
        u16 device;
        int (*acs_enabled)(struct pci_dev *dev, u16 acs_flags);
} pci_dev_acs_enabled[] = {
	{ PCI_VENDOR_ID_ATI, 0x4385, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_ATI, 0x439c, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_ATI, 0x4383, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_ATI, 0x439d, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_ATI, 0x4384, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_ATI, 0x4399, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_AMD, 0x780f, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_AMD, 0x7809, pci_quirk_amd_sb_acs },
	{ PCI_VENDOR_ID_SOLARFLARE, 0x0903, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_SOLARFLARE, 0x0923, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10C6, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10DB, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10DD, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10E1, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10F1, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10F7, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10F8, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10F9, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10FA, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10FB, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10FC, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1507, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1514, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x151C, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1529, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x152A, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x154D, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x154F, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1551, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1558, pci_quirk_mf_endpoint_acs },
	/* 82580 */
	{ PCI_VENDOR_ID_INTEL, 0x1509, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x150E, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x150F, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1510, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1511, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1516, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1527, pci_quirk_mf_endpoint_acs },
	/* 82576 */
	{ PCI_VENDOR_ID_INTEL, 0x10C9, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10E6, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10E7, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10E8, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x150A, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x150D, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1518, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1526, pci_quirk_mf_endpoint_acs },
	/* 82575 */
	{ PCI_VENDOR_ID_INTEL, 0x10A7, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10A9, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10D6, pci_quirk_mf_endpoint_acs },
	/* I350 */
	{ PCI_VENDOR_ID_INTEL, 0x1521, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1522, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1523, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1524, pci_quirk_mf_endpoint_acs },
	/* 82571 (Quads omitted due to non-ACS switch) */
	{ PCI_VENDOR_ID_INTEL, 0x105E, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x105F, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x1060, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x10D9, pci_quirk_mf_endpoint_acs },
	/* I219 */
	{ PCI_VENDOR_ID_INTEL, 0x15b7, pci_quirk_mf_endpoint_acs },
	{ PCI_VENDOR_ID_INTEL, 0x15b8, pci_quirk_mf_endpoint_acs },
	/* Intel PCH root ports */
	{ PCI_VENDOR_ID_INTEL, PCI_ANY_ID, pci_quirk_intel_pch_acs },
	{ 0x19a2, 0x710, pci_quirk_mf_endpoint_acs }, /* Emulex BE3-R */
	{ 0x10df, 0x720, pci_quirk_mf_endpoint_acs }, /* Emulex Skyhawk-R */
+ 	{ PCI_ANY_ID, PCI_ANY_ID, pcie_acs_overrides },
	{ 0 }
};
```

### 修改完核心參數後,記得必須再打包成原來的壓縮檔！ ლ(・ω・ლ)  

(σ･ω･)σ 先將原本的核心檔的壓所檔刪除  
```bash
[root@KVM SOURCES]# ll
總計 107100
-rw-rw-r--.  1 root root    187945 11月 25 09:20 config-5.4.0-x86_64
-rw-rw-r--.  1 root root       150 11月 24 19:38 cpupower.config
-rw-rw-r--.  1 root root       294 11月 24 19:38 cpupower.service
-rwxrwxr-x.  1 root root      3962 11月 24 19:38 filter-modules.sh
-rwxrwxr-x.  1 root root       561 11月 24 19:38 filter-x86_64.sh
-rwxrwxr-x.  1 root root       652 11月 24 19:38 generate_bls_conf.sh
drwxrwxr-x. 24 root root      4096 11月 24 19:32 linux-5.4
-rw-r--r--.  1 root root 109441440 11月 28 17:36 linux-5.4.tar.xz
-rwxrwxr-x.  1 root root      1386 11月 24 19:38 mod-extra-blacklist.sh
-rw-rw-r--.  1 root root      2251 11月 24 19:38 mod-extra.list
-rwxrwxr-x.  1 root root      1484 11月 24 19:38 mod-extra.sh
[root@KVM SOURCES]# rm linux-5.4.tar.xz
rm：是否移除普通檔案'linux-5.4.tar.xz'? y
[root@KVM SOURCES]# ll
總計 220
-rw-rw-r--.  1 root root 187945 11月 25 09:20 config-5.4.0-x86_64
-rw-rw-r--.  1 root root    150 11月 24 19:38 cpupower.config
-rw-rw-r--.  1 root root    294 11月 24 19:38 cpupower.service
-rwxrwxr-x.  1 root root   3962 11月 24 19:38 filter-modules.sh
-rwxrwxr-x.  1 root root    561 11月 24 19:38 filter-x86_64.sh
-rwxrwxr-x.  1 root root    652 11月 24 19:38 generate_bls_conf.sh
drwxrwxr-x. 24 root root   4096 11月 24 19:32 linux-5.4
-rwxrwxr-x.  1 root root   1386 11月 24 19:38 mod-extra-blacklist.sh
-rw-rw-r--.  1 root root   2251 11月 24 19:38 mod-extra.list
-rwxrwxr-x.  1 root root   1484 11月 24 19:38 mod-extra.sh
```


(σ･ω･)σ 把編譯完成的核心資料夾進行壓縮動作
```bash
[root@KVM SOURCES]# tar -Jcvf linux-5.4.tar.xz linux-5.4
linux-5.4/
linux-5.4/.clang-format
linux-5.4/.cocciconfig
linux-5.4/.get_maintainer.ignore
linux-5.4/.gitattributes
linux-5.4/.gitignore
linux-5.4/.mailmap
linux-5.4/COPYING
linux-5.4/CREDITS
linux-5.4/Documentation/
linux-5.4/Documentation/.gitignore
linux-5.4/Documentation/ABI/
linux-5.4/Documentation/ABI/README
....
DoReMiSo
....
linux-5.4/virt/kvm/kvm_main.c
linux-5.4/virt/kvm/vfio.c
linux-5.4/virt/kvm/vfio.h
linux-5.4/virt/lib/
linux-5.4/virt/lib/Kconfig
linux-5.4/virt/lib/Makefile
linux-5.4/virt/lib/irqbypass.c
[root@KVM SOURCES]# ll
總計 110052
-rw-rw-r--.  1 root root    187945 11月 25 09:20 config-5.4.0-x86_64
-rw-rw-r--.  1 root root       150 11月 24 19:38 cpupower.config
-rw-rw-r--.  1 root root       294 11月 24 19:38 cpupower.service
-rwxrwxr-x.  1 root root      3962 11月 24 19:38 filter-modules.sh
-rwxrwxr-x.  1 root root       561 11月 24 19:38 filter-x86_64.sh
-rwxrwxr-x.  1 root root       652 11月 24 19:38 generate_bls_conf.sh
drwxrwxr-x. 24 root root      4096 11月 24 19:32 linux-5.4
-rw-r--r--.  1 root root 112464048 11月 28 18:07 linux-5.4.tar.xz
-rwxrwxr-x.  1 root root      1386 11月 24 19:38 mod-extra-blacklist.sh
-rw-rw-r--.  1 root root      2251 11月 24 19:38 mod-extra.list
-rwxrwxr-x.  1 root root      1484 11月 24 19:38 mod-extra.sh
[root@KVM SOURCES]# 
```


### 啟用VFIO_PCI功能設定檔案 → config-5.4.0-x86_64  

(σ･ω･)σ 在打包成rpm檔案之前，開啟VGA支援VFIO功能，大約在5782行下面新增橘色字體程式碼:
```bash
[root@KVM SOURCES]# ll
總計 110052
-rw-rw-r--.  1 root root    187945 11月 25 09:20 config-5.4.0-x86_64
-rw-rw-r--.  1 root root       150 11月 24 19:38 cpupower.config
-rw-rw-r--.  1 root root       294 11月 24 19:38 cpupower.service
-rwxrwxr-x.  1 root root      3962 11月 24 19:38 filter-modules.sh
-rwxrwxr-x.  1 root root       561 11月 24 19:38 filter-x86_64.sh
-rwxrwxr-x.  1 root root       652 11月 24 19:38 generate_bls_conf.sh
drwxrwxr-x. 24 root root      4096 11月 24 19:32 linux-5.4
-rw-r--r--.  1 root root 112464048 11月 28 18:07 linux-5.4.tar.xz
-rwxrwxr-x.  1 root root      1386 11月 24 19:38 mod-extra-blacklist.sh
-rw-rw-r--.  1 root root      2251 11月 24 19:38 mod-extra.list
-rwxrwxr-x.  1 root root      1484 11月 24 19:38 mod-extra.sh
[root@KVM SOURCES]# vim config-5.4.0-x86_64
```


config-5.4.0-x86_64  
約6054行  
```diff
CONFIG_UIO_MF624=m
CONFIG_VFIO_IOMMU_TYPE1=m
CONFIG_VFIO_VIRQFD=m
CONFIG_VFIO=m
CONFIG_VFIO_PCI=m
# CONFIG_VFIO_PCI_VGA is not set
+ CONFIG_VFIO_PCI_VGA=y
CONFIG_VFIO_PCI_MMAP=y
CONFIG_VFIO_PCI_INTX=y
CONFIG_IRQ_BYPASS_MANAGER=m
```


設定要打包的核心檔案壓縮檔 → kernel-ml-5.4.spec
```bash
[root@KVM SOURCES]# cd ../SPECS/
[root@KVM SPECS]# ll
總計 48
-rw-rw-r--. 1 root root 46322 11月 24 19:38 kernel-ml-5.4.spec
[root@KVM SPECS]# vim kernel-ml-5.4.spec
```


更改核心檔案路徑:
```bash
=====  原本程式碼 o(^-^)o  =====
# Sources.
Source0: https://www.kernel.org/pub/linux/kernel/v5.x/linux-%{LKAver}.tar.xz
Source1: config-%{version}-x86_64
...
..

=====  更改後程式碼 (o^∀^)  =====
# Sources.
Source0: linux-%{LKAver}.tar.xz
Source1: config-%{version}-x86_64
...
..
```


### 編譯打包成RPM → rpmbuild -bb 最後一個步驟,編譯剛剛更改的訂定值打包成一個RPM  
#### 安裝rpmbuild  
```bash
[root@KVM SPECS]# dnf -y install rpm-build 
```

```bash
[root@KVM SPECS]# ll
總計 48
-rw-rw-r--. 1 root root 46277 11月 28 18:14 kernel-ml-5.4.spec
[root@KVM SPECS]# rpmbuild -bb kernel-ml-5.4.spec
錯誤：相依性建置失敗：
        asciidoc 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        audit-libs-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        binutils-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        bison 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        elfutils-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        flex 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        gcc 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        git 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        java-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        libcap-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        m4 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        make 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        ncurses-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        newt-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        numactl-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        openssl-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        pciutils-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        perl(ExtUtils::Embed) 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        perl-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        perl-generators 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        python3-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        python3-docutils 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        xmlto 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        xz-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
        zlib-devel 被 kernel-ml-5.4.0-1.el8.x86_64 需要
[root@KVM SPECS]#
  ```


### 進行編譯如果發生相依性建置失敗時,依據前面軟體名稱一一透過YUM指令安裝
```bash
[root@KVM SPECS]# dnf -y install asciidoc audit-libs-devel binutils-devel bison elfutils-devel flex gcc git java-devel libcap-devel m4 make ncurses-devel newt-devel numactl-devel openssl-devel pciutils-devel perl-ExtUtils-Embed perl-devel perl-generators python3-devel python3-docutils xmlto xz-devel zlib-devel
```



### (σ･ω･)σ 開始打包動作
```bash
[root@KVM SPECS]# rpmbuild -bb kernel-ml-5.4.spec
=====  進行打包時需要一些時間才會完成 o(^-^)o  =====
正在執行(%prep)：/bin/sh -e /var/tmp/rpm-tmp.IkpKqr
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd /root/rpmbuild/BUILD
+ rm -rf kernel-ml-5.4.0
+ /usr/bin/mkdir -p kernel-ml-5.4.0
+ cd kernel-ml-5.4.0
+ /usr/bin/xz -dc /root/rpmbuild/SOURCES/linux-5.4.tar.xz
+ /usr/bin/tar -xof -
...
DoReMiSo
...
已寫入：/root/rpmbuild/RPMS/x86_64/kernel-ml-modules-extra-5.4.0-1.el8.x86_64.rpm
正在執行(%clean)：/bin/sh -e /var/tmp/rpm-tmp.RujWJU
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd kernel-ml-5.4.0
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/kernel-ml-5.4.0-1.el8.x86_64
+ exit 0


[root@KVM SPECS]# cd ..
=====  編譯打包後會多出很多資料夾，當然包括我們需要的RPM檔案~ o(^-^)o  =====
[root@KVM rpmbuild]# ll
總計 0
drwxr-xr-x. 3 root root  29 11月 29 03:57 BUILD
drwxr-xr-x. 2 root root   6 11月 29 05:34 BUILDROOT
drwxr-xr-x. 3 root root  20 11月 29 05:34 RPMS
drwxr-xr-x. 3 root root 270 11月 29 03:47 SOURCES
drwxr-xr-x. 2 root root  32 11月 29 03:48 SPECS
drwxr-xr-x. 2 root root   6 11月 29 03:57 SRPMS
[root@KVM rpmbuild]# cd RPMS/
[root@KVM RPMS]# ll
總計 4
drwxr-xr-x. 2 root root 4096 11月 29 05:34 x86_64
[root@KVM RPMS]# cd x86_64/
=====  黃色的rpm就是剛編譯好，具有VFIO功能核心檔案 o(^-^)o  =====
[root@KVM x86_64]# ll
總計 72308
-rw-r--r--. 1 root root  1053164 11月 29 05:34 bpftool-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root    13944 11月 29 05:34 kernel-ml-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root 27945620 11月 29 05:34 kernel-ml-core-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root 13232872 11月 29 05:34 kernel-ml-devel-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root  1312860 11月 29 05:34 kernel-ml-headers-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root 22082992 11月 29 05:34 kernel-ml-modules-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root   811648 11月 29 05:34 kernel-ml-modules-extra-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root   199552 11月 29 05:34 kernel-ml-tools-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root    45412 11月 29 05:34 kernel-ml-tools-libs-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root    16716 11月 29 05:34 kernel-ml-tools-libs-devel-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root  6659440 11月 29 05:34 perf-5.4.0-1.el8.x86_64.rpm
-rw-r--r--. 1 root root   642104 11月 29 05:34 python3-perf-5.4.0-1.el8.x86_64.rpm
[root@KVM x86_64]#
```



(σ･ω･)σ 開始安裝新核心，記得安裝完需要重新開機，且選擇新的核心開機才對~  
現在要安裝kernel會比舊版的核心安裝還要多裝兩個rpm，不過別擔心，編譯完後也都一起編好了，而且系統也會提醒需要裝哪些。
```bash
[root@KVM x86_64]# dnf install kernel-ml-5.4.0-1.el8.x86_64.rpm
上次中介資料過期檢查：1:59:26 以前，時間點為 西元2019年11月29日 (週五) 06時34分58秒。
錯誤：
 問題: conflicting requests
  - nothing provides kernel-ml-core-uname-r = 5.4.0-1.el8.x86_64 needed by kernel-ml-5.4.0-1.el8.x86_64
  - nothing provides kernel-ml-modules-uname-r = 5.4.0-1.el8.x86_64 needed by kernel-ml-5.4.0-1.el8.x86_64
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
[root@KVM x86_64]# dnf -y install kernel-ml-core-5.4.0-1.el8.x86_64.rpm kernel-ml-modules-5.4.0-1.el8.x86_64.rpm
==========================================================================================
 軟體包                                  架構                         版本
=================================================================================
Installing:
 kernel-ml-core                          x86_64                       5.4.0-1.el8
 kernel-ml-modules                       x86_64                       5.4.0-1.el8

處理事項摘要
=================================================================================
安裝  2 軟體包

總大小：48 M
安裝的大小：82 M
下載軟體包：
執行處理事項檢查
處理事項檢查成功。
執行處理事項測試
處理事項測試成功。
執行處理事項
  準備        :
  Installing  : kernel-ml-core-5.4.0-1.el8.x86_64
  執行指令小稿: kernel-ml-core-5.4.0-1.el8.x86_64
  Installing  : kernel-ml-modules-5.4.0-1.el8.x86_64
  執行指令小稿: kernel-ml-modules-5.4.0-1.el8.x86_64
  執行指令小稿: kernel-ml-core-5.4.0-1.el8.x86_64
  執行指令小稿: kernel-ml-modules-5.4.0-1.el8.x86_64
  核驗        : kernel-ml-core-5.4.0-1.el8.x86_64
  核驗        : kernel-ml-modules-5.4.0-1.el8.x86_64

已安裝:
  kernel-ml-core-5.4.0-1.el8.x86_64                                      kernel-m

完成！
#####裝完上面兩個軟體就可以接著安裝核心了#####
[root@KVM x86_64]# dnf install kernel-ml-5.4.0-1.el8.x86_64.rpm                           上次中介資料過期檢查：0:01:24 以前，時間點為 西元2019年11月29日 (週五) 08時38分07秒。
依賴關係解析完畢。
==========================================================================================
 軟體包              架構             版本                   軟體庫                  大小
==========================================================================================
Installing:
 kernel-ml           x86_64           5.4.0-1.el8            @commandline            14 k

處理事項摘要
==========================================================================================
安裝  1 軟體包

總大小：14 k
安裝的大小：0
這樣可以嗎 [y/N]： y
下載軟體包：
執行處理事項檢查
處理事項檢查成功。
執行處理事項測試
處理事項測試成功。
執行處理事項
  準備        :                                                                       1/1
  Installing  : kernel-ml-5.4.0-1.el8.x86_64                                          1/1
  核驗        : kernel-ml-5.4.0-1.el8.x86_64                                          1/1

已安裝:
  kernel-ml-5.4.0-1.el8.x86_64

完成！
```


(σ･ω･)σ 重新開機時在開機選單裡面出現5.4.0版本的核心，選擇第一個進入  

開機選單  
![開機選單](https://github.com/d93y70123123/PCI-passthrough/blob/master/%E9%81%B8%E6%A0%B8%E5%BF%83%E9%96%8B%E6%A9%9F.PNG)
