# BehaveScope 架构设计（DESIGN.md）

> BehaveScope v0.1 四引擎自动路由架构 · 2026-08-24
> 目标平台: Windows 优先（客户 ROG 幻16 Win11），开发平台 macOS

---

## 1. 产品定位

**BehaveScope** = CTF/应急/红队现场的离线软件行为分析器。一条命令拿到任何样本后，自动完成静态→动态→取证→流量四引擎分析，输出可提交的结构化报告（JSON + Markdown + HTML）。

### 核心差异化
- **离线闭环**：纯本地运行，无需网络、无需云端API、无需部署VM
- **自动路由**：根据输入文件类型自动选择/组合引擎
- **CTF现场优化**：秒级静态分析 → 分钟级动态监控 → 一键出报告
- **U盘绿色便携**：单目录解压即用，`.bat` 一键启动

---

## 2. 整体架构

```
behave-scope/
├── start.bat              # Windows 一键启动（set PATH + invoke behave.py）
├── behave.exe             # PyInstaller onedir 打包的主程序
├── engine/
│   ├── __init__.py
│   ├── router.py          # 自动路由引擎（根据文件类型选择分析路径）
│   ├── static/            # 静态引擎
│   │   ├── analyzer.py    # 主分析器：hash/entropy/packer/strings/import_table/yara
│   │   ├── pe_analyzer.py # PE专项：节表/导入/导出/资源/RVA偏移
│   │   ├── elf_analyzer.py# ELF专项（Linux/Mach-O 交叉分析）
│   │   └── apk_analyzer.py# APK专项（AndroidManifest/smali解析）
│   ├── dynamic/           # 动态引擎
│   │   ├── sandbox_runner.py  # Windows Sandbox .wsb 沙箱编排
│   │   ├── frida_monitor.py   # Frida API hook 监控（降级方案）
│   │   └── etw_capture.py     # ETW 事件捕获（备用）
│   ├── forensic/          # 取证引擎
│   │   ├── mem_forensics.py   # volatility3 --offline 内存分析
│   │   ├── registry_parser.py # Windows注册表离线解析
│   │   └── timeline_builder.py# 时间线重建（Eric Zimmerman集成）
│   └── network/           # 流量引擎
│       ├── pcap_analyzer.py   # PCAP解析：C2/webshell/隧道特征
│       └── protocol_decoder.py# 协议深度解码
├── rules/                 # YARA + 行为特征规则库（随包）
│   ├── yara/              # YARA规则文件
│   │   ├── malware_families.yar    # 已知恶意家族特征
│   │   ├── packer_signatures.yar   # 加壳签名
│   │   └── ctf_flag_patterns.yar   # CTF flag格式模式
│   └── behavioral/        # 行为特征库
│       ├── dns_tunnel.rules     # DNS隧道检测
│       ├── c2_beacon.rules      # C2信标检测
│       └── webshell.rules       # Webshell特征
├── ai/                    # 离线AI引擎（可选）
│   ├── llm_engine.py        # llama.cpp GGUF推理封装
│   └── report_writer.py     # AI辅助报告润色
├── tools/                 # 第三方工具便携版（打包后目录）
│   ├── ghidra/            # Ghidra headless（含analyzeHeadless.bat）
│   ├── volatility3/       # vol3 + plugins
│   ├── frida-server/      # frida-server.exe (win x64)
│   └── ...
├── scripts/               # 辅助脚本
│   ├── sandbox_setup.ps1  # Windows Sandbox 配置脚本
│   └── evidence_collector.sh  # 证据收集通用脚本
├── docs/                  # 文档
│   ├── README.md
│   ├── TOOLS.md           # 工具链清单（已推送）
│   ├── DESIGN.md          # 本文件
│   └── USAGE.md           # 使用说明
└── tests/                 # 测试
    ├── test_static.py
    ├── test_router.py
    └── samples/           # 测试样本（无害）
```

---

## 3. CLI 接口设计

### 主命令
```bash
# Windows 一键启动
start.bat sample.exe          # → report.json + report.md + evidence/

# 或直接调用 Python
python behave.py <target>     # 自动路由分析
```

### 子命令（可选）
```bash
behave static sample.exe      # 仅静态分析
behave dynamic sample.exe     # 动态沙箱运行
behave forensic mem.dmp       # 内存取证
behave network traffic.pcap   # 流量分析
behave all sample.exe         # 四引擎全开
```

### 输出格式
```bash
behave sample.exe --format json   # JSON报告（默认）
behave sample.exe --format md     # Markdown报告
behave sample.exe --format html   # HTML可视化报告
behave sample.exe --flag          # 仅提取flag（CTF场景）
```

