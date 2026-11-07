# 我的日程（my-schedule）— 使用说明 & 开发文档

单文件个人日程管理器。无框架、无依赖、无构建步骤——整个应用就是一个 `index.html`，数据全部存在浏览器 localStorage，打开即用，离线可用。

- **在线地址**：https://withnohat1.github.io/my-schedule/
- **代码仓库**：https://github.com/withnohat1/my-schedule
- **技术栈**：原生 HTML + CSS + JavaScript（ES6+），PWA（manifest + Service Worker）
- **唯一源码**：`index.html`（约 3700 行，内联全部样式与脚本）

---

## 一、快速开始

### 本地运行
```bash
# 直接双击 index.html 用浏览器打开即可（file:// 协议下功能完整，仅 Service Worker 不注册）
# 或起一个静态服务器：
python3 -m http.server 8000
# 浏览器访问 http://localhost:8000
```

### 安装到手机（PWA）
手机 Chrome 打开线上地址 → 右上角菜单 →「添加到主屏幕」。安装后：
- 桌面出现应用图标（绿色圆环 +「日」字），全屏独立运行；
- **离线可打开**（Service Worker 缓存外壳）；
- 更新策略见「五、发布流程」。

### 数据说明（重要）
- 所有数据只存在**当前设备、当前浏览器**的 localStorage 里，**不上传、不跨设备同步**；
- 清除浏览器数据 = 清空所有记录，重要数据请定期自行导出（周报 PDF 是唯一的成品输出）；
- 换设备需重新录入。

---

## 二、功能与使用说明

### 三个视图（底部标签切换）
| 视图 | 内容 |
|---|---|
| **日** | 9 张每日任务卡（点击点亮/再点取消）、喝水 8 杯计数、支出记账（5 分类）、本日待办、每日西语一句 |
| **周** | 每天一列的完成热区、本周待办（按周存储）、6 项每周任务（点击点亮）、周报入口卡 |
| **月** | 工资/资产两个入口（每月一条记录）、月历热区（每格显示当天 9 任务点亮小方块）、右侧每周任务概览列、底层「No Risk / Full Push」水印 |

### 每日任务（9 项）
定义在 `DEFAULT_TASKS`（约 1076 行）。其中：
- **「学习新语言」「英语加强」「练习吉他」等 7 项**：点击即点亮（二元打卡）；
- **「状态」（原睡眠，改名为状态，同时记体重）**：点击弹记录层，填入睡/起床时间自动算时长，可记体重；
- **「运动」**：点击弹记录层，三类可选——有氧（时长）、功能（时长）、专项（部位：肩背/核心/臀腿 + 时长）；
- 点亮判定见 `isDone()`：有外部工具同步的日报则自动点亮（见「七、外部工具对接」），否则看本地打卡。

### 周报（A5 横版 PDF）
周视图底部「本周周报」卡进入。页面是一张按 A5 横版（210×148mm）排版的报告：完成率四格 + 每日完成率柱、睡眠时间轴、运动三类图表、支出分类圆环与明细。「存 PDF」按钮用 Canvas 按 300dpi 重画整页并生成 PDF 下载，**不走打印对话框**。

### 工资 / 资产（月视图顶部）
各只有**一个数字**输入框，每月一条记录，再次填写即覆盖；清空后保存 = 删除本月记录。工资必须 > 0，资产可为负（净资产口径）。输入框灰字提示上一次的数值（往回最多找 12 个月）。

### 理财复盘（周视图「理财复盘」任务）
点击进入，按周记录持仓（代码/名称/本金/收益），自动算收益率，周报里汇总展示。

---

## 三、架构与文件结构

```
my-schedule/
├── index.html            # 全部源码：CSS（~1100 行）+ HTML 骨架 + JS（~2600 行）
├── manifest.webmanifest  # PWA 清单（standalone、主题色 #F7F5F0）
├── sw.js                 # Service Worker：导航请求网络优先，静态资源缓存优先
├── icons/                # 180 / 192 / 512 三档应用图标
└── my_schedule_readme.md # 本文档
```

