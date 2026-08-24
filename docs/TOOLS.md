# BehaveScope 工具链清单（TOOLS.md）

> 四引擎依赖的工具集合 · 本地资产盘点 + 待收集清单
> 更新日期: 2026-08-24 · 目标平台: Windows 优先（客户 ROG 幻16 Win11），开发平台 macOS

## 收集状态图例

- ✅ **READY** — 本地已有，可用
- 🔶 **PENDING** — 待下载（网络恢复后补，见 TODO）
- 🔧 **BUILD** — 需构建/打包（Windows 便携化）

---

## 一、静态引擎（Static）

| 工具 | 版本 | 用途 | 平台 | 状态 | 路径/来源 |
|---|---|---|---|---|---|
| Ghidra headless | 12.1 | 反编译/函数级分析 | win/mac/linux | ✅ READY | `~/tools/ghidra_12.1_PUBLIC/support/analyzeHeadless`（含 .bat win版） |
| capstone | 5.0.6 | 反汇编引擎 | python | ✅ READY | pip 已装 |
| pefile | 2024.8.26 | PE 解析（导入表/节表/特征） | python | ✅ READY | pip 已装 |
| angr | 9.2.223 | 符号执行/自动化分析 | python | ✅ READY | pip 已装（unicorn 支持关，功能可用） |
| z3-solver | 4.13.0.0 | 约束求解（密码/混淆） | python | ✅ READY | pip 已装 |
| binwalk | 2.1.0 | 固件/文件解包、魔数识别 | python | ✅ READY | pip 已装 |
| 7z | — | 压缩包/复合文档解包 | win/mac | ✅ READY | `/opt/homebrew/bin/7z`（win: 7-Zip 便携） |
| file | — | 文件类型识别 | win/mac | ✅ READY | 系统自带 |
| strings | — | 字符串提取 | win/mac | ✅ READY | 系统自带 |
| **capa** | — | 能力识别（攻击技术标注） | win/mac | 🔶 PENDING | github.com/mandiant/capa releases |
| **FLOSS** | — | 混淆字符串提取 | win/mac | 🔶 PENDING | github.com/mandiant/flare-floss releases |
| **DIE (Detect It Easy)** | — | 加壳/编译器检测 | win | 🔶 PENDING | github.com/horsicq/Detect-It-Easy |
| **yara** | 4.5.8 | 规则匹配引擎 | win/mac | 🔶 PENDING | brew yara（pip 绑定 TLS 断，走 brew 或 win 版） |

## 二、动态引擎（Dynamic）

| 工具 | 版本 | 用途 | 平台 | 状态 | 路径/来源 |
|---|---|---|---|---|---|
| frida + frida-tools | 17.15.3 / 14.10.4 | API hook / 进程内监控 | win/mac/linux | ✅ READY | pip 已装（win 需装对应 win 版） |
| DynamoRIO | — | 动态插桩框架 | win/linux | ✅ READY | `~/tools/instrumentation/tools/DynamoRIO` |
| TinyInst | — | 轻量插桩（Windows 主） | win | ✅ READY | `~/tools/instrumentation/tools/TinyInst` |
| pin | — | Intel 插桩 | win/linux | ✅ READY | `~/tools/instrumentation/tools/pin` |
| valgrind | — | 内存/调用跟踪 | mac/linux | ✅ READY | `~/tools/instrumentation/tools/valgrind` |
| Windows Sandbox | — | 隔离沙箱（动态运行样本） | win11 Pro+ | 🔧 BUILD | 系统功能，.wsb 配置脚本化调用 |
| **procmon / sigcheck** | — | Sysinternals 监控兜底 | win | 🔶 PENDING | learn.microsoft.com/sysinternals |

## 三、取证引擎（Forensic）

| 工具 | 版本 | 用途 | 平台 | 状态 | 路径/来源 |
|---|---|---|---|---|---|
| volatility3 | 2.28.0 | 内存镜像分析（--offline 离线） | win/mac/linux | ✅ READY | `vol`（pip） |
| memprocfs | — | 实时内存分析（Volatility 兼容） | win | ✅ READY | `~/tools/instrumentation/tools/memprocfs` |
| lime | — | Linux 内存采集 | linux | ✅ READY | `~/tools/instrumentation/tools/lime` |
| psutil | 7.2.2 | 进程/系统信息采集 | python | ✅ READY | pip 已装 |
| **Rekall** | — | 内存取证补充 | python | 🔶 PENDING | github.com/google/rekall |

