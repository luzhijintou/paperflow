# PaperFlow 更新日志

> 每次版本更新（新功能 / 修复 / 优化）都在**最上方**新增一条记录。
> 惯例：标题为「版本号 + 日期」；版本号与 `backend/api.py` 的 `version`、发布包 `PaperFlow-win64-vX.Y.Z.zip` 保持一致。未打包发布的修复轮次先以日期为标题，打包发布时补上版本号。

---

## 2026-09-10 · v0.8.0 补充：图谱节点配色修复 + 复习页重影修复 + README 截图换代

**背景**：为 README 重拍 v0.8.0 截图时，在干净的演示库上发现两处既有 UI 缺陷（与截图工作无关）。

**🔧 修复**
- **知识图谱节点全变灰**（`frontend/js/views/graph.js`）：节点染色选择器写的是 `node[color]`，而 cytoscape 的 `[color]` 是「存在该属性」匹配——`color` 为 `null` / 空串的节点同样命中，随后 `background-color: data(color)` 解析成 null、回退到 cytoscape 默认灰 `#999`，把文档（靛蓝）/ 标签（橙）的类型配色整层覆盖。改为真值匹配 `node[?color]` 后：普通节点恢复类型配色，设置过自定义颜色的节点仍按 `data(color)` 上色。实测（浏览器内读 cytoscape 计算样式）：文档节点 `rgb(129,140,248)`、标签节点 `rgb(245,158,11)`、自定义绿节点 `rgb(34,197,94)`，且 `[?color]` 只命中那 1 个有颜色的节点。
- **复习页说明行重影**（`frontend/js/views/review.js`）：点「显示答案」整卡重渲染，但清理逻辑只删 `.flashcard` / `.rate-row` / `.review-prog`，漏了「新卡 · 首次复习」那行——每揭一次答案就多叠一行。该行加上 `review-meta` 类并在重渲染时一并移除（修复后复拍确认只剩一行）。

**📸 README 截图换代**（`docs/screenshots/01-library` · `02-graph` · `03-review` · `04-reader`，1600×1000，已同步 GitHub）：在**独立演示库**（9 篇样例论文 + 多页报告 + 1 篇加密 PDF；含 3 组标签 / 3 条手工链接 / 3 张复习卡 / 1 条正文高亮批注）重拍，四张图分别突出——① 文档库列表视图（分类 / 状态含「需密码 1」/ 标签齐备）；② 知识图谱（文档 + 标签节点按类型配色、力学面板）；③ 复习页（展示答案后的四档评分与下次间隔预览）；④ 阅读器（新版工具栏「查找 / 进度 / 全屏 / 侧栏」+ 批注侧栏 + 正文高亮）。

**📦 重建与重传**：备份生产数据（v26，2487 文件 / 28 篇文档）→ 关闭正在运行的 PaperFlow 实例 → 重建冻结 exe（exit 0）→ 恢复数据 → 冻结冒烟通过（~1.0s 就绪、`docs=28`）→ 重打 `release\PaperFlow-win64-v0.8.0.zip`（2154 成员 / 99.7 MB；新增断言 `graph.js` 含 `node[?color]` 且不含旧 `node[color]`、`review.js` 含 `review-meta`；解压实跑 `/api/openapi.json` = `0.8.0` + 加密 PDF 端点齐备 + 空库 `documents=0`）→ `gh release upload v0.8.0 --clobber` **覆盖 GitHub 上的 Release 资产**，GitHub 端 digest `sha256:93dbaa42…` 与本地 zip 逐字节一致。新 exe sha256 `5c5a9291…`、zip sha256 `93dbaa42…`（`release\360-误报申诉说明.md` 已同步）。

**📝 README 定位放宽（中英）**：按用户要求把「文献」改为「文档」，不再把使用场景限定在学术——标题改为「本地优先的文档管理器 / A local-first document manager」；§1 新增明确段落「不局限于学术场景：论文、技术报告、产品手册、标准规范、教材、电子书，乃至轻小说 / 画集，只要是 PDF / EPUB，都能获得同一套检索、批注、关联与复习能力」；§2 改为「与常见工具的定位对比」、§5 改为「推荐的使用流程」；正文中「论文 / 文献 / 评审 / 学术」等措辞逐处替换（与 Zotero 的引文管理对比保留，仅作为"不做什么"的诚实说明）。GitHub 仓库简介同步改为「本地优先的文档管理器 · Local-first PDF/EPUB document manager…」。README 更新已上传 GitHub 并回读逐字节一致（包内 README 也是新版）。

---

## v0.8.0 · 2026-09-10 · 加密 PDF 支持（导入不再失败 → 阅读器输密码 → 记住密码自动建全文索引）

