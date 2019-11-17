# PCI-passthrough
這個研究的目的是希望能夠做出省錢又省空間的個人主機，主要是透過虛擬化來建置這個環境。  
一般的虛擬機在使用上都會有明顯的延遲，不太能當作日常使用的個人電腦。  
然而透過CPU的虛擬化技術以及PCI-passthrough就可以將CPU和PCI裝置直接給虛擬機使用。  

大致上作業流程如下：
1. 安裝作業系統
2. 將核心的VFIO功能開啟(需要重新編譯核心)
3. 將CPU & PCI裝置分配給虛擬機
4. 在虛擬機上安裝作業系統

PCI-passthrough在INTEL或AMD分別稱為VT-D跟IOMMU，所以針對不同CPU需要做不同的檢查。
在設定上有兩個地方要特別注意:
CPU的虛擬化有沒有支援
```diff
# lscpu |grep -e 'svm' -e 'vmx'
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht
tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni
pclmulqdq dtes64 monitor ds_cpl [[[vmx]]] smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx
f16c rdrand lahf_lm abm cpuid_fault epb invpcid_single pti tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 hle
avx2 smep bmi2 erms invpcid rtm xsaveopt dtherm ida arat pln pts
```

PCI的虛擬化有沒有支援
```diff
# dmesg | grep -E "DMAR|IOMMU"
[    0.000000] Warning: PCIe ACS overrides enabled; This may allow non-IOMMU protected peer-to-peer DMA
[    0.015053] ACPI: DMAR 0x00000000AC9D3420 0000B8 (v01 INTEL  BDW      00000001 INTL 00000001)
[    0.081810] DMAR: IOMMU enabled
[    0.108660] DMAR: Host address width 39
[    0.108661] DMAR: DRHD base: 0x000000fed90000 flags: 0x0
[    0.108664] DMAR: dmar0: reg_base_addr fed90000 ver 1:0 cap c0000020660462 ecap f0101a
[    0.108664] DMAR: DRHD base: 0x000000fed91000 flags: 0x1
[    0.108666] DMAR: dmar1: reg_base_addr fed91000 ver 1:0 cap d2008c20660462 ecap f010da
[    0.108666] DMAR: RMRR base: 0x000000ade7a000 end: 0x000000ade88fff
[    0.108667] DMAR: RMRR base: 0x000000af000000 end: 0x000000bf1fffff
[    0.108668] DMAR-IR: IOAPIC id 8 under DRHD base  0xfed91000 IOMMU 1
[    0.108668] DMAR-IR: HPET id 0 under DRHD base 0xfed91000
[    0.108669] DMAR-IR: x2apic is disabled because BIOS sets x2apic opt out bit.
[    0.108669] DMAR-IR: Use 'intremap=no_x2apic_optout' to override the BIOS setting.
[    0.108976] DMAR-IR: Enabled IRQ remapping in xapic mode
[    0.737506] DMAR: No ATSR found
[    0.737532] DMAR: dmar0: Using Queued invalidation
[    0.737536] DMAR: dmar1: Using Queued invalidation
+ [    0.861114] DMAR: Intel(R) Virtualization Technology for Directed I/O
[    0.877439] ehci-pci 0000:00:1a.0: DMAR: 32bit DMA uses non-identity mapping
[    0.889036] ehci-pci 0000:00:1d.0: DMAR: 32bit DMA uses non-identity mapping
[    8.496837] [drm] DMAR active, disabling use of stolen memory
```

兩者都有的話就可以接著往下做了。
