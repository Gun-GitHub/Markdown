[[海光/BW101-测试|BW101-测试]]
[[天数智芯/RM-V100|RM-V100]]
# 一. 测试环境信息
以下测试环境在同一节点:
```bash
# ======================================> 节点 ip <===========================================
(base) sophgo@sophgo-SG6-06-A32:~/Documents/performance_loss_test$ ip addr |grep 192.168.100
    inet 192.168.100.98/24 brd 192.168.100.255 scope global noprefixroute eno1
# ======================================> 操作系统 <===========================================
(base) sophgo@sophgo-SG6-06-A32:~/Documents/performance_loss_test$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.6 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
# ======================================> cpu 信息 <===========================================
(base) sophgo@sophgo-SG6-06-A32:~/Documents/performance_loss_test$ lscpu
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   46 bits physical, 57 bits virtual
CPU(s):                          80
On-line CPU(s) list:             0-79
Thread(s) per core:              2
Core(s) per socket:              20
Socket(s):                       2
NUMA node(s):                    2
Vendor ID:                       GenuineIntel
CPU family:                      6
Model:                           106
Model name:                      Intel(R) Xeon(R) Silver 4316 CPU @ 2.30GHz
Stepping:                        6
CPU MHz:                         801.178
CPU max MHz:                     3400.0000
CPU min MHz:                     800.0000
BogoMIPS:                        4600.00
Virtualization:                  VT-x
L1d cache:                       1.9 MiB
L1i cache:                       1.3 MiB
L2 cache:                        50 MiB
L3 cache:                        60 MiB
NUMA node0 CPU(s):               0-19,40-59
NUMA node1 CPU(s):               20-39,60-79
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and secco
                                 mp
Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitizat
                                 ion
Vulnerability Spectre v2:        Mitigation; Enhanced IBRS, IBPB conditional, RSB filling
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat
                                  pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx
                                  pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_goo
                                 d nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes6
                                 4 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pc
                                 id dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes x
                                 save avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb ca
                                 t_l3 invpcid_single ssbd mba ibrs ibpb stibp ibrs_enhanced tpr_sh
                                 adow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 a
                                 vx2 smep bmi2 erms invpcid cqm rdt_a avx512f avx512dq rdseed adx 
                                 smap avx512ifma clflushopt clwb intel_pt avx512cd sha_ni avx512bw
                                  avx512vl xsaveopt xsavec xgetbv1 xsaves cqm_llc cqm_occup_llc cq
                                 m_mbm_total cqm_mbm_local wbnoinvd dtherm ida arat pln pts hwp hw
                                 p_act_window hwp_epp hwp_pkg_req avx512vbmi umip pku ospke avx512
                                 _vbmi2 gfni vaes vpclmulqdq avx512_vnni avx512_bitalg tme avx512_
                                 vpopcntdq rdpid md_clear pconfig flush_l1d arch_capabilities
# ======================================> 内存信息 <===========================================
(base) sophgo@sophgo-SG6-06-A32:~/Documents/performance_loss_test$ lsmem
RANGE                                  SIZE  STATE REMOVABLE BLOCK
0x0000000000000000-0x000000007fffffff    2G online       yes     0
0x0000000100000000-0x000000207fffffff  126G online       yes  2-64

Memory block size:         2G
Total online memory:     128G
Total offline memory:      0B
# ======================================> 驱动安装情况 <========================================
(base) sophgo@sophgo-SG6-06-A32:~/Documents/performance_loss_test$ ixsmi 
Timestamp    Mon May 25 20:50:24 2026
+--------------------------------------------------------------------------------+
|  IX-ML: 4.4.0            Driver Version: 4.4.0          CUDA Version: 10.2     |
|----------------------------------+----------------------+----------------------|
| GPU  Name                        | Bus-Id               | Clock-SM  Clock-Mem  |
| Fan  Temp  Perf    Pwr:Usage/Cap |      Memory-Usage    | GPU-Util  Compute M. |
|==================================+======================+======================|
| 0    Iluvatar MR-V100 DUO        | 00000000:35:00.0     | 500MHz    1600MHz    |
| N/A  37C   P0       N/A / N/A    | 68MiB / 32768MiB     | 0%        Default    |
+----------------------------------+----------------------+----------------------+
| 1    Iluvatar MR-V100 DUO        | 00000000:38:00.0     | 500MHz    1600MHz    |
| N/A  36C   P0       40W / 350W   | 68MiB / 32768MiB     | 0%        Default    |
+----------------------------------+----------------------+----------------------+

+--------------------------------------------------------------------------------+
| Processes:                                                          GPU Memory |
|  GPU        PID      Process name                                   Usage(MiB) |
|================================================================================|
|  No running processes found                                                    |
+--------------------------------------------------------------------------------+
(base) sophgo@sophgo-SG6-06-A32:~/Documents/performance_loss_test$ hy-smi

================================= System Management Interface ==================================
================================================================================================
HCU     Temp     AvgPwr     Perf     PwrCap     VRAM%      HCU%      Dec%      Enc%      Mode     
0       46.0C    51.0W      auto     400.0W     0%         0.0%      0.0%      0.0%      Normal   
================================================================================================
======================================== End of SMI Log ========================================
(base) sophgo@sophgo-SG6-06-A32:~/Documents/performance_loss_test$ ht-smi
ht-smi  version: 2.3.1
=================== Mars System Management Interface Log ===================
Timestamp                                         : Wed May 27 17:16:21 2026
Attached GPUs                                     : 1
+---------------------------------------------------------------------------------+
| HT-SMI 2.3.1                       Kernel Mode Driver Version: 3.8.1            |
| HPCC Version: 3.7.1.5              BIOS Version: 1.31.1.0                       |
|------------------+-----------------+---------------------+----------------------|
| Board       Name | GPU   Persist-M | Bus-id              | GPU-Util      sGPU-M |
| Pwr:Usage/Cap    | Temp       Perf | Memory-Usage        | GPU-State            |
|==================+=================+=====================+======================|
| 0      Mars X201 | 0           Off | 0000:98:00.0        | 0%          Disabled |
| 38W / 350W       | 44C          P0 | 858/65536 MiB       | Available            |
+------------------+-----------------+---------------------+----------------------+

+---------------------------------------------------------------------------------+
| Process:                                                                        |
|  GPU                    PID         Process Name                 GPU Memory     |
|                                                                  Usage(MiB)     |
|=================================================================================|
|  no process found                                                               |
+---------------------------------------------------------------------------------+

End of Log
```

