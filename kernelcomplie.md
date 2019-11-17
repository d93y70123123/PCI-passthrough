# 核心編譯
底下使用ELRepo網站提供的SRPM檔案進行核心編譯(以kernel-5.3.11核心版本為例)，整體流程如下:  

1. 先到ELRepo網站下載SRPM 連結網址  
2. 透過 rpm 指令安裝下載的SRPM檔案  
3. 下載linux核心 連結網址  
4. 解壓縮linux核心，並修改核心 連結網址  
5. 最後使用SRPM重新包裝核心rpm檔案  


！！！！！(⊙ˍ⊙) ！！注意！！ (⊙ˍ⊙)！！！！！  

下載/編譯 的核心檔案（linux-4.4.69.tar.xz）一定要放在SRPM解壓縮後再ROOT家目錄底下rpmbuild/SOURCES資料夾才行!  
修改完核心參數後，打包成原來的壓縮檔前，最好刪除原本的壓縮檔比較好!  
如果是CentOS 7.2的版本以上，需要額外安裝打包指令 yum install rpm-build  


## 安裝步驟:  
下載SRPM → wget https://elrepo.org/linux/kernel/el8/SRPMS/kernel-ml-5.3.11-1.el8.elrepo.nosrc.rpm  

安裝SRPM → rpm -ivh kernel-lt-4.4.69-1.el7.elrepo.nosrc.rpm
```bash
[root@localhost ~]# ll
總計 88
-rw-------. 1 root root  1366  8月  8  2015 anaconda-ks.cfg
-rw-rw-r--. 1 dic  dic  83143  5月 25 02:18 kernel-lt-4.4.69-1.el7.elrepo.nosrc.rpm
[root@localhost ~]# rpm -ivh kernel-lt-4.4.69-1.el7.elrepo.nosrc.rpm
警告：kernel-lt-4.4.69-1.el7.elrepo.nosrc.rpm: 表頭 V4 DSA/SHA1 Signature, key ID baadae52: NOKEY
Updating / installing...
   1:kernel-lt-4.4.69-1.el7.elrepo    ################################# [100%]
警告：使用者 ajb 不存在 - 現使用 root 代替
警告：群組 ajb 不存在 - 現使用 root 代替
警告：使用者 ajb 不存在 - 現使用 root 代替
警告：群組 ajb 不存在 - 現使用 root 代替
警告：使用者 ajb 不存在 - 現使用 root 代替
警告：群組 ajb 不存在 - 現使用 root 代替
警告：使用者 ajb 不存在 - 現使用 root 代替
警告：群組 ajb 不存在 - 現使用 root 代替
[root@localhost ~]# ll
總計 88
-rw-------. 1 root root  1366  8月  8  2015 anaconda-ks.cfg
-rw-rw-r--. 1 dic  dic  83143  5月 25 02:18 kernel-lt-4.4.69-1.el7.elrepo.nosrc.rpm
drwxr-xr-x. 4 root root    32  5月 25 02:27 rpmbuild
[root@localhost ~]# 
```


下載核心 → wget http://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.3.11.tar.xz

移動核心檔案至指定目錄 → mv linux-4.4.69.tar.xz rpmbuild/SOURCES

解壓縮核心檔案 → tar Jxvf linux-4.4.69.tar.xz
```bash
[root@localhost ~]# mv linux-4.4.69.tar.xz rpmbuild/SOURCES/
[root@localhost ~]# cd rpmbuild/SOURCES/
[root@localhost SOURCES]# ll
總計 85516
-rw-rw-r--. 1 root root   168102  5月 22 05:44 config-4.4.69-x86_64
-rw-rw-r--. 1 root root      150  5月 22 05:44 cpupower.config
-rw-rw-r--. 1 root root      294  5月 22 05:44 cpupower.service
-rw-rw-r--. 1 dic  dic  87384340  5月 25 02:18 linux-4.4.69.tar.xz
[root@localhost SOURCES]# tar Jxf linux-4.4.69.tar.xz 
[root@localhost SOURCES]# ll
總計 85520
-rw-rw-r--.  1 root root   168102  5月 22 05:44 config-4.4.69-x86_64
-rw-rw-r--.  1 root root      150  5月 22 05:44 cpupower.config
-rw-rw-r--.  1 root root      294  5月 22 05:44 cpupower.service
drwxrwxr-x. 24 root root     4096  5月 20 20:27 linux-4.4.69
-rw-rw-r--.  1 dic  dic  87384340  5月 25 02:18 linux-4.4.69.tar.xz
[root@localhost SOURCES]# 
```