> 版本号从 `0.6.0` 直接提到 **`0.8.0`**（`0.7.0` 未发布）：本版把 v0.6.0 之后的两轮补充（浏览器缓存目录迁出 `data\` + WebView2 磁盘缓存参数）与加密 PDF 支持一并纳入，发布包 `release\PaperFlow-win64-v0.8.0.zip`（顶层目录 `PaperFlow-win64-v0.8.0/`）。

**背景**：导入 `D:\qq下载\各种13（4）.pdf` 时事件日志出现 `document.import_failed · FileNotDecryptedError: File has not been decrypted`。核实为**文件本身带打开密码**（pypdf：`/Encrypt` V4/R4 / 128-bit RC4，空密码解密返回 `NOT_DECRYPTED`；PDFium 独立复核同样报 `Incorrect password error`），不是程序缺陷——旧行为是直接拒收、文件进不了库。本轮把「有密码的 PDF」变成一等公民。

**🔧 改动**
- **导入不再失败**（`backend/textproc.py`、`backend/ingest.py`）：新增 `PdfPasswordRequired` 异常；加密文件照常入库（文件 + 卡片 + 元数据，状态 `needs_password`），不再写 `import_failed`。**仍然先试空密码**，所以「只设了权限标志」的出版方 PDF 一如既往直接导入。
- **阅读器输密码即读**（`frontend/js/views/reader.js`）：打开加密文档弹密码框（含「记住此密码」勾选），密码交给 pdf.js 解密；密码错显示「密码错误，请重试」并重弹，取消则退出加载。**踩坑**：本版 pdf.js 从 loading task 对象读 `onPassword`，作为 `getDocument` 参数传入会被忽略（表现为直接报 `No password given`），必须写 `task.onPassword = ...`。
- **解锁后自动建索引**（新增 `POST /api/documents/{id}/unlock` + `ingest.unlock_document()`）：密码**经 pdf.js 校验通过后才回填后端**（错密码不会发出去，也不会多冒一条错误提示）→ 抽文本 → 重建 FTS / 向量 / MinHash → 状态转 `ready`（扫描件转 `needs_ocr`）→ 顺带补作者元数据。勾选「记住密码」时密码存入新表 `doc_secrets`（**仅本机数据库**，只在单文档接口下发给阅读器，列表接口不带），下次打开自动解锁、无需再输。
- **忘记密码**（`DELETE /api/documents/{id}/password` + 阅读器信息栏「忘记密码」）：删掉保存的密码，文件照常可读、已建索引保留。
- **文档库状态**：新增 `需密码` 状态（筛选器 + 卡片徽章 `.badge.needs_password`）；导入结果提示区分「新增 / 重复 / 需密码 / 失败」，含加密文件时额外提示勾选「记住密码」。
- **OCR 守卫**（`backend/api.py`、`backend/ocr.py`）：OCR 是服务端渲染，需要密码；未记住密码的加密文档在单篇 / 批量 OCR 时被明确拦下并提示先去阅读器输密码。三个渲染器（pypdfium2 / PyMuPDF / pdf2image）都支持传入密码。
- **已知限制**：AES 加密（AESV2 及 V5/R5/R6）需要 `cryptography` 依赖，当前 `deps\` 未内置且本机无网络补装——这类文件**仍可在阅读器中输密码正常阅读**，但后端无法抽文本 / OCR，会明确提示「此 PDF 使用了当前版本不支持的加密方式」；有网络后把依赖放进 `deps\` 即可覆盖。

**✅ 验证**：`tools/test_backend.py` **107 passed / 0 failed**（新增 17e 段 13 项：加密文件导入 → `locked`、初始无密码、错密码 → 403、空密码 → 400、正确密码 → pages/has_text/状态、记住的密码只从单文档接口下发、解锁后 `/api/search` 命中解密正文、重复导入仍去重、忘记密码后不再下发且索引保留、测试文档清理）。`check_i18n.py` 覆盖 503 键 / **0 缺英译 / 0 后端漂移**（新增 21 条前端词条 + 6 条后端运行时文案）；`check_imports.py` 干净；`node --check` 全部改动模块通过。**Playwright 无头实测 13/13**：库卡片显示「需密码」→ 打开弹密码框 → 错密码重弹并提示（且不产生多余 toast）→ 正确密码后 PDF 正常渲染 + 「已解锁，文本索引已建立」→ 后端 `has_text=1`、`/api/search` 命中解密后的正文 → 信息栏显示「已记住（仅保存在本机）」→ **关掉重开无需再输密码**。

**📦 版本号与发布包**：`backend/api.py` 的 `version` 由 `0.6.0` 提到 **`0.8.0`**（`/api/openapi.json`、`/api/docs`、exe 版本资源随之）；中英 README 升级为**合并版**——以 GitHub 上的学术风结构（徽章 + 目录 + 截图）为骨架，并入原长文档独有的「REST API（节选）」「开发与测试」「路线图」「桌面窗口与自行打包」「数据目录与备份」等章节，并把版本标注更新到 0.8.0（旧版 README 存档于 `backup/src-20260910-readme-v1/`，截图已拉回本地 `docs/screenshots/`）。流程：备份生产数据（v25，2482 文件 / 1.1 GB，WAL 已 checkpoint）→ 重建冻结 exe（exit 0）→ 恢复数据（文件数与备份一致）→ 冻结冒烟通过（~1.0s 就绪、`docs=23`）→ 重打 `release\PaperFlow-win64-v0.8.0.zip`（2154 成员 / 99.7 MB、无 `data\` 泄漏、包内 exe sha256 与 `dist` 一致、前端 `task.onPassword` / 「记住此密码」/ `unlockDocument` / 「需密码」徽章等标记齐全）→ **解压缩实跑**：`/api/openapi.json` = `0.8.0` 且含 `POST /api/documents/{id}/unlock` 与 `DELETE /api/documents/{id}/password`（证明后端代码确实进包）、空库 `documents=0`；exe 版本资源 8 个字段均为 `0.8.0`。包内 exe sha256 `7b2f9e9e…`、zip sha256 `625ea780…`（该轮产物后经同日补充轮重建，**最终哈希以顶部条目为准**：exe `5c5a9291…` / zip `93dbaa42…`）；`release\360-误报申诉说明.md` 的版本号 / 包名 / 两个 SHA256 已同步为新构建。

**🚀 GitHub 发布**：代理（UniClash 7993）恢复后，用 `gh release create v0.8.0 --target main` 一条命令建 tag + 上传资产（99.7 MB）——GitHub 端资产 digest `sha256:625ea780…` 与本地 zip 逐字节一致，**v0.8.0 已是仓库 Latest**（此前只有 v0.1.0，v0.2~v0.6 的包因代理不通从未上传）。远端 main 同步更新：README.md / README_EN.md 上传为上述合并版（Contents API，回读校验逐字节一致），并**新增 CHANGELOG.md 到仓库**（README 的更新日志链接指向它）。发现并解决一处长期漂移：远端 README 是学术风 v2、本地仓库 README 是旧长文版，两份不同源——本次合并后**本地与远端统一为同一份**。

---

## 2026-09-10 · v0.6.0 补充（防杀软误报）：浏览器缓存目录迁出 `data\` + WebView2 落盘收敛

**背景**：当日下午 360 安全大脑以「发现大量可疑文件同步/上传操作，如果不是您主动操作，可能遭遇勒索攻击」弹窗拦截 `PaperFlow.exe`（勒索防护启发式，误报；库内 23 篇文档始终完好）。根因：内嵌窗口（WebView2）的配置目录 `data\.webview2-profile` 每次启动都写入上千个小文件、累计 626 MB，而它就在**用户文献库目录里面**——「程序在文档目录里批量写文件 + 起本地 HTTP 服务 + 扫描全库」正是勒索软件的特征组合。本轮把「程序的缓存写入」与「用户的文档目录」彻底分开，并压低单次落盘量。发布包 `release\PaperFlow-win64-v0.6.0.zip` 已按新构建重打。

**🔧 改动**
- **浏览器配置迁出库目录**（`run_desktop.py` 新增 `_state_dir()`）：WebView2 配置目录、Edge app 模式配置、启动页 HTML 从 `data\`（`.webview2-profile` / `.app-window-profile` / `.window-loading.html`）迁到 `%LOCALAPPDATA%\PaperFlow\`（不可写时回退系统临时目录），库目录里只剩用户数据。本机升级时已把旧 `data\.webview2-profile` 迁到新位置以保留主题 / 视图等界面偏好；旧的 `.app-window-profile` / `.window-loading.html` 可直接删除（已自动改用新位置）。实测：一次启动在库目录落盘的文件数 **124 → 2**（只剩 `paperflow.log` 与 `paperflow.db`），全部缓存写入改落在 `%LOCALAPPDATA%\PaperFlow\`。
- **WebView2 磁盘缓存压到 1 MB**（`run_desktop.py` 新增 `_limit_webview2_disk_cache()`）：给浏览器进程加 `--disk-cache-size=1048576`（默认 50 MB）。踩到的坑：pywebview 6.2.1 在 `EdgeChrome.__init__` 里**硬编码** `AdditionalBrowserArguments`，所以 ① `WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS` 环境变量被忽略（应用显式设值优先），② 该 `__init__` 末尾的 `EnsureCoreWebView2Async` 会当场消费这份 props，构造完再改也来不及。最终做法是把 `CoreWebView2CreationProperties` 换成子类、在 `__setattr__` 里追加参数（对 pywebview 内部改动只降级不报错）。
- **README（中/英）**：数据目录说明补一句「界面缓存位于 `%LOCALAPPDATA%\PaperFlow\`，不在 `data/` 内」。
- 另两条防误报途径**不在本轮代码内**，按需再取：**代码签名证书**（OV/EV，根治误报；自签名对 360 无效）与**厂商申诉**（360 误报反馈，按文件 hash 处理，每次重新构建 hash 都会变）。弹窗本身直接选「忽略/信任」即可，**不要点「立即阻止」**。

**✅ 验证**：`tools/test_desktop_logic.py` 新增第 7 项断言（拦截只在首次赋值追加参数、不重复叠加、其他属性透传）并通过；`tools/check_imports.py` 干净；`tools/test_frozen_exe.py` 对重建后的冻结 exe 通过（~1.0s 就绪、`docs=23`、前端 2239 B / `style.css` 44859 B）；**隔离数据目录实跑冻结 exe**：25 秒内库目录只生成 `logs/paperflow.log` 与 `paperflow.db`、无任何浏览器缓存产物，且浏览器进程命令行确认带 `--disk-cache-size=1048576` 与 `--user-data-dir=%LOCALAPPDATA%\PaperFlow\webview2\EBWebView`。发布包按新构建重打：2154 个成员 / 99.7 MB、无 `data\` 泄漏、包内 exe sha256 `d18e5b50…e5393fe` 与 `dist` 一致；解压实跑 `/api/openapi.json` = `0.6.0`、`/api/health` `documents=0`（纯净空库），exe 版本资源 0.6.0 完好。

---

## 2026-09-10 · v0.6.0：纯净发布打包（版本号落地 + 中英 README / 更新日志收口）

**本轮是打包发布，不改功能代码**（唯一代码改动是构建脚本 `build_exe.py` 注入 exe 版本资源，见下）：发布内容 = 下列两条条目（0.61 导出正确性与阅读器六项交互修复、0.60 目录识别 / 分类右键删除 / 页内查找与快捷键），其中已含 0.61 条目补记的 **B6**（双页 + 横向翻页下 `↓` 死按钮）修复——即 2026-09-10 上午那次重建的成果，本次一并打成正式包。

**📦 发布**
- `backend/api.py` 的 `version` 由 `0.4.1` → **`0.6.0`**（`/api/openapi.json`、`/api/docs` 随之）；发布包 `release\PaperFlow-win64-v0.6.0.zip`，顶层目录 `PaperFlow-win64-v0.6.0/`。
- 纯净包 = `dist\PaperFlow` 整目录**剔除 `data\`** 后压缩，内附 `README.md` + `README_EN.md`；**不含**真实文档库、缩略图缓存、`data\pylibs`（OCR 引擎由用户在设置页首次使用时一键安装）。沿用 `release\PaperFlow-win64-v0.4.0.zip` 的目录结构。
- **exe 补上 Windows 版本资源信息**（`build_exe.py` 新增 `app_version()` / `write_version_file()`）：经 `--version-file` 注入 CompanyName / ProductName / FileDescription / FileVersion / ProductVersion / LegalCopyright / OriginalFilename / InternalName，版本号**构建时从 `backend/api.py` 的 `version` 正则提取**（单一事实来源，不会漂移）；临时 VSVersionInfo 写在系统 temp，不放 `build\`（`--clean` 会在启动时清掉该目录）。动机：**无版本信息、无签名的 PyInstaller 程序是杀软启发式误报的高发特征**，补全元数据可降低误报率（根治仍需代码签名证书）。资源管理器「属性 → 详细信息」现已可见上述字段。
- 打包前把 `dist\PaperFlow\data`（23 篇 / 1.673 GB）镜像备份到 `backup\dist-data-20260910-v20`，`build_exe.py` 重建后原样回灌（1146 目录 / 4281 文件，0 失败）；补版本资源后又按同流程重建一次（备份 `v21`，同样 1146/4281、0 失败回灌），**发布包以第二次构建（含版本资源）为准**，zip 已覆盖重打。

**✅ 验证**：`tools/test_frozen_exe.py` 对打包所用构建通过（健康就绪 ~1.0s、`docs=23`、前端 2239 B 与 `style.css` 44859 B 正常返回）；另跑一次性探针确认**冻结 exe 的 `/api/openapi.json` 版本号 = 源码 `backend/api.py` 的 `0.6.0`**，且包内 `_internal\frontend\js\views\reader.js` 含 B6 新写法、不含旧写法；至此打包内容与源码、文档三者一致。补版本资源后对**重打的发布包**再做一轮端到端检查：包内 `PaperFlow.exe` 的 sha256 与 `dist\PaperFlow\PaperFlow.exe` 一致（`a2639ef3…6d80c051`）、共 2154 个成员、无 `data\` / 库文件泄漏、`README.md` + `README_EN.md` 在内；**解压到临时目录实际运行**：资源管理器可见全部 8 个版本字段（SignatureStatus=NotSigned，符合预期），`/api/openapi.json` = `0.6.0`、`docs=0`（纯净库，未带任何真实数据）、前端正常返回。

---

## 2026-09-09 · 0.61：导出正确性修复（Anki 静默丢卡 / Obsidian 同名覆盖）+ 阅读器交互修复（向下翻页·四条路径 / 工具栏换行 / 查找不显眼 / 右键选不中 / 全屏看不见浮层）

这一轮全部来自「用了才发现」的缺陷：导出侧三处会造成**数据丢失且毫无提示**的问题，阅读器侧六处 0.60 及之前遗留的交互问题。均为修复，无新增依赖。

**🐛 修复 · 导出**
- **A1 Anki 推送静默丢卡**（`exports.py` `export_anki`，`backend/db.py` 默认设置，`settings.js` 集成面板）：旧实现把笔记类型写死成 `Basic` + 字段 `Front`/`Back`，而 `addNote` 的失败是以**响应体里的 `error`**（如 "note type 'Basic' does not exist"）返回、并非 HTTP 错误，代码又用 `except Exception: pass` 吞掉 → 用户看到「已推送 N 张」，Anki 里**一张都没有**。现改为：牌组名之外新增**笔记类型 / 正面字段 / 背面字段**三项可配置（设置 → 集成 → Anki 集成，默认仍 `Basic`/`Front`/`Back`，随 `anki` 设置持久化），推送前先调 `modelFieldNames` 校验字段是否存在、缺失即**直接报错并列出该类型实际字段**，`addNote` 的 error 拆成「重复跳过」与「真实失败」两类计数，失败样例随响应返回并在 toast 中显式提示（`{n} 张失败：{e}`）。旧库残留的 `anki` 设置缺这些键时由 `or` 兜底，不需迁移。
- **A2 单篇 Anki 推送直接 500**（同函数）：按文档导出分支拼 SQL 时把 `(doc_id,)` 元组拼到了字符串位置（`q + (" WHERE …", (doc_id,))`）→ `TypeError`，阅读器侧栏的「导出 Anki」对任意文档都必然失败。改为两条显式 `c.execute()` 语句。
- **A3 Obsidian 同名文档互相覆盖**（`exports.py` `export_obsidian` / 新增 `_different_doc`）：文件名仅由标题派生，冲突时旧判据只看「路径存在」→ 标题规范化后同名的**两篇不同文档**第二篇会直接覆盖第一篇的笔记（丢一整篇）。现在每篇 Markdown 的 frontmatter 写入 `paperflow_id`，冲突时读回已有笔记头部比对身份标记：**同文档 → 原地覆盖（重复导出仍幂等更新）**，**异文档或人工手写笔记 → 追加 `_2`/`_3` 后缀另存**，不再误伤他人内容。

**🐛 修复 · 阅读器**
- **B1 向下翻页按钮点了没反应**（`reader.js` `currentPage`/`gotoPage`，新增 `docTop`）：滚动模式下页码定位用 `offsetTop`，而 pdf.js 页面元素的 `offsetParent` 是 `SECTION.view`，`offsetTop` 因此**以视口为原点**（实测恒比文档坐标大一个工具栏高度 59px）；`scrollTo` 却按 `scrollTop` 的文档坐标计算 → 目标算小，滚动条实际没动，`↑` 因为方向恰好"够用"而显得正常、`↓` 完全无反应。统一改用 `getBoundingClientRect()` 差值 + `scrollTop` 换算（`docTop`），`currentPage` 与 `gotoPage` 同源于同一坐标系，上下翻页、目录跳转、进度条定位一并校正。
- **B2 全屏时工具栏挤成两行**（`reader.js` `buildToolbar`，`style.css`）：工具栏原本允许换行，全屏窄宽度下「全屏 / 收起侧栏」被挤到第二行、排版塌陷。改为**强制单行 + 横向可滚动**（`flex-wrap: nowrap` + `overflow-x: auto`），并把「页内查找 🔍」提到原先右侧按钮的位置，**「进度百分比 / 全屏 / 收起展开侧栏」整体移到最右**（新增 `.tb-right` 簇 + `margin-left: auto`）；中间的操作提示文字改为**弹性省略号收缩**（`.tool-hint`），窗口再窄也不会撑破布局。
- **B3 页内查找命中不显眼**（`reader.js` `paintFindHighlight`/`collectFindRanges`，`style.css`）：pdf.js 文本层的 span 本身是 `color: transparent`，而 `::highlight()` 若不指定 `color`，命中处只有一层半透明底色、文字仍"看不见"。改为**命中统一绘制不透明黄底 + 深色字**、**当前项橙色底 + 白字 + 下划线描边**，并把高亮范围从「仅当前页」扩展到**所有已渲染页同时上色**；`‹ ›` 跳转时当前命中**自动滚动到视口上三分之一**（原为贴边或被遮住）。
- **B4 非全屏时右键无法选中文字 / 右键不弹出选择工具条**（`reader.js` `setupSelection` + 新增 `onContextMenu`/`selectWordAt`，`onGlobalMouseUp` 排除右键）：Chromium 的原生右键菜单会**吃掉本次右键拖出的选区**，且右键从未被当作一次「选择」提交，于是「选中即浮出工具条」在右键路径下完全不触发。现仅在命中 PDF 文本层 span 时接管右键：先用 `caretRangeFromPoint` + `Selection.modify(..., "word")`（ICU 分词，中文按词）**自动选中光标处的词**，若已有选区包含该 span 则原样保留，随后走既有的选区处理逻辑浮出工具条；页面图区之外的右键**仍走原生菜单**，不牺牲浏览器/系统能力。
- **B5 全屏下浮层全部不可见**（`util.js` 新增 `mountOverlayRoot`，`reader.js` 工具条挂载点）：按规范，全屏时**只有全屏元素子树参与渲染**，挂在 `document.body` 上的 `#modal-root` / `#toast-root` 与选区工具条会整体消失（表现为「点了没反应」）。现监听 `fullscreenchange` 把两个浮层根节点**动态重新挂载**到 `document.fullscreenElement`，选区工具条改挂进阅读器根节点；工具条的可用横向边界也改为按**该根节点的矩形**（而非视口）钳制，避免全屏下溢出屏幕外。
- **B6 双页 + 横向翻页下 `↓` 仍是死按钮**（`reader.js` `stepPage`）：B1 修的是坐标系，这一处是**另一条独立的根因**。「上一页/下一页」按钮走 `stepPage`，其中「双页按跨页步进」的分支原本带 `!R.flip` 条件（意图是「横向模式的跨页步进已由 `flipStep` 处理」），但该分支同时决定了 `gotoPage` 拿到的**目标页号**；关掉条件后 `flip` 走的是 `gotoPage(cur + 1)` → `flipTo(cur + 1)`，而 `flipTo` 开头会把目标折叠回所属跨页起点 `spreadStart(n)`——`[4,5]` 跨页的 `4 + 1 = 5` 折回**还是 4**，紧接着的 `if (n === R.curPage) return` 直接返回 → **`↓` 在该组合下 100% 无反应**（键盘方向键走 `flipStep`，路径正确，所以只有工具栏按钮坏）。现改为**双页一律按跨页步进**（`gotoPage(nextSpreadPage(spreadStart(cur), d))`），去掉 `!R.flip`；`↑` 此前只是 `5 - 1 = 4` 折算术上碰巧对才没暴露。**教训**：B1 的验证只覆盖了开发库的 `page_mode=vertical`，而真实使用是 `page_mode=horizontal`——同一功能的四条路径（竖向单页 / 竖向双页 / 横向单页 / 横向双页）必须逐个跑，缺一即漏。