# 二. 测试核函数
```c++
__global__ void gpu_fp64_complex_kernel(const double* input, double* output, int n) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < n) {
        double x = input[idx];
        double accum = 0.0;
        for (int i = 0; i < 100; i++) {
            double temp = x * (3.0 + i * 0.01) + 2.0 / (x + 1.5 + i * 0.005);
            // temp = temp - (x * 0.5 + i * 0.02) + tanh(x + i * 0.01);
            // temp += exp(sin(x * 0.1 + i * 0.01));
            // temp /= (cos(x * 0.05 + i * 0.02) + 1.5);
            // temp += log(fabs(x) + 1.0 + i * 0.001);
            // temp *= sqrt(fabs(x) + 0.1 + i * 0.003);
            accum += temp;
        }
        output[idx] = accum;
    }
}

__global__ void gpu_fp32_complex_kernel(const float* input, float* output, int n) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < n) {
        float x = input[idx];
        float accum = 0.0f;
        for (int i = 0; i < 100; i++) {
            float temp = x * (3.0f + i * 0.01f) + 2.0f / (x + 1.5f + i * 0.005f);
            // temp = temp - (x * 0.5f + i * 0.02f) + tanhf(x + i * 0.01f);
            // temp += expf(sinf(x * 0.1f + i * 0.01f));
            // temp /= (cosf(x * 0.05f + i * 0.02f) + 1.5f);
            // temp += logf(fabsf(x) + 1.0f + i * 0.001f);
            // temp *= sqrtf(fabsf(x) + 0.1f + i * 0.003f);
            accum += temp;
        }
        output[idx] = accum;
    }
}

// CPU 对应计算
double cpu_fp64_complex_compute(double x) {
    double accum = 0.0;
    for (int i = 0; i < 100; i++) {
        double temp = x * (3.0 + i * 0.01) + 2.0 / (x + 1.5 + i * 0.005);
        // temp = temp - (x * 0.5 + i * 0.02) + std::tanh(x + i * 0.01);
        // temp += std::exp(std::sin(x * 0.1 + i * 0.01));
        // temp /= (std::cos(x * 0.05 + i * 0.02) + 1.5);
        // temp += std::log(std::fabs(x) + 1.0 + i * 0.001);
        // temp *= std::sqrt(std::fabs(x) + 0.1 + i * 0.003);
        accum += temp;
    }
    return accum;
}
```
模拟*多层全连接神经网络一次前向传播计算量*
放开注释即为 *带三角函数等特殊运算*

# 三. 测试结果

**需要注意的是, RM-V100 是一卡双芯片,以下测试所有都只用到了 RM-V100 其中一颗计算核心,但其价格也相对昂贵,因此测试一半相对公平**