(σ･ω･)σ 修改核心檔案quirks.c,大約在3910行與3984行兩部份新增橘色字體程式碼 :
```bash
[root@localhost linux-4.4.69]# cd drivers/pci/
[root@localhost pci]# vim quirks.c 
```

quirks.c
```bash
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


static bool acs_on_downstream;
static bool acs_on_multifunction;

#define NUM_ACS_IDS 16
struct acs_on_id {
	unsigned short vendor;
	unsigned short device;
};
static struct acs_on_id acs_on_ids[NUM_ACS_IDS];
static u8 max_acs_id;

static __init int pcie_acs_override_setup(char *p)
{
	if (!p)
		return -EINVAL;

	while (*p) {
		if (!strncmp(p, "downstream", 10))
			acs_on_downstream = true;
		if (!strncmp(p, "multifunction", 13))
			acs_on_multifunction = true;
		if (!strncmp(p, "id:", 3)) {
			char opt[5];
			int ret;
			long val;

			if (max_acs_id >= NUM_ACS_IDS - 1) {
				pr_warn("Out of PCIe ACS override slots (%d)\n",
					NUM_ACS_IDS);
				goto next;
			}

			p += 3;
			snprintf(opt, 5, "%s", p);
			ret = kstrtol(opt, 16, &val);
			if (ret) {
				pr_warn("PCIe ACS ID parse error %d\n", ret);
				goto next;
			}
			acs_on_ids[max_acs_id].vendor = val;

			p += strcspn(p, ":");
			if (*p != ':') {
				pr_warn("PCIe ACS invalid ID\n");
				goto next;
			}

			p++;
			snprintf(opt, 5, "%s", p);
			ret = kstrtol(opt, 16, &val);
			if (ret) {
				pr_warn("PCIe ACS ID parse error %d\n", ret);
				goto next;
			}
			acs_on_ids[max_acs_id].device = val;
			max_acs_id++;
		}
next:
		p += strcspn(p, ",");
		if (*p == ',')
			p++;
	}

	if (acs_on_downstream || acs_on_multifunction || max_acs_id)
		pr_warn("Warning: PCIe ACS overrides enabled; This may allow non-IOMMU protected peer-to-peer DMA\n");

	return 0;
}
early_param("pcie_acs_override", pcie_acs_override_setup);

static int pcie_acs_overrides(struct pci_dev *dev, u16 acs_flags)
{
	int i;

	/* Never override ACS for legacy devices or devices with ACS caps */
	if (!pci_is_pcie(dev) ||
	    pci_find_ext_capability(dev, PCI_EXT_CAP_ID_ACS))
		return -ENOTTY;

	for (i = 0; i < max_acs_id; i++)
		if (acs_on_ids[i].vendor == dev->vendor &&
		    acs_on_ids[i].device == dev->device)
			return 1;

	switch (pci_pcie_type(dev)) {
	case PCI_EXP_TYPE_DOWNSTREAM:
	case PCI_EXP_TYPE_ROOT_PORT:
		if (acs_on_downstream)
			return 1;
		break;
	case PCI_EXP_TYPE_ENDPOINT:
	case PCI_EXP_TYPE_UPSTREAM:
	case PCI_EXP_TYPE_LEG_END:
	case PCI_EXP_TYPE_RC_END:
		if (acs_on_multifunction && dev->multifunction)
			return 1;
	}

	return -ENOTTY;
}


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
	{ PCI_ANY_ID, PCI_ANY_ID, pcie_acs_overrides },
	{ 0 }
};
```

修改完核心參數後,記得必須再打包成原來的壓縮檔！ ლ(・ω・ლ)  

(σ･ω･)σ 先將原本的核心檔的壓所檔刪除  
```
[root@localhost ~]# cd /root/rpmbuild/SOURCES/
[root@localhost SOURCES]# ll
總計 85520
-rw-rw-r--.  1 root root   168102  5月 22 05:44 config-4.4.69-x86_64
-rw-rw-r--.  1 root root      150  5月 22 05:44 cpupower.config
-rw-rw-r--.  1 root root      294  5月 22 05:44 cpupower.service
drwxrwxr-x. 24 root root     4096  5月 20 20:27 linux-4.4.69
-rw-rw-r--.  1 dic  dic  87384340  5月 25 02:18 linux-4.4.69.tar.xz
[root@localhost SOURCES]# rm linux-4.4.69.tar.xz 
rm：是否移除普通檔案‘linux-4.4.69.tar.xz’? y
[root@localhost SOURCES]# ll
總計 180
-rw-rw-r--.  1 root root 168102  5月 22 05:44 config-4.4.69-x86_64
-rw-rw-r--.  1 root root    150  5月 22 05:44 cpupower.config
-rw-rw-r--.  1 root root    294  5月 22 05:44 cpupower.service
drwxrwxr-x. 24 root root   4096  5月 20 20:27 linux-4.4.69
[root@localhost SOURCES]# 
```


