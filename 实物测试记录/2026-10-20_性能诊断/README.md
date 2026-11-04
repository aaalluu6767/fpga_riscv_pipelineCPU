# 主工程 CoreMark 性能诊断上板记录

记录日期：2026-10-20。依据用户本轮提供的串口与 PDS 截图及资源数字；截图本身只显示串口时刻，不显示日期。原始截图保存在本目录的 `01`—`07` 文件中。未取得串口纯文本、PDS 原始报告或配置流哈希。

## 版本与条件

- 本次截图中的编译参数包括 GCC 15.2.0、`-march=rv32i_zicsr`、`-mabi=ilp32`、`-O2`、`ITERATIONS=0`、`PERFORMANCE_RUN=1`、`COREMARK_UART_GAP_CYCLES=540000`、`COREMARK_PROFILE=1`，与主工程导出的 `board_coremark_performance_uart20ms_profile.zip` 诊断配置一致。
- 用户反馈已编译并烧录。截图和口头数据未给出配置流哈希，因此尚不能独立证明烧录内容与 ZIP 逐字节对应。
- 目标时钟约束为 27 MHz；性能计数器版仅用于瓶颈诊断，不能直接与旧的非诊断上板版比较正式成绩或资源效率。

## 串口结果

| 字段 | 截图值 |
|---|---:|
| CoreMark Size | 666 |
| Total ticks | 371325 |
| Total time | 14.082844 s |
| Iterations | 400 |
| Iterations/Sec、CoreMark 1.0 | 28.403353 |
| CoreMark/MHz，按 27 MHz 计算 | 1.051976 |
| seedcrc / crclist / crcmatrix / crcstate | e9f5 / e714 / 1fd7 / 8e3a |
| crcfinal | 25b5 |

截图显示 `Correct operation validated.`，且 14.082844 秒满足赛题至少 10 秒的要求。`371325 × 1024 ÷ 27000000 = 14.0828444` 秒，与报告相符。

| 诊断计数器 | 截图值 | 占 Profile cycles |
|---|---:|---:|
| Profile cycles | 380236702 | 100% |
| Retired instructions | 292793398 | 77.003% |
| Memory wait cycles | 27735507 | 7.294% |
| RAW wait cycles | 0 | 0% |
| Branch instructions | 79318021 | 20.860% |
| Taken branches | 49216962 | 12.944% |
| Branch bubble cycles | 49216962 | 12.944% |
| Jump bubble cycles | 10490835 | 2.759% |

上述百分比都以 Profile cycles 为分母；这些事件计数有重叠或口径差异，不能直接相加为总周期。分支空拍是目前最显著的单项线索。`Profile cycles` 与 CoreMark 计时器边界相差约 98 拍，符合工程 README 中对两种计数边界的说明。

## 资源与时序

| PDS 资源 | 已用 / 总量 |
|---|---:|
| LUT | 1810 / 159800 |
| FF | 781 / 319600 |
| IO | 15 / 400 |
| DRAM | 32 / 39500 |
| DRM | 23 / 415 |
| APM | 0 / 740 |

PDS 截图显示 `All Constraints Met.`；27 MHz 约束下建立余量 +14.812 ns、保持余量 +0.11 ns，失败端点均为 0。界面显示可达 44.9944 MHz，这是工具报告值，未在该频率烧录验证。按截图数字计算的诊断版 CoreMark/LUT 为 0.015692，仅供本版本内部记录；不可用它与旧版非诊断资源效率直接比较。

## 后续所需证据

保存此次配置流、PDS 原始资源/时序报告、完整串口文本及其哈希，建立可追溯的同一版本测量。随后根据分支空拍与访存等待做一项有回退版本的优化，并以相同条件复测。