## 四、流量引擎（Network）

| 工具 | 版本 | 用途 | 平台 | 状态 | 路径/来源 |
|---|---|---|---|---|---|
| tcpdump | — | 抓包 | mac/linux | ✅ READY | 系统自带（win 用 npcap+windump） |
| **scapy** | — | pcap 解析/协议分析 | python | 🔶 PENDING | pip（TLS 断待恢复） |
| **tshark** | — | pcap 深度解析（C2/隧道） | win/mac | 🔶 PENDING | wireshark.org（win 便携版） |
| **Wireshark 便携** | — | 图形化兜底 | win | 🔶 PENDING | wireshark.org |

## 五、AI 引擎（可选，离线 LLM）

| 工具 | 版本 | 用途 | 平台 | 状态 | 路径/来源 |
|---|---|---|---|---|---|
| **Qwen3.5-9B GGUF** | Q4_K_M (5.6GB) | 行为解读/报告润色 | win/mac | ✅ READY | ollama 本地（`ZimaBlueAI/Qwen3.5-9B-DeepSeek-V4-Flash-GGUF`），262K ctx |
| llama.cpp | — | GGUF 推理（CUDA） | win/mac | ✅ READY(mac) / 🔧 BUILD(win CUDA) | brew llama-cli；win 需打包 Blackwell 兼容版（见风险） |

> ⚠️ **RTX 5080 (Blackwell sm_120) 已知坑**：必须 CUDA 12.8+ / 新版 llama.cpp（PR #27215 修复 sharedMemPerBlockOptin 驱动 bug）；CUDA 13.2 某些量化（IQ3/IQ2）出乱码 → 用 CUDA 12.8 打包。Q4_K_M 5.63GB / Q8_0 9.53GB 均装得进 16GB VRAM。

## 六、开发/打包（Build）

| 工具 | 版本 | 用途 | 平台 | 状态 | 路径/来源 |
|---|---|---|---|---|---|
| pwntools | 4.15.0 | PWN/exploit 辅助 | python | ✅ READY | pip 已装 |
| uncompyle6 | — | Python 反编译（pyc 样本） | python | ✅ READY | hermes venv |
| **pyinstaller** | — | Windows 便携 exe 打包 | win | 🔶 PENDING | ⚠️ 必须 Windows 环境构建（不能跨编译）；用 **onedir** 模式降低 AV 误报 |
| **Nuitka** | — | 备选打包（快/难逆向） | win | 🔶 PENDING | 视需要 |

---

## Windows 便携版获取速查（客户 ROG 现场组装用）

| 工具 | Windows 便携形态 | 获取方式 |
|---|---|---|
| Ghidra | 官方 zip（含 analyzeHeadless.bat） | github.com/NationalSecurityAgency/ghidra/releases |
| volatility3 | pip 包（Python 跨平台） | `pip install volatility3`（win 上同样可用） |
| frida | pip 包 + frida-server.exe | `pip install frida frida-tools` |
| 7-Zip | 便携版 7zr.exe/7za.exe | 7-zip.org |
| capa | capa-vX-win64.zip（自带 python 环境） | github.com/mandiant/capa/releases |
| FLOSS | floss-vX-windows.zip | github.com/mandiant/flare-floss/releases |
| DIE | die_win64_portable_x.x.x.zip | github.com/horsicq/Detect-It-Easy/releases |
| yara | yara-x win64 或 pyyara wheel | github.com/VirusTotal/yara-x/releases |
| tshark/Wireshark | WiresharkPortable64 | www.wireshark.org/download.html |
| Sysinternals | SysinternalsSuite.zip（procmon/sigcheck） | learn.microsoft.com/sysinternals/downloads |
| llama.cpp (CUDA) | llama-bXXXX-bin-win-cuda-12.4-x64.zip | github.com/ggml-org/llama.cpp/releases（选 CUDA 12.8+ 版） |
| Python 便携 | embeddable zip + pip | python.org（可选，PyInstaller 后无需） |

