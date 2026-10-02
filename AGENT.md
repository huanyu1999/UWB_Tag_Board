# UWB Tag Hardware Plan

## Project Status

### Phase 1: Project framework — complete

- KiCad 10 project
- Single-sheet schematic; do not create hierarchical sheets
- Net naming rules

### Phase 2: STM32 minimum system — planned

MCU: STM32G071CBU3 (LCSC C724076), confirmed by the user.

Include:

- Decoupling capacitors
- NRST circuit
- SWD interface
- Power pins
- Clock circuit

### Phase 3: Peripherals — planned

Add in this order:

1. User-supplied UWB RF module (model and specification pending); do not assume a discrete DW3000
2. SPI Flash (RF configuration, firmware-update staging, and LCD resources; no measurement logs)
3. LCD
4. IMU
5. Power management

## Confirmed and Tentative Decisions

- **MCU:** STM32G071CBU3 (LCSC C724076).
- **IMU:** LIS3DHTR.
- **3.3 V supply:** TPS63802DLAR buck-boost converter, supplied from the charger SYS rail and set to the existing +3V3 system rail. Use a 0.47 µH inductor. AP2112K-3.3TRG1 is rejected and must not be used. The IC supports 1.3–5.5 V input and an adjustable output; confirm the full external component set against the TPS63802 datasheet before schematic sign-off. [TI product page](https://www.ti.com/product/TPS63802) · [TI datasheet](https://www.ti.com/lit/ds/symlink/tps63802.pdf)
- **Battery and charger:** BQ25185DLHR single-cell linear charger with power path, for a 1S 1000 mAh, 4.2 V Li-ion/Li-polymer battery. Connect the battery to BAT and use SYS as the buck-boost input. Magnetic charging remains the intended charging method; the source/interface presented to IN is TBD. Program the 4.2 V battery regulation setting; charge current and input current limit remain TBD. [TI product page](https://www.ti.com/product/BQ25185) · [TI datasheet](https://www.ti.com/lit/ds/symlink/bq25185.pdf)
- **Power button:** MAX16054, momentary button toggles power on each debounced press (first press on, next press off). Power-off is an immediate rail cut; no MCU shutdown sequence is required. Keep MAX16054 on the battery side and connect its output to TPS63802 EN so charging remains available while the 3.3 V system rail is off. [ADI product page](https://www.analog.com/en/products/max16054.html)
- **Measurement data retention:** do not save UWB measurement logs.
- **SPI Flash:** **W25Q64JVXGIQ** (64 Mbit / 8 MiB, XSON-8, 4 × 4 mm, 2.7–3.6 V; compatible with the 3.3 V rail). Use it for host-downloaded RF configuration, firmware-update staging, and LCD fonts/images. Recheck space after the LCD and firmware requirements are defined. [Winbond product selection guide](https://www.winbond.com/export/sites/winbond/product-selection-guide/file/2025-Product-Selection-Guide-Winbond-Code-Storage-Flash-Memory.pdf)
- **LCD:** ZJY114T-IF03, 1.14-inch 135 × 240 TFT, ST7789V3, 4-wire SPI, based on `docs/datesheet/ZJY114T-IF03.pdf`.
- **Host interface:** required; reserve the connector and MCU pins for communication with a host computer and delivery of RF configuration. Physical interface, connector, and protocol remain to be specified.
- **UWB module:** user has an existing module and will provide its specification later. Exclude its current from estimates until then.
- **UWB antenna:** tentatively an IPEX-connected flexible antenna; verify compatibility and placement after reviewing the module specification.

## Initial Operating-Current Estimate (UWB Excluded)

This is a component-level estimate, not a measured value or a completed-schematic/netlist result. It predates the selected BQ25185/TPS63802 power architecture and is obsolete as a design budget. Recalculate it using the selected power path before relying on the figures. It covers the MCU, LCD, LIS3DHTR, W25Q64, and MAX16054; it excludes the UWB module, charger, host-interface load, and unselected backlight-driver losses.

- At 64 MHz MCU clock, LCD awake with typical backlight, LIS3DH sampling at 50 Hz, and Flash in deep power-down between accesses: approximately **34.5 mA** typical.
- At 16 MHz under the same display conditions: approximately **30 mA** typical.
- With the LCD controller awake but backlight off: approximately **14.5 mA** at 64 MHz.
- With the LCD and backlight asleep but the MCU still running at 64 MHz: approximately **6.5 mA**.
- Flash reads can add up to approximately **20 mA** while active. High backlight current plus Flash access gives a provisional **55–67 mA** peak envelope; confirm against the selected backlight driver and firmware activity.

The 1000 mAh / 15-day target permits an ideal average battery current of at most **2.78 mA**, before capacity reserve, charger losses, and UWB consumption. Reaching the target therefore depends on MCU sleep, screen-off time, and duty cycle; the active-current figures above are not the expected long-term average.

## Remaining Decisions

1. Magnetic charging input implementation and electrical limits at BQ25185 IN; charge-current and input-current-limit settings; TS/NTC, status-pin, and battery-protection details.
2. Host interface electrical standard, connector, protocol, and MCU pin assignment.
3. PCB dimensions, mounting, enclosure, and connector placement.
4. UWB module specification, exact antenna model, supported band/channel, and RF placement constraints.
5. MCU clock frequency and complete MCU pin assignment.
6. Peripheral bus topology, including the SPI Flash bus.
7. Recalculate the current and runtime budget for the BQ25185/TPS63802 power path after charger settings, UWB module, and host-interface loads are known.

## Design Rules

1. Do not modify confirmed architecture without approval.
2. Before changing MCU pins, check STM32 alternate functions.
3. Do not assign UWB module interfaces or share their buses until its module specification has been reviewed.
4. Do not share LCD SPI.
5. Keep test points for important signals.
6. Confirm before selecting additional ICs.
7. Hardware design decisions require human approval.

## 原理图参考与绘图规范（2026-09-28）

### 后续工作顺序

1. 用户会先提供一份接近完整的原理图作为布局与绘图风格参考；收到前，后续优先讨论器件选型和参数，不主动继续补画或重排当前原理图。用户明确要求直接修改时，按最新指令执行。
2. 收到参考图后，先沿用其器件摆放、网络画法、标注方式和留白习惯讨论方案；器件与参数确认后再落图。
3. 未确认的器件型号、频率、负载等参数保留为 TBD，不把讨论中的暂定值写成最终设计。

### 标签、引线与电源符号

1. 单页同名网络使用普通本地标签；不放置全局标签，除非用户明确要求。
2. GND 网络至少放置一个 KiCad 标准 `power:GND` 电源符号。其余同页 GND 节点可通过普通本地 `GND` 标签连接；根据参考图风格添加更多 GND 符号。
3. 器件引脚的常规引线长度为 400 mil（10.16 mm）。标签名称较长时单独处理方向、旋转、位置和间距；确实需要延长时作为局部特例，仍须保证引线和文字清楚。
4. 标签文字不得与其他文字、器件、引脚或引线重叠。按参考图分组摆放器件并留出清楚间距，避免把器件和标签堆在一起。
5. 修改后用 KiCad 导出检查引线端点、标签方向和文字间距；仅有网表正确不能代替版面检查。

## Current Goal

Start KiCad 10 hardware design after the system plan is sufficiently defined.

First objective: complete the STM32G071CBU3 minimum system on a single schematic sheet.

MCU placement and library files have been created. The power architecture is now selected, but its schematic implementation and the MCU clock circuit remain pending. Do not treat this as a completed minimum system.

Do not start PCB layout before schematic review.

## 执行约束：本次错误的防复发要求

### 主次与授权

1. 先完成用户指定的当前主任务并验证结果，再处理下一件事。不得在关键问题未解决时插入清理、定时任务、额外研究或其他附带工作。
2. 用户限定“只处理”某项时，严格限制修改范围；一次性任务限制不固化为长期禁令，后续范围以用户最近的明确指令为准。
3. 文件修改须有用户确认；已经明确授权的范围直接执行，不反复索要同一确认。新增选型或未确认的硬件参数先给出具体方案和依据，再请求确认。
4. 简洁汇报当前结果、实际阻碍和下一步。不得长时间停留在无产出的分析，也不得用反复承诺代替操作。

### 器件导入与项目库

1. 每个导入器件必须交付完整的符号、封装和实际存在且可加载的 3D 模型。封装中的模型路径或空目录不代表模型已下载。
2. 下载前先检查已有导出文件、缓存和临时目录；可复用的文件直接复用。只有确认缺失或损坏的资源才重新下载。
3. 目录固定为 `Libs/symbols`、`Libs/footprints`、`Libs/3d_mods`。符号库和封装库只注册到项目根目录的 `sym-lib-table`、`fp-lib-table`，不擅自注册到全局库。
4. 项目库及 3D 模型使用 `${KIPRJMOD}` 相对路径，禁止引用个人临时目录。3D 模型需检查比例、位置、旋转和引脚 1 方向。
5. 按 KiCad 10 当前格式保存。优先使用已安装的 KiCad 自身升级和加载文件，不凭经验反复研究或手改格式后仅检查括号。
6. 导入引脚按完整型号及对应封装的数据手册核对；不得混用其他引脚数、封装或型号的引脚表。没有证据不得改动引脚编号，也不得把怀疑报告为已确认错误。

### 验证与完成标准

1. 查找软件时同时检查 PATH、安装注册表和实际安装目录；PATH 中没有命令不代表软件未安装。本机已确认 KiCad 安装目录为 `D:\ProgramFiles\KiCad`，已验证版本为 10.0.6；后续操作先确认路径仍有效。
2. 明确区分“文件已写入”“KiCad 命令行加载成功”“当前 GUI 已刷新并显示”。只报告实际完成的层级，不能互相替代。
3. 库交付至少验证：项目库引用可解析、符号和封装可由 KiCad 加载、3D 模型实际渲染可见。渲染命令成功后还必须查看图像，确认不是空焊盘图。
4. 外部改写文件后，当前 KiCad 窗口可能仍保留旧内容。先核对磁盘与窗口状态，再安全重新加载；不得用旧的空白窗口覆盖已有文件，不得擅自丢弃用户未保存的修改。
5. 用户报告界面错误时，以该错误仍未解决处理；命令行成功不能作为界面报错已修复的证明。权限受阻、未验证部分和剩余问题必须明确说明。
6. 原理图保持单页，不创建层次页。器件引脚与连线对齐连接网格。未接线的 MCU 不能宣称最小系统已完成。
7. MCU 供电按数据手册落实并核对电源脚、去耦与接地；晶振频率、型号、负载电容及相关参数未确认时，不擅自落为最终设计。
8. MCP 分别记录安装、配置、协议握手、实际工具调用及当前对话工具加载状态；服务端测试成功不代表当前对话已经可以调用。不能把文件型 MCP 描述成可自动刷新 KiCad GUI。

### 临时文件管理

1. 记录每次下载的来源、精确目录和已交付文件；将正式资源完整迁入项目库后，才考虑清理冗余临时产物。
2. 清理排在当前主任务完成之后，不能抢占修复缺失模型等关键工作的优先级。创建定时清理任务不能代替完成交付。
3. 清理前核对路径、引用关系和是否仍需备份，提供具体清单并遵守用户确认要求；禁止泛化删除系统 Temp、正式 Libs、KiCad 或 MCP 安装目录。
4. 本次已知临时导出位于 `C:\Users\Matsu\AppData\Local\Temp\UWB_Tag_C724076.*`、`C:\Users\Matsu\AppData\Local\Temp\UWB_Tag_C529343.*`；后续模型下载与验证产物位于项目 `_review`。这些是待检查范围，不是可直接删除的授权清单。