(σ･ω･)σ 把編譯完成的核心資料夾進行壓縮動作
```
[root@localhost SOURCES]# tar Jcvf linux-4.4.69.tar.xz linux-4.4.69
linux-4.4.69/
linux-4.4.69/.get_maintainer.ignore
linux-4.4.69/.gitignore
linux-4.4.69/.mailmap
linux-4.4.69/COPYING
linux-4.4.69/CREDITS
linux-4.4.69/Documentation/
linux-4.4.69/Documentation/00-INDEX
linux-4.4.69/Documentation/ABI/
linux-4.4.69/Documentation/ABI/README
DoReMiSo
ux-4.4.69/virt/kvm/irqchip.c
linux-4.4.69/virt/kvm/kvm_main.c
linux-4.4.69/virt/kvm/vfio.c
linux-4.4.69/virt/kvm/vfio.h
linux-4.4.69/virt/lib/
linux-4.4.69/virt/lib/Kconfig
linux-4.4.69/virt/lib/Makefile
linux-4.4.69/virt/lib/irqbypass.c
[root@localhost SOURCES]# ll
總計 88132
-rw-rw-r--.  1 root root   168102  5月 22 05:44 config-4.4.69-x86_64
-rw-rw-r--.  1 root root      150  5月 22 05:44 cpupower.config
-rw-rw-r--.  1 root root      294  5月 22 05:44 cpupower.service
drwxrwxr-x. 24 root root     4096  5月 20 20:27 linux-4.4.69
-rw-r--r--.  1 root root 90060908  6月  1 00:45 linux-4.4.69.tar.xz
[root@localhost SOURCES]# 
```


啟用VFIO_PCI功能設定檔案 → config-4.4.69-x86_64

(σ･ω･)σ 在打包成rpm檔案之前，開啟VGA支援VFIO功能，大約在5782行下面新增橘色字體程式碼:
```bash
[root@localhost SOURCES]# ll
總計 88128
-rw-rw-r--.  1 root root   168102  5月 22 05:44 config-4.4.69-x86_64
-rw-rw-r--.  1 root root      150  5月 22 05:44 cpupower.config
-rw-rw-r--.  1 root root      294  5月 22 05:44 cpupower.service
drwxrwxr-x. 24 root root     4096  5月 20 20:27 linux-4.4.69
-rw-r--r--.  1 root root 90060908  6月  1 00:45 linux-4.4.69.tar.xz
[root@localhost SOURCES]# vim config-4.4.69-x86_64
```


config-4.4.69-x86_64
```bash
CONFIG_UIO_MF624=m
CONFIG_VFIO_IOMMU_TYPE1=m
CONFIG_VFIO_VIRQFD=m
CONFIG_VFIO=m
CONFIG_VFIO_PCI=m
# CONFIG_VFIO_PCI_VGA is not set
CONFIG_VFIO_PCI_VGA=y
CONFIG_VFIO_PCI_MMAP=y
CONFIG_VFIO_PCI_INTX=y
CONFIG_IRQ_BYPASS_MANAGER=m
```


設定要打包的核心檔案壓縮檔 → kernel-lt-4.4.spec
```bash
[root@localhost SOURCES]# cd ../SPECS/
[root@localhost SPECS]# ll
總計 56
-rw-rw-r--. 1 root root 56680  5月 22 05:44 kernel-lt-4.4.spec
[root@localhost SPECS]# vim kernel-lt-4.4.spec
```


更改核心檔案路徑:
```bash
=====  原本程式碼 o(^-^)o  =====
# Sources.
Source0: https://www.kernel.org/pub/linux/kernel/v4.x/linux-%{LKAver}.tar.xz
#######################################
#Source1: config-%{version}-i686
#Source2: config-%{version}-i686-NONPAE
#######################################


=====  更改後程式碼 (o^∀^)  =====
# Sources.
Source0: linux-%{LKAver}.tar.xz
#######################################
#Source1: config-%{version}-i686
#Source2: config-%{version}-i686-NONPAE
#######################################
```