**验证**：`tools/test_backend.py` **94 passed / 0 failed**（新增 14b 段 10 项，以本地 `http.server` 桩模拟 AnkiConnect：逐文档推送不再 500 且 `created==total`、配置化模型/字段生效（`Q&A` + `Q`/`A`）、字段名错误→502 并带出 AnkiConnect 的真实报错、二次推送→`created 0 / skipped 2` 不重复入库；同名两篇导出后**两篇笔记并存且内容各自正确**、重名身份标记 `paperflow_id` 存在、重复导出原地更新自己的笔记）。`tools/test_epub.py` **55 passed / 0 failed**；`check_i18n.py` 覆盖 480 键 / **0 缺英译 / 0 后端漂移**，`check_imports.py` 干净，`node --check` 四改动模块通过。浏览器 DOM 级实测：工具栏单行且顺序为「…🔍 · 进度 · 全屏 · 侧栏」、`↓` 翻页的滚动目标与 `docTop` 一致（此前恒偏 59px）、右键选中→工具条出现链路成立、查找高亮 Range 集合与居中滚动成立。B6 的回归按**四条路径逐个跑**：横向双页修复前 `↓` 点 5 次页码输入框纹丝不动（`[4,5]` 原地）、修复后轨迹 `[4,5] →↓ [6] →↑ [4,5] →↑ [2,3] →↑ [1]` 且末尾 `↓` 正确停在末跨页；横向单页 `5 →↓ 6 →↑ 5`；竖向单页 `scrollTop 0 → 365`；竖向双页以**只记录不替换**的 `scrollTo` 探针取到目标 `196 / 12`，与各页文档坐标（页 2/3 起点 204、页 1 起点 20）吻合。**（真实窗口的全屏观感、文本层描边、平滑滚动手感与右键习惯仍需人工确认——自动化窗口处于隐藏态，Chromium 不会推进 `scroll-behavior: smooth` 的滚动动画，故竖向模式只能验证"目标算对了"、无法验证"滚到位"；这不是产品缺陷。）** （v0.6.0 发布时已处理：`version` 提至 `0.6.0`、纯净包 `release\PaperFlow-win64-v0.6.0.zip` 已打出，见顶部 2026-09-10 条目；GitHub 上传仍待代理可用。）