### index.html 内部分区（按行号，总 3720 行）
| 区域 | 行号范围 | 内容 |
|---|---|---|
| `<head>` + CSS | 1–908 | `<style>` 在 13–908：`:root` 主题变量（亮色）→ 暗色 `@media (prefers-color-scheme: dark)` → 各组件样式（顶部、宫格、面板、周视图、月视图、周报 `.rpt-*`、弹层 `.sh-*`、tabbar） |
| HTML 骨架 | 910–1068 | `.page-head`（标题/导航）+ 三个 `<section id="view-day/week/month">`（月视图含 `#wm` 水印）+ 弹层容器 + tabbar |
| JS：常量 | 1069–1199 | `DEFAULT_TASKS`、存储键 `KEY_*`、`EXP_CATS`、`EX_KINDS`、`WEEKLY_TASKS`、`REPORT_LINKS`、`ES_QUOTES` |
| JS：工具 | 1201–1300 | 日期工具（`iso/fromISO/addDays/startOfWeek`）、颜色工具（`hexRGB/shade/cdOf`）、load/save 系列 |
| JS：状态与数据操作 | 1281–1635 | `state` 对象、`setLog`、喝水、`isDone/toggle/doneCount/streak`、周任务、待办、支出、理财、月资金 |
| JS：渲染 | 1637–1930 | `render/paintHead/renderDay/renderWeek/renderMonth` 及各子渲染 |
| JS：周报 | 1934–2755 | `buildWeeklyReport`（DOM 版）→ `reportHTML` → Canvas 重画（`buildReportCanvas` 及 img* 系列图表）→ `makePDF` → `saveReportPDF` |
| JS：弹层 | 2835–3600 | 通用 `openSheet/closeSheet` + 睡眠/运动/记账/理财/工资资产各弹层 |
| JS：启动 | 3670–3720 | 步进导航、旧数据迁移 `migrateLegacy`、SW 注册（仅 https） |

---

## 四、数据模型（localStorage）

| 键 | 结构 | 说明 |
|---|---|---|
| `song.tasks.v1` | `[{id,name,icon,color}]` | 任务列表（可被用户改名改色，`loadTasks` 做迁移） |
| `song.records.v1` | `{ 'YYYY-MM-DD': { taskId: true } }` | 二元打卡记录 |
| `song.logs.v1` | `{ 'YYYY-MM-DD': { water:3, sleep:{bed,wake,weight}, exercise:{kind,minutes,parts} } }` | 数值明细 |
| `song.weekly.v1` | `{ 'YYYY-MM-DD'(周一): { taskId: true } }` | 每周任务完成 |
| `song.todos.v1` | `{ 'YYYY-MM-DD'(周一): [{id,text,done}] }` | 本周待办 |
| `song.daytodos.v1` | `{ 'YYYY-MM-DD': [{id,text,done}] }` | 本日待办 |
| `song.expenses.v1` | `{ 'YYYY-MM-DD': [{id,reason,amount,cat,at}] }` | 支出明细 |
| `song.finance.v1` | `{ '周一日期': [{id,code,name,principal,profit}] }` | 理财复盘 |
| `song.money.v1` | `{ 'YYYY-MM': { salary:Number, assets:Number, at } }` | 工资/资产，每月一条；兼容旧版 `assets` 为账户数组（读时求和）；两字段全空整条删除 |
| `song.reports.v1` | `{ 'YYYY-MM-DD': { kind, minutes, done, source } }` | 手动学习日报（已弃用入口，保留兼容） |
| `song.report.language.v1` / `song.report.english.v1` | `{ 'YYYY-MM-DD': { done, minutes, ... , at } }` | **外部工具写入**，应用只读（见第七节） |

> 旧数据迁移集中在 `migrateLegacy()` 和 `loadTasks()`：任务改名（睡眠→状态）、运动卡换深绿色 `var(--ex-done)`、支出旧分类 id 映射（`LEGACY_CAT`）等都在那里做，新增迁移逻辑也放这两处。

---

## 五、发布流程

线上托管在 GitHub Pages（main 分支根目录），改完代码需要重新发布：

### 方式 A：git push（有网络/代理时）
```bash
cd ~/WorkBuddy/my-schedule   # 本地发布目录（git 仓库，main 分支）
# …修改文件…
git add -A && git commit -m "…" && git push
```

### 方式 B：GitHub Contents API（git 协议被墙时）
仓库里 `git push` 可能被网络环境阻断，但 `gh api`（REST）通常可用。更新单文件：
```bash
python3 - <<'PY'
import base64, json, subprocess
REPO="withnohat1/my-schedule"
path="index.html"
content = base64.b64encode(open(path,"rb").read()).decode()
sha = json.loads(subprocess.run(["gh","api",f"repos/{REPO}/contents/{path}"],
                 capture_output=True,text=True).stdout)["sha"]
subprocess.run(["gh","api","-X","PUT",f"repos/{REPO}/contents/{path}","--input","-"],
    input=json.dumps({"message":"更新说明","content":content,"sha":sha}))
PY
```