編譯打包成RPM → rpmbuild -bb 最後一個步驟,編譯剛剛更改的訂定值打包成一個RPM
```bash
[root@localhost SPECS]# ll
總計 56
-rw-rw-r--. 1 root root 56635  6月  1 02:07 kernel-lt-4.4.spec
[root@localhost SPECS]# rpmbuild -bb kernel-lt-4.4.spec
錯誤：相依性建置失敗：
	asciidoc 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	gcc >= 3.4.2 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	m4 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	openssl-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	xmlto 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	audit-libs-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	binutils-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	bison 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	elfutils-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	java-1.8.0-openjdk-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	libunwind-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	newt-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	numactl-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	perl(ExtUtils::Embed) 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	python-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	slang-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	xz-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	zlib-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	ncurses-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
	pciutils-devel 被 kernel-lt-4.4.69-1.el7.centos.x86_64 需要
  ```


進行編譯如果發生相依性建置失敗時,依據前面軟體名稱一一透過YUM指令安裝
```bash
root@localhost SPECS]# yum install -y asciidoc gcc m4 openssl-devel xmlto audit-libs-devel binutils-devel bison elfutils-devel java-1.8.0-openjdk-devel libunwind-devel newt-devel numactl-devel \
python-devel perl-ExtUtils-Embed slang-devel xz-devel zlib-devel ncurses-devel pciutils-devel
```


安裝rpmbuild
```bash
[root@localhost SPECS]# yum install rpm-build -y
```


(σ･ω･)σ 開始打包動作
```bash
[root@localhost SPECS]# rpmbuild -bb kernel-lt-4.4.spec
=====  進行打包時需要一些時間才會完成 o(^-^)o  =====
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd /root/rpmbuild/BUILD
+ rm -rf kernel-lt-4.4.69
+ /usr/bin/mkdir -p kernel-lt-4.4.69
+ cd kernel-lt-4.4.69
+ /usr/bin/xz -dc /root/rpmbuild/SOURCES/linux-4.4.69.tar.xz
+ /usr/bin/tar -xf -
DoReMiSo
+ cp -pr linux-4.4.69-1.el7.centos.x86_64/tools/perf/Documentation/examples.txt /root/rpmbuild/BUILDROOT/kernel-lt-4.4.69-1.el7.centos.x86_64/usr/share/doc/perf-4.4.69
+ exit 0
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd kernel-lt-4.4.69
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/kernel-lt-4.4.69-1.el7.centos.x86_64
+ exit 0

root@localhost SPECS]# cd ..
[root@localhost rpmbuild]# ll
=====  編譯打包後會多出很多資料夾，當然包括我們需要的RPM檔案~ o(^-^)o  =====
總計 0
drwxr-xr-x. 3 root root  29  6月 12 18:57 BUILD
drwxr-xr-x. 2 root root   6  6月 12 19:17 BUILDROOT
drwxr-xr-x. 3 root root  19  6月 12 19:16 RPMS
drwxr-xr-x. 2 root root 104  6月  8 23:11 SOURCES
drwxr-xr-x. 2 root root  31  6月 12 19:17 SPECS
drwxr-xr-x. 2 root root   6  6月  8 23:31 SRPMS
[root@localhost rpmbuild]# cd RPMS
[root@localhost RPMS]# ll
總計 4
drwxr-xr-x. 2 root root 4096  6月 12 19:17 x86_64
[root@localhost RPMS]# cd x86_64
[root@localhost x86_64]# ll
=====  黃色的rpm就是剛編譯好，具有VFIO功能核心檔案 o(^-^)o  =====
總計 52148
-rw-r--r--. 1 root root 39790928  6月 12 19:16 kernel-lt-4.4.69-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 10599620  6月 12 19:17 kernel-lt-devel-4.4.69-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root  1014944  6月 12 19:17 kernel-lt-headers-4.4.69-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root   124536  6月 12 19:17 kernel-lt-tools-4.4.69-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root    43680  6月 12 19:17 kernel-lt-tools-libs-4.4.69-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root    30600  6月 12 19:17 kernel-lt-tools-libs-devel-4.4.69-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root  1340652  6月 12 19:17 perf-4.4.69-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root   438764  6月 12 19:17 python-perf-4.4.69-1.el7.centos.x86_64.rpm
```



(σ･ω･)σ 開始安裝新核心，記得安裝完需要重新開機，且選擇新的核心開機才對~
```
[root@localhost x86_64]# yum install kernel-lt-4.4.69-1.el7.centos.x86_64.rpm 

================================================================================================================
 Package          Arch          Version                      Repository                                    Size
================================================================================================================
Installing:
 kernel-lt        x86_64        4.4.69-1.el7.centos          /kernel-lt-4.4.69-1.el7.centos.x86_64        171 M

Transaction Summary
================================================================================================================
Install  1 Package

Total size: 171 M
Installed size: 171 M
Is this ok [y/d/N]: 
```


(σ･ω･)σ 重新開機時在開機選單裡面出現4.4.69版本的核心，選擇第一個進入

開機選單