---

## 2026-09-09 · 0.60：PDF 目录识别修复 + 分类右键删除 + 页内查找与阅读快捷键

延续 T1 的阅读体验层，这一轮先修掉 T1 遗留的一处「点了没反应」的目录 bug，再补齐两项高频交互：侧边栏分类的右键删除、以及 PDF 页内查找（Ctrl+F）与一套贯穿滚动/翻页/全屏的阅读快捷键。

**🐛 修复**
- **C1 PDF 目录点击不跳转**（`reader.js` `sideOutline` 内 `resolvePage`）：T1 引入的书签解析里，缓存 key 误按数组写 `d[0].num.join(",")`，而 pdf.js 的 `Ref.num` 是**数字**不是数组 → 每条目都抛错被 `try/catch` 吞成 `null`，于是 52 条书签全被判成「无目标页」而点击无反应。改为数字/数组兼容取 key 后，实测 52/52 正常解析并跳转。同时清洗标题里的 NUL/控制符/零宽字符，无页码条目标 `navless` 置灰、空标题回退「第 n 页」，并新增 `syncOutlineActive` 让目录随当前页高亮、点目录即时点亮（EPUB `renderEpubToc` 同步对齐）。

**✨ 新增**
- **侧边栏分类右键删除**（`library.js` `collectionItem` + 新增 `collectionMenu`）：分类条目支持右键弹出菜单（打开/取消筛选 + 删除），并保留原 `×` 快捷删除。删除判据是**结构化的** `deletable = c.custom && !c.n`（用户自建 且 空分类），**绝不按名字匹配**——顶部的合成「全部文档」项从不带 `custom` 标记、天然不可删；而用户把自己分类命名成「全部文档」的，只要它是空的照样可删，规避了「按名字判断导致同名误伤」的坑。非空分类走后端既有保护（`api.py` 拒删带文档的分类），前端显示「该分类下还有 N 篇文档，请先移出后再删除」。
- **PDF 页内查找 Ctrl+F**（`reader.js` 新增 `openFind/closeFind/findEnsureIndex/findCompute/findGo/findStep/paintFindHighlight` 模块 + 工具栏 `🔍` 按钮）：`findEnsureIndex` 用 pdf.js `getTextContent` 分块（每帧 8 页）异步索引全文，命中数用 **CSS Custom Highlight API**（`::highlight(find-all/find-cur)`）在文本层上色，**不侵入 pdf.js 文本层 DOM**（选区/批注不受干扰）；`‹ ›` 或 Enter/Shift+Enter 在结果间跳转，状态栏显示「第 i/N · 已索引 x/y」，EPUB 暂回退提示用顶部搜索。索引分块的让路由 `requestAnimationFrame` 改为 `setTimeout(…,0)`，避免窗口失焦/后台时 rAF 被彻底暂停导致索引卡在 0/N。