### 输出结构
```json
{
  "metadata": {
    "tool": "BehaveScope",
    "version": "0.1",
    "timestamp": "2026-08-24T12:00:00Z",
    "sample_hash": {"md5": "...", "sha1": "...", "sha256": "..."}
  },
  "static_analysis": {
    "file_type": "PE32 executable",
    "entropy": 7.2,
    "packer": "UPX detected",
    "strings_count": 1247,
    "imports": ["kernel32.dll", "advapi32.dll"],
    "yara_matches": ["malware_families:Emotet_variant"]
  },
  "dynamic_analysis": {
    "processes_spawned": ["cmd.exe", "powershell.exe"],
    "files_created": ["C:\\Users\\Public\\dropper.exe"],
    "registry_modified": ["HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Run"],
    "network_connections": [{"dst": "185.123.45.67:443", "type": "HTTPS"}]
  },
  "forensic_analysis": {
    "memory_artifacts": ["hidden_process_listed_in_psscan"],
    "registry_hives_analyzed": ["SYSTEM", "SAM", "SOFTWARE"]
  },
  "network_analysis": {
    "c2_indicators": [{"type": "DNS_tunnel", "confidence": 0.85}],
    "webshell_detected": false,
    "protocols": ["HTTP/1.1", "DNS", "HTTPS/TLS1.3"]
  },
  "ai_summary": {
    "behavior_classification": "Dropper with C2 capability",
    "threat_level": "HIGH",
    "recommendation": "Isolate and submit to sandbox for deeper analysis"
  },
  "flag_candidates": [
    {"text": "CTF{malware_analysis_is_fun}", "source": "strings_yara_match"}
  ]
}
```

---

## 4. 引擎详细设计

### 4.1 自动路由（router.py）

根据输入文件类型自动选择分析路径：

| 文件扩展名 | 引擎组合 | 预期时间 |
|---|---|---|
| `.exe/.dll/.sys` | static → dynamic → forensic | 3-5min |
| `.pdf/.docx/.xlsx` | static + oletools | 10-30s |
| `.pyc/.py` | static + uncompyle6/floss | 5-15s |
| `.apk/.dex` | static + apk_analyzer | 10-20s |
| `.dmp/.raw/.mem` | forensic only | 30-60s |
| `.pcap/.pcapng` | network only | 10-30s |
| 目录 | 批量扫描（所有文件类型） | 按文件大小 |

### 4.2 静态引擎（秒级完成）

**核心流程**：
1. **文件识别**：file + magic bytes → 精确格式判定
2. **哈希计算**：MD5/SHA1/SHA256/SHA512 + SSDEEP模糊哈希
3. **熵值分析**：分节熵值（高熵=加密/压缩段）
4. **加壳检测**：DIE签名 + YARA packer_signatures + PE异常特征
5. **字符串提取**：strings -eE（ASCII+Unicode）→ 可疑关键词匹配
6. **导入表分析**：pefile解析 → API调用链标注（suspicious imports）
7. **YARA扫描**：rules/yara/ 全部规则 → 家族/加壳/CTF特征匹配
8. **行为特征**：异常节名、可疑RVA、导出函数、资源段分析

**输出优先级**：
- P0: YARA命中 + CTF flag候选
- P1: 可疑导入 + 高熵段 + 混淆字符串
- P2: 完整分析报告

### 4.3 动态引擎（分钟级完成）

**首选方案：Windows Sandbox**
```
.start.bat → 检测Sandbox支持 → .wsb配置 → 自动运行样本 → Procmon/ETW捕获 → 停止沙箱 → 收集证据
```

**降级方案：Frida + ETW**
- 如果Sandbox不可用（家庭版Win10），自动降级到frida-server注入监控
- 继续降级：仅ETW事件捕获（无需注入）
- 最低降级：仅静态分析（无动态能力时）

**监控指标**：
- 进程创建/终止（CreateProcess, ExitProcess）
- 文件操作（CreateFile, WriteFile, DeleteFile）
- 注册表修改（RegSetValue, RegCreateKey）
- 网络活动（WinHttpSendRequest, sendto, connect）
- 内存操作（VirtualAlloc, WriteProcessMemory）
- 服务安装/启动

### 4.4 取证引擎

**内存镜像分析**：
- volatility3 --offline + psscan/skpcan/malfind/hivelist
- 隐藏进程/线程检测
- 注册表hive提取与解析
- 时间线重建（Eric Zimmerman Tools集成）

**注册表离线分析**：
- SYSTEM/SAM/SOFTWARE hives → 启动项/服务/USB设备历史
- NTUSER.dat → 用户行为痕迹
- 事件日志（EvtxECmd + Hayabusa IOC扫描）

### 4.5 流量引擎

**PCAP深度解析**：
- C2信标检测（周期性DNS/HTTP请求模式）
- Webshell特征匹配（命令执行模式）
- DNS隧道检测（TXT记录异常长度/频率）
- 协议解码（HTTP/HTTPS/DNS/SMB/IRC）
- IOC提取（IP/域名/URL/文件哈希）

---

## 5. CTF场景优化