### 缓存与更新生效（关键！）
`sw.js` 的缓存名是 `song-schedule-v2`：
- **页面导航请求走网络优先**——用户联网时每次打开都拿最新版，拿到后顺手更新缓存；
- **修改任何被 SW 预缓存的文件后，应把 `CACHE` 常量升一位**（如 `v2`→`v3`），旧缓存会在 `activate` 阶段自动删除；
- 历史教训：早期版本是纯缓存优先，导致服务器更新后手机端一直显示旧页面。不要改回去。

### Pages 构建
push 后等约 1 分钟，可用下面命令确认线上版本已更新：
```bash
curl -s "https://withnohat1.github.io/my-schedule/index.html?n=$(date +%s)" | grep -o "你要找的特征字符串"
```

---

## 六、主题与视觉系统

### CSS 变量（`<style>` 开头 `:root`）
- 基础色板：`--bg / --bg-2 / --surface / --ink / --ink-2 / --ink-3 / --line`；
- 强调绿：`--accent`（亮 #3F7A5C / 暗 #86C4A2）；
- **运动点亮色**：`--ex-done`（亮 `#2E5233` 深墨绿 / 暗 `#2E7D4F`）；
- **月视图水印墨色**：`--wm-ink`（亮 `rgba(38,80,58,.42)` / 暗 `rgba(134,196,162,.15)`）；
- 支出分类：`--cat-food/life/fun/shop/social`（饮食用绿点题，其余黑白灰米）。