**🔧 优化**
- **贯穿式阅读快捷键**（`reader.js` 全局 `keyHandler`）：原先仅 flip 模式生效的键盘处理升级为全局——`Ctrl/Cmd+F` 查找、`Esc` 关查找、`→/PgDn/空格` 与 `←/PgUp` 翻屏或翻页（flip 走 `flipStep/epubFlipStep`、滚动走 `stepPage`）、`Home/End` 首末页、`+ - 0` 缩放（PDF）；带 `typing` 守卫，在输入框/文本域/可编辑元素内不劫持按键。`teardown` 补 `clearFindHighlight` 防离开时高亮残留。

**验证**：`tools/test_backend.py` **84 passed / 0 failed**（含分类删除的自建/重名/非空/未知四路断言）；`tools/test_epub.py` **55 passed / 0 failed**；`check_i18n.py` 0 缺英译、0 后端漂移。浏览器实测：目录 52/52 解析跳转 + 联动高亮成立；分类右键删除四类用例（含「用户自建且恰名为全部文档」可删、合成「全部文档」不可删）全部正确；页内查找在 6 页样例上 `page→342 / report→6 / section→6`、无结果项显示「无结果」、`Ctrl+F/Cmd+F` 开、`Esc` 关成立。**（页内查找的高亮描边与目录/查找的滚动定位依赖文本层实际绘制，自动化窗口处于隐藏态文本层不渲染，未在该环境目视确认，待真实窗口人工核验。）** （发布已落地：2026-09-10 顶部条目把 `version` 从 `0.4.1` 提至 `0.6.0` 并打出 `release\PaperFlow-win64-v0.6.0.zip`；GitHub 上传仍待代理可用。本仓库非 git，源码回退点 `backup\src-20260909-v1-pre-reader-opt`）