### 5.1 Flag提取策略
```python
# 自动扫描所有输出中的flag候选
patterns = [
    r'flag\{[^}]+\}',
    r'CTF\{[^}]+\}',
    r'Diver\d+\{[^}]+\}',
    r'[a-f0-9]{32,}',      # 长hex字符串
    r'base64:[^\s]+',       # base64编码
]

# YARA规则库包含CTF flag格式模式
# rules/yara/ctf_flag_patterns.yar
```

### 5.2 AWD对抗支持
- 自动检测Webshell上传特征
- 进程监控异常启动（cmd.exe/powershell从Web目录）
- 文件创建时间线 → 快速定位被植入文件
- 网络外连检测 → 快速发现后门连接

---

## 6. 离线设计要点

### 6.1 离线三要素
1. **纯本地运行**：零外部依赖，无需任何API调用
2. **内置规则库**：YARA + 行为特征随包（rules/目录）
3. **绿色便携**：单目录解压即用，U盘拷贝

### 6.2 Windows Sandbox配置
```xml
<!-- sandbox.wsb -->
<Configuration>
  <VMConnections>
    <MappedFolder>
      <HostFolder>C:\samples</HostFolder>
      <SandboxFolder>C:\shared</SandboxFolder>
    </MappedFolder>
  </VMConnections>
  <Networking>DefaultNesting</Networking>
</Configuration>
```

### 6.3 AI引擎（可选）
- **离线模型**：Qwen3.5-9B GGUF Q4_K_M（5.6GB）
- **推理**：llama.cpp CUDA版（RTX 5080 Blackwell）
- **用途**：行为分类 + 报告润色（非必需，降级为规则引擎）

---

## 7. 里程碑规划

| 阶段 | 天数 | 交付物 |
|---|---|---|
| M1: 静态引擎+报告 | 2天 | `behave sample.exe → report.json+report.md` |
| M2: 动态引擎（Sandbox→Frida降级） | 2天 | `behave dynamic sample.exe` 完整流程 |
| M3: 取证+流量引擎 | 1.5天 | `behave forensic/ network` 子命令 |
| M4: 打包+AI集成+测试 | 1.5天 | PyInstaller onedir + rules库 + AI可选 |

---

## 8. 技术栈总览

| 层级 | 技术选型 | 理由 |
|---|---|---|
| 语言 | Python 3.10+ (win) / 3.12+ (mac dev) | 生态丰富，CTF选手熟悉 |
| PE解析 | pefile | 纯Python，无C依赖 |
| 反汇编 | capstone + angr | 符号执行+自动化分析 |
| YARA | yara-python / yara-x | 业界标准规则引擎 |
| 沙箱 | Windows Sandbox (.wsb) | Win11原生，轻量隔离 |
| 动态监控 | Frida 17.15 | 成熟API hook框架 |
| 内存取证 | volatility3 2.28 | --offline模式支持 |
| 流量分析 | scapy + tshark CLI | PCAP解析+协议解码 |
| AI推理 | llama.cpp GGUF (CUDA) | RTX 5080 Blackwell优化 |
| 打包 | PyInstaller onedir | 绿色目录，低AV误报 |

---

## 9. 风险与缓解

| 风险 | 等级 | 缓解策略 |
|---|---|---|
| RTX 5080 Blackwell CUDA兼容性 | 🔴 | M4必测：CUDA 12.8 + 新版llama.cpp + 驱动≥595.84 |
| Windows Sandbox 家庭版不可用 | 🟡 | 自动降级frida+ETW方案（复用memshell-auditor降级哲学） |
| PyInstaller AV误报 | 🟡 | onedir模式 + 重编译bootloader降误报 |
| 比赛现场网络隔离 | ✅ | 纯离线设计，规则库随包 |
| 样本过大导致分析超时 | 🟢 | 静态秒级、动态可配置超时（默认5min）、分块处理 |

---

## 10. 竞品对比总结

基于全网扫描（见TOOLS.md第七章），BehaveScope 的核心差异化：

| 维度 | CAPEv2/REMnux | AFST取证包 | capa/FLOSS/DIE | **BehaveScope** |
|---|---|---|---|---|
| 离线可用 | ✅需部署VM | ✅VM镜像 | ✅单工具 | ✅**一条命令自动闭环** |
| 自动化程度 | ⚠️需配置 | ⚠️人肉点用 | ❌单点工具 | ✅**四引擎自动路由+报告** |
| CTF场景优化 | ⚠️偏企业安全 | ✅CTF取证 | ❌不面向CTF | ✅**专为比赛现场设计** |
| 本地AI集成 | ⚠️需联网MCP | ✅Claude MCP(联网) | ❌ | ✅**离线Qwen3.5-9B GGUF** |
| 便携性 | ❌VM/Docker重 | ⚠️大镜像/VM | ✅单工具 | ✅**U盘绿色目录** |
| 报告输出 | ⚠️需配置 | ⚠️各工具格式不一 | ❌无 | ✅**统一JSON+MD+HTML** |

---

*BehaveScope v0.1 · x7peeps · 2026-08-24*