> 组装原则：**现场 U 盘只带 onedir 打包的 behave.exe + rules/ + ai/（可选模型）**，其余工具按需作为兜底。全部绿色免安装。

## 待下载 TODO（网络恢复后执行）

```bash
# 统一走代理（确认 Surge/privoxy 恢复后）
export https_proxy=http://127.0.0.1:1087 http_proxy=http://127.0.0.1:1087

# Python 依赖（清华源备用）
pip3 install yara-python scapy   # 或 -i https://pypi.tuna.tsinghua.edu.cn/simple

# brew 二进制
brew install yara

# GitHub releases（capa/floss/DIE 有 win 便携版）
#   capa:   https://github.com/mandiant/capa/releases   (capa-vX-win64.zip)
#   FLOSS:  https://github.com/mandiant/flare-floss/releases (floss-vX-windows.zip)
#   DIE:    https://github.com/horsicq/Detect-It-Easy/releases (die_win64_portable)
#   tshark: https://www.wireshark.org/download.html (WiresharkPortable64)
```

## 七、竞品与参考工具生态（2026-08-24 全网扫描）

### 7.1 开源沙箱平台

| 工具 | 离线? | 免费? | 平台 | 特点 | 来源 |
|---|---|---|---|---|---|
| **CAPEv2** | ✅ 本地部署 | ✅ | Linux+KVM | 最广泛使用的开源沙箱，提取配置/payload/YARA分类 | github.com/kevoreilly/CAPEv2 |
| **Cuckoo3** | ✅ | ✅ | Python/KVM | CAPE 上游重写版，Python3，活跃开发中 | github.com/cuckoosandbox/cuckoo |
| **DRAKVUF Sandbox** | ✅ | ✅ | Xen/HVM | agentless（hypervisor 层监控），恶意软件无法检测 | github.com/tklengyel/drakvuf |
| **detux** | ✅ | ✅ | Linux | Linux 恶意软件流量分析+IOC捕获沙箱 | github.com/detuxsandbox/detux |
| **Limon** | ✅ | ✅ | Python | Linux 恶意软件分析沙箱 | github.com/monnappa22/Limon |

### 7.2 预制分析环境（VM/Docker）

| 工具 | 离线? | 平台 | 特点 | 来源 |
|---|---|---|---|---|
| **REMnux** | ✅ Linux发行版+Docker | 200+恶意分析工具（radare2/retdec/thug/volatility等） | remnux.org / Docker Hub |
| **FLARE VM** | ✅ Windows发行版 | Mandiant出品，70+工具通过Chocolatey安装到WinVM | github.com/mandiant/flare-vm |
| **REMnux MCP Server** | ✅ | 连接AI代理到所有REMnux工具（workflow编码） | REMnux生态 |

### 7.3 静态分析工具（竞品常用）

| 工具 | 离线? | 平台 | 特点 | 来源 |
|---|---|---|---|---|
| **Ghidra** | ✅ | win/mac/linux | NSA逆向套件，50+架构反汇编/反编译 | github.com/NationalSecurityAgency/ghidra |
| **radare2 / Cutter** | ✅ | 全平台 | 开源逆向框架+Cutter GUI（Qt） | github.com/radareorg/radare2 |
| **IDA Pro** | 💰商业 | win/mac/linux | 业界标准逆向工具 | hex-rays.com |
| **capa** | ✅ | python/win | Mandiant出品，恶意软件能力检测（MITRE ATT&CK标注） | github.com/mandiant/capa |
| **FLOSS** | ✅ | python/win | Mandiant FLARE，混淆字符串自动提取 | github.com/mandiant/flare-floss |
| **PE-bear** | ✅ | Windows | PE文件图形分析（头/节/导入导出/异常检测） | hshrzd.wordpress.com/pe-bear |
| **Detect It Easy (DIE)** | ✅ | win/mac/linux | 加壳/编译器/链接器检测 | github.com/horsicq/Detect-It-Easy |
| **PEStudio** | ✅ | Windows | PE静态分析+VT结果（免费非开源） | winitor.com |
| **Manalyze** | ✅ | C++/python | PE文件静态分析 | github.com/JusticeRage/Manalyze |
| **MASTIFF** | ✅ | Python | 静态分析框架（多格式） | github.com/KoreLogicSecurity/mastiff |
| **MultiScanner** | ✅ | Python | 模块化文件扫描/分析框架 | github.com/MITRECND/multiscanner |
| **peframe** | ✅ | Python | PE文件与Office文档静态分析 | github.com/guelfoweb/peframe |
| **PortEx** | ✅ | Java | PE文件Java库 | github.com/katjahahn/PortEx |
| **Nauz File Detector** | ✅ | win/mac/linux | 跨平台链接器/编译器检测 | github.com/horsicq/Nauz-File-Detector |

