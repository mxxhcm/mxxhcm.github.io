---
title: linux cpu信息
date: 2019-05-07 16:30:27
tags:
 - linux
categories: linux
---

## 查看cpu核数和cpu信息
~$:lscpu
> Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                16
On-line CPU(s) list:   0-15
Thread(s) per core:    2
Core(s) per socket:    8
Socket(s):             1
NUMA node(s):          1
Vendor ID:             AuthenticAMD
CPU family:            23
Model:                 8
Model name:            AMD Ryzen 7 2700X Eight-Core Processor
Stepping:              2
CPU MHz:               3921.420
CPU max MHz:           3700.0000
CPU min MHz:           2200.0000
BogoMIPS:              7385.61
Virtualization:        AMD-V
L1d cache:             32K
L1i cache:             64K
L2 cache:              512K
L3 cache:              8192K
NUMA node0 CPU(s):     0-15
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf pni pclmulqdq monitor ssse3 fma cx16 sse4_1 sse4_2 movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb hw_pstate sme ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt sha_ni xsaveopt xsavec xgetbv1 xsaves clzero irperf xsaveerptr arat npt lbrv svm_lock nrip_save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif overflow_recov succor smca


总核数 = 物理CPU个数 X 每颗物理CPU的核数
总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数
拿我做测试的机器来说，一个cpu，每个cpu八核，每个核两个超线程。

## 查看物理CPU个数
~$:cat /proc/cpuinfo| grep "physical id"| sort| uniq |wc -l
> 1

## 查看每个物理CPU中core的个数(即核数)
~$:cat /proc/cpuinfo| grep "cpu cores"
> 8

## 查看逻辑CPU的个数
~$:cat /proc/cpuinfo| grep "processor"| wc -l
> processor	: 0
processor	: 1
processor	: 2
processor	: 3
processor	: 4
processor	: 5
processor	: 6
processor	: 7
processor	: 8
processor	: 9
processor	: 10
processor	: 11
processor	: 12
processor	: 13
processor	: 14
processor	: 15


## 参考文献
1.鸟哥的Linux私房菜