---

## 2026-09-09 · 阅读器 T1 优化：目录侧栏 + 封面秒开 + 长文页虚拟化

对标 Thorium「章节导航」、Aquile「现代 UI」、Edge「打开即用」的体验层，聚焦"看得见、翻得动、滚得久"。延续 T0，仍不改任何第三方依赖。

**✨ 新增**
- **C1 PDF 目录 / 书签侧栏**（`reader.js` `sideOutline`）：右侧栏「目录」tab 用 `getOutline()` 递归渲染 PDF 内嵌书签树，点击 `getDestination/getPageIndex` 解析目标页并跳转，缩进按层级、当前页高亮；结果按 ref/命名目标缓存进 `R._outlineIdx` 免重复解析。无书签时给友好空态。
- **D2 EPUB 章节目录侧栏**（`reader.js` `renderEpubToc`）：同一「目录」tab 复用，EPUB 下把 `R.chapters`（spine + nav/ncx 标题）渲染成可点击章节列表，点击翻到该章、高亮当前章；无标题回退「第 n 章」。
- **E1 封面缩略图服务端持久化**（`backend/api.py` 新增 `GET/PUT /api/documents/{id}/thumb`，`cover.js`）：内容寻址（`?v=<sha>` → `data/thumbs/{id}.{sha16}.jpg`，`immutable` 缓存）。库卡片首开时 pdf.js（PDF）/内嵌封面（EPUB）渲染一次，回传 `PUT` 落盘；此后每次重开，`cover.js` 先用 `probeThumb` 命中这张 ~20KB 小图即时上色，**彻底跳过整份文档解析**。无新依赖（不引入 pypdfium2 等原生渲染器，复用浏览器既有 pdf.js）。

**⚡ 性能**
- **B1 阅读器页面虚拟化（离屏 canvas 回收）**（`reader.js` `layoutPages` 观察者 + 新增 `releasePage`）：垂直滚动模式下，IntersectionObserver 除进入 ±600px 缓冲带渲染外，**离开缓冲带的页回收 canvas 位图与文本/批注 DOM**（置 `p.rendered=false`、canvas 归零、清空图层），回到缓冲带时 `renderPage` 从内存中的 `R.pdfDoc`/`R.annotations` 无损重建。占位 `.pdf-page-wrap` 的宽高与 `--scale-factor` 始终保留 → 滚动条长度、各页 `offsetTop`、进度定位都不抖动。任意时刻内存里只驻留视口附近的少数几页，**百页长文不再随滚动无限堆叠 canvas**。flip（横向翻页）本就只渲染当前页，跳过回收。