### 7.4 动态分析工具

| 工具 | 离线? | 平台 | 特点 | 来源 |
|---|---|---|---|---|
| **x64dbg** | ✅ | Windows | x64/x32开源调试器，OllyDbg替代品+插件生态 | github.com/x64dbg/x64dbg |
| **WinDbg** | ✅ | Windows | 微软多用途调试器（内核模式） | docs.microsoft.com/windbg |
| **Speakeasy** | ✅ | Python | Mandiant Windows内核/用户态模拟器（shellcode/PE） | github.com/mandiant/speakeasy |
| **Noriben** | ✅ | Python | 利用Procmon收集沙箱环境下的进程信息 | github.com/Rurik/Noriben |
| **Pafish** | ✅ | C | 检测沙盒/分析环境的PoC恶意软件（反调试） | github.com/a0rtega/pafish |
| **al-khaser** | ✅ | C++ | 反恶意软件系统检测PoC | github.com/LordNoteworthy/al-khaser |

### 7.5 取证与DFIR工具

| 工具 | 离线? | 平台 | 特点 | 来源 |
|---|---|---|---|---|
| **volatility3** | ✅ | python/win/mac/linux | 内存镜像分析（--offline模式） | github.com/volatilityfoundation/volatility3 |
| **Autopsy / Sleuth Kit** | ✅ | Java/Python | 开源数字取证平台+文件系统分析 | autopsy.sleuthkit.org |
| **FTK Imager** | ✅ | Windows | 镜像采集与查看（免费） | accessdata.com |
| **Eric Zimmerman Tools** | ✅ | Windows | Win取证工具全家桶（RECmd/PECmd/EvtxECmd等） | github.com/EricZimmerman |
| **Plaso (log2timeline)** | ✅ | Python | 时间线分析工具 | github.com/log2time/plaso |
| **Hayabusa** | ✅ | Rust | Windows事件日志威胁狩猎 | github.com/Yamato-Security/hayabusa |
| **Loki IOC Scanner** | ✅ | Python | Florian Roth的IOC+YARA扫描器 | github.com/Neo23x0/Loki |
| **Rekall** | ✅ | Python | Google出品内存取证框架 | github.com/google/rekall |

### 7.6 文档/脚本分析

| 工具 | 离线? | 平台 | 特点 | 来源 |
|---|---|---|---|---|
| **oletools** | ✅ | Python | Microsoft Office文档恶意宏/VBA/OLE检测 | github.com/decalage2/oletools |
| **ViperMonkey** | ✅ | Python | VBA宏模拟器（静态去混淆+模拟执行） | github.com/decalage2/ViperMonkey |
| **pdfid/pdf-parser** | ✅ | Python | DidierStevens PDF恶意分析工具 | github.com/DidierStevens/DidierStevensSuite |
| **jadx / jadx-gui** | ✅ | Java | APK反编译（含MCP插件） | github.com/skylot/jadx |

### 7.7 CTF取证专用（竞品参考：阿乐AFST等）

| 工具 | 离线? | 平台 | 特点 | 来源 |
|---|---|---|---|---|
| **Ale Forensic Suite Toolkit (AFST)** | ✅ Windows | 25类工具分类，80+取证工具，AI辅助（Claude MCP） | cn-sec.com/archives/5319584 |
| **StegoVeritas** | ✅ | Python | 自动化隐写分析 | github.com/InQuest/stegoVeritas |
| **zsteg** | ✅ | Ruby | PNG/BMP LSB隐写检测 | github.com/zed-0xff/zsteg |
| **steghide / stegseek** | ✅ | C | JPG/WAV隐写（支持密码爆破） | steghide.sourceforge.net |
| **stegsolve** | ✅ | Java | 图像位平面/颜色通道分析 | github.com/zed-0xff/stegsolve |
| **exiftool** | ✅ | Perl | 元数据提取（100+格式） | exiftool.org |
| **binwalk** | ✅ | Python | 固件解包+魔数识别 | github.com/ReFirmLabs/binwalk |
| **foremost / scalpel** | ✅ | C | 文件雕刻恢复 | foremost.sourceforge.net |