|               显卡                |          BW101           |     BW101(Kylin V10-arm)     |         RM-V00          |             X201             |                          说明                          |
| :-----------------------------: | :----------------------: | :--------------------------: | :---------------------: | :--------------------------: | :--------------------------------------------------: |
|               厂商                |            海光            |              海光              |          天数智芯片          |              沐曦              |                                                      |
|         是否存在 fp64 计算模块          |          厂商未说明           |            厂商未说明             |          厂商未说明          |            厂商未说明             |                                                      |
|         是否存在 fp32 计算模块          |            存在            |              存在              |           存在            |              存在              |                                                      |
|              显存频率               |      ***1800 MHz***      |        ***1800 MHz***        |        1600 MHz         |           1809 MHz           |                                                      |
|            核心计算时峰值频率            |         1350 MHz         |           1350 MHz           |     ***1600 MHz***      |                              |                                                      |
|              显存大小               |       ***65536***        |         ***65536***          |          32768          |         ***65536***          |                                                      |
|       与cpu相比 fp64 计算误差最大值       | *0.00000000000090949470* |    0.00000000000341060513    | 0.00875667685090775194  | ***0.00000000000068212103*** | 可能是 RM-V100 未进行三角函数相关计算导致的,***后经过测试 普通四则运算差距也是这么大*** |
|       与cpu相比 fp32 计算误差最大值       |  0.00118991340650609345  |    0.00079957676734920824    | 0.00088450437351639266  |    0.00103452777193524526    |                                                      |
|       与cpu相比 fp64 计算误差平均值       | *0.00000000000010842882* |    0.00000000000034830805    | 0.00412309574948392307  | ***0.00000000000003564082*** | 可能是 RM-V100 未进行三角函数相关计算导致的,***后经过测试 普通四则运算差距也是这么大*** |
|       与cpu相比 fp32 计算误差最大值       |  0.00014915226851994136  |    0.00007794356314451534    | 0.00014795610787007263  |    0.00010184497606479681    |                                                      |
|         是否支持 fp64 四则运算          |            支持            |              支持              |           支持            |              支持              |                                                      |
|       是否支持 fp64 三角函数等特殊运算       |         ***支持***         |           ***支持***           |           不支持           |           ***支持***           |            在测试用例中,涉及三角函数特殊预算 fp64返回结果全为 0            |
|         是否支持 fp32 四则运算          |            支持            |              支持              |           支持            |              支持              |                                                      |
|       是否支持 fp32 三角函数等特殊运算       |            支持            |              支持              |           支持            |              支持              |                                                      |
| fp64 带三角函数等特殊运算一次计算平均时间(单位:ms)  | *1.89906299114227294922* |   *1.62834620475769042969*   | 20.98007965087890625000 | ***1.49896776676177978516*** |     RM-V100 虽然不支持 fp64 三角函数特殊运算,但是它不会报错,继续运算,不会停     |
| fp32 带三角函数等特殊运算一次计算平均时间(单位:ms)  |  0.75064218044281005859  |    0.73794603347778320312    | 0.60118657350540161133  | ***0.64375907182693481445*** |                                                      |
| 带三角函数等特殊运算一次计算平均时间 fp64 : fp32  | *2.52744412422180175781* |   *2.20659255981445312500*   | 34.6879997253417968750  | ***2.32846069335937500000*** |                                                      |
|   fp64 普通四则运算一次计算平均时间(单位:ms)    | *0.07398736476898193359* | ***0.06997083127498626709*** | 1.50069546699523925781  |   *0.11261849850416183472*   |                                                      |
|   fp32 普通四则运算一次计算平均时间(单位:ms)    |  0.04850733280181884766  |    0.04637610539793968201    | 0.04620220884680747986  |   *0.07534234225749969482*   |                                                      |
|   普通四则运算一次计算平均时间 fp64 : fp32    | *1.52528202533721923828* |    1.50876903533935546875    | 32.48103332519531250000 | ***1.49712884426116943359*** |                                                      |
| 四方向中值滤波算子计算时长<br>(100x320x320)  |         *8.892*          |          *8.69461*           |         8.9348          |        ***8.27287***         |                                                      |
|  四方向中值滤波算子计算时长<br>(6000x6000)   |        *25.9315*         |        ***24.6478***         |        *25.9466*        |           40.1755            |                                                      |
| 四方向中值滤波算子计算时长<br>(16x6000x6000) |        *393.928*         |        ***392.752***         |         423.822         |          *398.923*           |                  天数智芯出现显存不足,分批计算4次                   |
|        fp32 Mean GFLOPS         |          43549           |           43672.7            |        *51526.3*        |        ***57496.6***         |                        **存疑**                        |
|        fp64 Mean GFLOPS         |        *43394.9*         |          *43517.7*           |         961.63          |        *** 57523.3***        |                        **存疑**                        |
### ***结论:***
 1. X201 和 BW101 的计算精度在 fp64 远高于 RM-V100
 2. X201 和 BW101 支持更多的在 fp64 的三角函数等特殊运算
 3. X201 和 BW101 在 fp64 计算速度远快于 RM-V100
 4. X201 和 BW101 ***显存大小是*** RM-V100 的 ***2倍***
 5. RM-V100 直接支持 CUDA, BW101 需要学习 hip_runtime, X201 需要学习 hc_runtime 但是函数有对 CUDA进行一一映射,几乎可说换个函数名前缀就行了
## ***总结:***
X201 和 BW101 在 fp64 计算上表现更好更稳定, 显存是 RM-V100 的2倍,但使用它需要一定量的学习, 从 CUDA 迁移过来有一定的成本, 其他层面上相差不大
X201 在 cuda 层面转换相对来说比较容易, 函数名替换就可以