**验证**：后端新增 11 项断言（A2 immutable+accept-ranges、下载仍 attachment+no-store；E1 缺失 404 → PUT 落盘 → GET immutable jpeg → 旧版本 404 → 非 JPEG/未知文档被拒），`tools/test_backend.py` **84 passed / 0 failed**；`tools/test_epub.py` 顺带修正两处 T0 后失效的 `no-store` 旧断言为 immutable+Range，**55 passed / 0 failed**；`check_i18n.py` 465 键 / 0 缺英译。浏览器实测：库首开两封面上色→`data/thumbs` 落盘两张内容寻址文件，**硬刷新后两张即时从 `/api/.../thumb` 上色（`api:2, data:0`），pdf.js 全程未触发**，E1 秒开闭环成立。**（B1 离屏回收受自动化浏览器视口被压到 ~30px、且未渲染 canvas 默认宽 300 干扰判读，未在该环境目视确认；渲染路径无新增控制台异常、6 页占位与 2157px 滚动高度保持不变，待真实窗口人工确认长文滚动手感。）**

---

## 2026-09-09 · 阅读器 T0 性能优化：流式加载 + 内容寻址缓存 + worker 预热 + EPUB 预取 + CJK 字形

对标 Edge「WebUI 2.0 资源管理提速」/ Foxit「内存映射·预加载缓存」/ Sumatra「秒开」的思路，落到 web 技术栈的等价手段，聚焦"打开更快、重复打开不重传"。

**⚡ 性能**
- **A2 文件响应改为内容寻址缓存 + 流式**（`backend/api.py` `get_document_file`）：阅读路径去掉 `filename` → **inline + 无 content-disposition**，`Cache-Control: private, max-age=31536000, immutable`，ETag 用文档 sha256。`?v=<sha256>` 由前端 URL 携带（sha256 `UNIQUE NOT NULL` 且导入后 stored 文件永不改动 → 同内容同 URL、改内容换 URL），故重开命中磁盘缓存、Range 分段也从缓存读，**不再整份重传**；且天然不会复发历史"打开 A 显示 B"跨库串缓存 bug。下载按钮拆到 `?dl=1` → `attachment` 弹保存框 + `no-store`，不污染阅读缓存（新增 `api.docDownloadUrl`、`library.js` 下载改用它）。
- **A1 pdf.js 显式流式加载**（`reader.js` `getDocument`）：`disableStream:false`、`disableAutoFetch:false`、`rangeChunkSize:256KB`——先渲首页/视口页、后台预取其余，localhost 下减少 Range 往返。
- **F2 共享 pdf.js worker 预热**（`reader.js` 新增 `getSharedWorker/warmReader`，`app.js` 空闲时调用）：应用启动 idle 时提前起 worker，真实打开复用同一个（`doc.destroy()` 不销毁外部 worker），省掉每次首开的 worker 冷启动；含 worker 意外终止时销毁重建、回退自动 worker 重试一次的兜底。
- **D1 EPUB 相邻章节预取**（`reader.js` `epubPrefetchAround`）：flip 模式（原本只渲当前章）在当前章加载后，空闲把前/后/后二章预取进 `_epubChapterCache`，翻章命中缓存即时渲染、不再等网络。

**✨ 新增**
- **A4 CJK 字形与基础字体**：引入 pdf.js 4.8.69 自带的 `cmaps/`（168 个 .bcmap，1.4 MB）与 `standard_fonts/`（16 个 .pfb，0.8 MB）到 `frontend/vendor/`，`getDocument` 接 `cMapUrl/cMapPacked/standardFontDataUrl`（reader + cover 均接）。中日韩非内嵌字体的 PDF 不再缺字/方框；随 `--add-data frontend` 自动打进 exe。

**验证**：后端 `curl` 确认阅读路径 200 immutable+inline+accept-ranges、Range 请求 206 `content-range`、`?dl=1` 仍 attachment+no-store、`/vendor/cmaps/*.bcmap` 与 `/vendor/standard_fonts/*.pfb` 静态 200；`tools/test_backend.py` 73 passed / 0 failed 无回归。**（浏览器内 worker 复用、流式首屏与 CJK 实际渲染仍需人工视觉确认。）** 源码改动前已快照 `backup\src-20260909-v1-pre-reader-opt`（本仓库非 git，故留源码回退点）。

---

**✨ 新增**
- **实时力导向物理（对标 Obsidian 图谱）**：图谱不再是一次性布局，而是持续运行的物理模拟——**拖拽任意节点，相邻节点会像弹簧一样被牵动、整张网络实时重排**。用自研轻量引擎（`frontend/js/graphsim.js`，d3-force 语义、无第三方依赖）接管坐标，cytoscape 只负责渲染；模拟稳定后自动停帧省电，一有交互即唤醒。
- **「力学」可调面板**：右侧栏新增与 Obsidian 一致的四项力度滑块——**图谱向心力**(0–1, 默认 0.52)、**节点间的排斥力**(0–20, 默认 10)、**相连节点间的吸引力**(0–2.5, 默认 1)、**连线长度**(30–500, 默认 250)，拖动即时生效；侧栏分组可折叠。
- **稀疏标签自动隐藏**：标签节点仅在其关联文档数 **≥ 全部文档数的 9%** 时才显示（后端 `/api/graph?min_tag_ratio` 可调），去掉只挂一两篇的噪声标签，图谱更聚焦。
- **图谱状态记忆**：四项力学参数、是否显示标签、相似边阈值都会**自动保存**，退出软件再打开时恢复上次的配置（存于设置项 `graph` 键，独立存储不干扰其它偏好）。

**🔧 优化**
- 「显示标签 / 相似阈值」从顶栏移入侧栏「筛选」分组，整体布局更贴近 Obsidian；顶栏保留搜索、建立联系、重新布局、缩放适配、刷新。

**✅ 验证**
- `tools/test_graphsim.mjs`（node）：12/12 通过——布局稳定不发散、默认参数下连线均长≈250、四参数单调响应正确、拖拽 pin 与邻居联动成立。
- `tools/test_backend.py`：73/73 通过，含新增「标签 9% 过滤」4 项断言（低阈值显示 / 高阈值隐藏 / 隐藏时连带去边 / 默认阈值保留）。
- 设置读写往返、i18n 全量审计（0 缺漏）通过。浏览器内的动画目视验证因本会话权限策略未能自动执行。