### 7.8 在线沙箱（不可用于比赛现场，但了解竞品格局）

| 工具 | 联网? | 特点 | 来源 |
|---|---|---|---|
| **ANY.RUN** | ✅ 必须联网 | 交互式在线沙箱 | any.run |
| **Hybrid Analysis** | ✅ | VxSandbox支持，免费分析 | github.com/Forcepoint/hybrid-analysis |
| **Joe Sandbox** | ✅ | 深度恶意软件分析（商业） | joesecurity.org |
| **微步云沙箱** | ✅ 中国 | 国内主流在线沙箱 | threatbook.cn |
| **VirusTotal** | ✅ | 多引擎扫描+URL分析 | virustotal.com |

### 7.9 AI辅助分析（竞品趋势）

| 工具 | 离线? | 特点 | 来源 |
|---|---|---|---|
| **GhidraMCP / GhidrAssist** | ✅ | AI代理驱动Ghidra逆向工程 | REMnux生态 |
| **r2ai** | ✅ | radare2的AI插件 | github.com/ahellier/r2ai |
| **REMnux MCP Server** | ✅ | AI代理连接所有REMnux工具（workflow编码） | REMnux生态 |

---

## 八、BehaveScope 差异化定位（基于竞品分析）

| 维度 | CAPEv2/REMnux/FLARE VM | AFST取证包 | capa/FLOSS/DIE | **BehaveScope** |
|---|---|---|---|---|
| **离线可用** | ✅ 但需部署VM | ✅ VM镜像 | ✅ 单工具 | ✅ **一条命令自动闭环** |
| **自动化程度** | ⚠️ 需配置/脚本化 | ⚠️ 工具合集，人肉点用 | ❌ 单点工具 | ✅ **四引擎自动路由+报告生成** |
| **CTF场景优化** | ⚠️ 偏企业安全 | ✅ CTF取证 | ❌ 不面向CTF | ✅ **专为比赛现场设计（秒级静态→分钟级动态）** |
| **本地AI集成** | ⚠️ REMnux有MCP但需联网 | ✅ Claude MCP（需联网API） | ❌ | ✅ **离线Qwen3.5-9B GGUF（5080可跑）** |
| **便携性** | ❌ VM/Docker重 | ⚠️ 大镜像/VM | ✅ 单工具 | ✅ **U盘绿色目录，一条命令启动** |
| **报告输出** | ⚠️ JSON/HTML需配置 | ⚠️ 各工具格式不一 | ❌ 无 | ✅ **统一JSON+MD+HTML可提交报告** |

> **核心差异化总结**：BehaveScope = 离线自动分析闭环（非工具合集）+ CTF现场优化（非企业安全）+ 本地AI推理（非联网MCP）+ U盘绿色便携（非VM部署）。

## 风险登记

| 风险 | 等级 | 缓解 |
|---|---|---|
| 网络 TLS 全断（Surge 代理黑洞，2026-08-24 实测） | 🟠 | 恢复后批量下载；工具收集不阻塞核心开发 |
| RTX 5080 Blackwell 跑 LLM 需新版 llama.cpp | 🔴 | M4 里程碑必测项：打包 CUDA 12.8 版 + 确认驱动 ≥595.84 |
| PyInstaller 不能跨平台编译 | 🟠 | 用 GitHub Actions windows runner 或客户 ROG 本机打包 |
| PyInstaller onedir AV 误报 | 🟡 | onedir 模式（非 onefile）+ 重编译 bootloader 降误报 |
| Windows Sandbox 需 Win11 专业版/企业版 | 🟡 | 需确认客户系统版本；家庭版降级 frida+ETW 方案 |