### 点亮卡的"加深色"必须走 `cdOf()`
```js
const cdOf = c => /^#/.test(c) ? shade(c, -0.14) : `color-mix(in srgb, ${c}, #000 14%)`;
```
`shade()` 只认 hex 码；`var(--ex-done)` 这类变量色必须交给浏览器 `color-mix` 现算，否则会得到 `rgb(NaN,…)`（历史 bug）。卡片模板里写 `--c:${t.color};--cd:${cdOf(t.color)}`。

### 月视图水印（`#wm`）
- DOM：`#view-month` 的第一个子元素，两行 `<span>No Risk</span><span>Full Push</span>`；
- CSS：绝对定位 `inset:0; z-index:0`，`.mo-money/.mo-head/.mo-grid` 抬到 `z-index:1`；字体 Snell Roundhand（iOS/macOS 花体，退 Georgia italic），字号 `clamp(58px,17vw,118px)`；`pointer-events:none` 不挡交互；
- **水印能看见的前提是卡片半透明**：`.mo-cell` 背景 = `color-mix(in srgb, var(--surface) 52%, transparent)`；点亮小方块 `.mo-mini i.on` = `color-mix(in srgb, var(--mc) 55%, transparent)`（`--mc` 可能是 var 色也能算）。改回不透明背景水印就看不见了。

### 周视图标题
`paintHead()` 里周视图分支加 `.date-title.range`（22px、`white-space:nowrap`、tabular-nums），**只有日期一行**（如 `11.23—11.29`），无「本周」字样；副标题行 `headSub` 置空并隐藏。每次 `paintHead` 开头先重置 `t.className='date-title'` 防止视图切换残留样式类。

---

## 七、外部工具对接（学习日报自动点亮）

「学习新语言」「英语加强」支持外部工具写入数据后自动点亮，二选一：

**① 同域写 localStorage（推荐）**——外部工具页面与应用同域部署时自动同步：
```js
localStorage['song.report.language.v1'] = JSON.stringify({
  "2026-11-07": { done:true, minutes:25, items:"30 题", note:"Unit 3", at:Date.now() }
});
// 英语加强对应 key：song.report.english.v1
```

**② 直接调用全局 API**：
```js
ScheduleLink.pushReport('language', '2026-11-07', { minutes:25, done:true });
```

读取逻辑在 `reportOf(date, id)`：优先读外部 key（现读不落库），否则读 `song.reports.v1` 手动记录。卡片副行显示「N 分钟 · 已同步」。

---

## 八、周报 PDF 生成原理

这是全项目最复杂的部分，改版式前先读懂这一节：

1. **双份渲染**：屏幕上是 DOM 版（`reportHTML()` 生成 HTML + CSS），「存 PDF」时用 `buildReportCanvas()` 在 `<canvas>` 上**按同样的版式重画一遍**（300dpi，A5 横版 = 2480×1748px）→ `toDataURL('image/jpeg',.95)` → `makePDF()` 手写 PDF 二进制（MediaBox = mm×72/25.4，`/DCTDecode` 内嵌 JPEG，xref 偏移手工累计）→ `downloadBlob()`。不依赖任何第三方库，不走打印对话框。
2. **DOM 与 Canvas 共享同一组硬数字**（约 2339 行起）：`PADX=22, PADY=22, TILE_H=96, TILE_GAP=8, TOP_LW=380, TOP_GAP=24, SLEEP_DROP=58`、睡眠图 `H=272, L=48` 等。**改 CSS 版式必须同步改这些常量**（反之亦然），否则 PDF 与屏幕对不上。注意 `h1` 的实测行盒是 32px（写 28 也会因默认行高变 32），Canvas 用 `y+=32`。
3. 纸张物理尺寸：`.rpt-page` = 793.70×559.37px（96dpi 下 A5 横版），`fitReportPage()` 用 CSS `zoom` 等比缩放适配窗口；换纸张尺寸要同时改 `A5W/A5H` 常量、`.rpt-page` 尺寸、`buildReportCanvas` 的像素尺寸和 `makePDF` 的 mm 参数。
4. 运动柱高等用 **px 绝对值**（`(105-13)*min/max`），不要用百分比——会被数值标签挤变形（历史坑）。

---

## 九、常见修改任务指南

| 想改什么 | 去哪改 |
|---|---|
| 增删每日任务 | `DEFAULT_TASKS`（~1076 行）；新任务默认二元打卡，要弹层记录就加进 `DETAIL_TASKS`（当前 `{sleep:1, exercise:1}`） |
| 改任务颜色 | `DEFAULT_TASKS` 的 `color` 字段；hex 直接写，主题变量写 `var(--xxx)`（点亮深色自动走 `cdOf`） |
| 改水印文字/颜色/字号 | `#view-month` 里的 `#wm` 两个 `<span>`；颜色改 `--wm-ink`；字号在 `#wm span` 的 clamp |
| 改周视图标题样式 | `.date-title.range`（~111 行）与 `paintHead()` 周视图分支（~1660 行） |
| 改工资/资产规则 | `openMoneySheet()`（~2977 行）与 `setMoney/salaryOf/assetsOf` |
| 换纸张尺寸 | 见第八节第 3 条，四处同步改 |
| 换主题色 | `:root` 的 `--accent` 与相关 `--ex-*` 变量，暗色块同步 |
| 加新存储键 | `KEY_*` 常量 + `state` 字段 + `loadJSON/saveXxx` + `migrateLegacy` 兼容旧数据 |

## 十、调试与测试

项目无测试框架，验证靠无头 Chrome：
```bash
# 语法检查：抽出内联 <script> 后 node --check
# 行为测试：复制 index.html，在 <body> 前注入伪造 localStorage（Object.defineProperty 覆盖 window.localStorage）
#          再注入测试脚本，读 document.title 输出 JSON 结果：
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new \
  --no-sandbox --disable-gpu --user-data-dir=/tmp/cp --virtual-time-budget=5000 \
  --dump-dom file:///tmp/test.html | grep -o '<title>[^<]*</title>'
# 截图：--screenshot=/tmp/shot.png --window-size=430,900
```
注意：`--virtual-time-budget` 会把 CSS 过渡/动画冻在中间态，量几何前 `setTimeout` 800ms 以上；macOS 无头窗口宽最小 500px，测更窄视口用 iframe 包一层。

## 十一、已知坑（踩过的）

1. **SW 缓存**：导航请求务必保持网络优先，否则更新永远到不了用户手里（见第五节）。
2. **`shade()` 只认 hex**：新任务颜色若用 `var()`，点亮渐变必须走 `cdOf()`，别直接调 `shade`。
3. **`getComputedStyle(el).zoom` 返回字符串**：不能 `.toFixed()`，先 `String()` / `parseFloat`。
4. **睡眠时间轴跨天**：入睡 23:00、起床 07:00 时起床要 `+1440` 分钟再算，坐标函数 `axis = t => t < 720 ? t+1440 : t`。
5. **数据兼容**：改任何 localStorage 结构都要在 `migrateLegacy()`/`loadTasks()` 里兼容旧格式——用户的真实数据在手机上，升级后不能要求他清数据。

---

*文档版本：2026-11-07 · 对应 index.html 提交「月视图加 No Risk/Full Push 淡绿水印，日历卡片与点亮色块半透明化」之后的版本。*