---

## 2026-09-09 · 英文界面审计：补全后端错误消息翻译

**✅ 审计结果**
- 全量扫描 5 个视图 + 公共模块共 453 处 `t()/tf()` 文案，英文缺口 **0**；逐行核对剩余的未包装中文均为「经字典动态渲染的 key」（导航 / 设置分区）或有意保留的界面文字（语言按钮使用语言本名「中文 / English」，属惯例）。
- 修复了唯一一类此前不可达的文案：**后端接口返回的中文错误消息**（如「该分类已存在」「图片过大（上限 12 MB）」）此前即使界面语言为 English 也会以中文弹出。现在 API 错误统一经翻译层（frontend/js/api.js 错误出口接 `t()`），9 条中文错误消息全部提供英文版；未登记的消息仍回退原文，不影响现有中文匹配逻辑（如 OCR 并发提示）。
- 新增可复用的审计脚本 `tools/check_i18n.py`：检查每个 `t()` key 的中英覆盖、疑似未包装中文、后端错误消息与翻译表是否同步（防后端改文案后翻译失联）。当前全部通过：EN 458 条 / 运行时消息 9 条，漂移 0。

**🐛 修复**
- 清除 BACKEND_EN 翻译表中 3 条从未被引用的陈旧条目（doc_id required、card not found 等自映射项），改为真正由后端抛出的 9 条消息。

---

## v0.4.1 · 2026-09-09 · 修复：翻页续读 / OCR 引擎失效 / 图谱标签染色

**🐛 修复**
- **翻页 / 双页模式下切到其它模块再回来，文档回到第 1 页**：阅读会话恢复重开文档时，初始页变量被默认值 1 遮蔽，回退不到上次保存的阅读位置（垂直滚动不受影响）。现在切走再回来自动落在上次翻到的跨页。
- **OCR 识别不完成 / 报错的根因**：1.21 及更新版本的 onnxruntime（CPU 与 DirectML）在打包后的 exe 内无法加载 DLL（普通 python 环境正常，仅冻结应用受影响），旧版安装逻辑会静默装到坏版本，导致识别卡死或报错。现已将运行时版本锁定到实测可用的 1.20.1（CPU + GPU），并把「一键安装 / 修复」升级为带真实识别冒烟验证，装错即时报错提示重试。
- **OCR 引擎缺失 / 损坏时文档不再卡在「处理中」**：回落到「待识别」并记录原因，在文档上可直接重试。

**✨ 新增**
- **知识图谱标签节点支持染色**：点选标签节点即可在详情面板挑颜色（与文档节点一致，可恢复默认）。
- **图谱染色配色从 8 色扩到 16 色**，文档与标签节点通用。
- 修复恢复默认色后节点颜色不立即刷新的问题。

**如何验证**：阅读中把翻页方式设为横向并翻到某页 → 切到图谱再点「文档库」应回到该跨页；设置 → OCR 安装一次引擎（GPU 可选）后，对扫描版书点「识别」应正常出字。

---

## 2026-09-09 · 体验修复：界面状态记忆 / 阅读会话 / 最近阅读排序

**🐛 修复**
- **桌面窗口重启后界面状态不再丢失**：此前 exe 窗口每次启动都回到默认状态（文档库列表/网格、排序方向、侧边栏收起、明暗主题、阅读器字号等自定义全部重置）——根因是内嵌窗口以“无痕会话”运行（pywebview 默认 `private_mode`），localStorage 从不写入磁盘。现改为持久化会话，偏好数据存于 `data/.webview2-profile`（随数据目录一起备份、可携带），重启后恢复上次自定义状态。
- **切换模块不再关闭阅读中的文档**：阅读中切到知识图谱 / 复习 / 收件箱 / 设置等任意模块后，打开的文档保持为“当前阅读”（阅读会话）；从任意模块点回「文档库」即回到那本书继续阅读（自动落在上次进度）。阅读页左上「←」，或阅读中再点一次「文档库」，才是返回列表并结束会话。
- **「最近阅读」排序对未翻页的书不再失效**：此前只有翻页/滚动并停顿约半秒才会记录阅读时间，快速打开或读几页就切走、直接关窗口的书永远排不进“最近阅读”。现在：**打开文档即记一次阅读时间**；**离开阅读（返回 / 切换模块 / 关闭窗口）时立即落盘**最后位置与时间。
- **「最近阅读」排序稳定可复现**：同一时刻/NULL（从未读过）并列时按文档 id 兜底，不再出现乱序。

**如何验证**：重开软件 → 文档库视图、侧边栏与上次一致；打开一本书再切到图谱/复习 → 点「文档库」回到原书；「最近阅读」排序里刚打开过的书立即排到最前。

---

## v0.4.0 · 2026-09-08

- **EPUB 横向翻页升级为按屏分页**：一章按屏幕高度切成多屏，章内逐屏翻、章首尾自动跨章；整页插图自动收进一屏不裁切；字号/窗口尺寸/全屏变化自动重排；修复首次打开显示不良
- **一键全屏阅读**（PDF/EPUB，横向模式与右侧栏随全屏保留）
- **EPUB 阅读优化**：正文 A－/A＋ 字号调节、内链跳转排版更顺、章节内悬浮元素（`position:fixed/absolute`）自动中和避免遮挡正文
- **PDF 双页浏览**（书册跨页惯例：第 1 页独占，之后 2-3 / 4-5 成跨页；垂直与横向模式均可用）
- **知识图谱**：节点按类型染色 + 手动建立文档间联系
- **文档库**：五种排序（最近导入 / 最近更新 / 最近阅读 / 标题 / 年份 / 阅读进度）+ 升/降序切换并记住偏好；侧边栏筛选组可收起（记忆状态）；自建分类
- **双语界面 + 高清 SVG 图标**：中文/English 一键切换并持久化；导航与设置图标全部重绘为矢量内联 SVG

---

## 更早历史

v0.4.0 之前的功能与修复记录见 `README.md`（🎨 / 🗺 等章节）与 GitHub 提交历史。
