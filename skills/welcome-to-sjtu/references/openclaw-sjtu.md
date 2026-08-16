# openclaw-sjtu 接入地图

## 目录

- [核对范围](#核对范围)
- [运行约定](#运行约定)
- [核心能力](#核心能力)
- [其他校园能力](#其他校园能力)
- [已知限制](#已知限制)
- [执行安全](#执行安全)

## 核对范围

- 上游仓库：<https://github.com/xhh678876/openclaw-sjtu>
- 本参考按提交 `53d6cdca92efa0435e8a118fd1ec1d7cdf73f2d4`（2026-05-30）核对。
- 上游仍可能变化。执行前优先读取当前检出版本的根 `SKILL.md`、目标脚本帮助和对应子技能，不把本文当成固定 API。
- 上游采用 MIT License。复用或分发其实质代码时保留上游版权与许可文本。

## 运行约定

1. 从上游仓库根目录运行脚本。脚本依赖相对位置查找 `config.json`、`templates/` 和 `fonts/`。
2. 按需从 `config.example.json` 复制配置；不要提交包含真实凭证的 `config.json`。
3. 主功能常用依赖包括 Python 3、`requests`、`beautifulsoup4`、`python-pptx`、`pdfplumber`、`python-docx`、Pillow 等；水源与 SJTU Date 脚本需要 Node.js 18+。
4. 只启用当前任务需要的凭证。Canvas、jCourse、传承、邮箱、水源和龙虾广场各自独立，不要一次性索取全部账号信息。

## 核心能力

| 能力 | 经核对入口 | 状态与用法 |
|---|---|---|
| 作业追踪 | `python3 scripts/canvas_api.py ddls`；`ddls-all` | Canvas 实时；需要 token。`ddls-all` 含统计、待交与反馈 |
| AI 作业辅导上下文 | `python3 scripts/auto_homework.py context <course_id> <assignment_id>` | 拉题面、图片与课件，生成 `ai_prompt.md`；不会自动提交 |
| 紧急作业 | `python3 scripts/auto_homework.py urgent 24` | 列 N 小时内的未交项 |
| 跨平台 DDL | `python3 -m scripts.unified_ddl` | 聚合 Canvas、物理实验、MOOC、实验预约；部分平台需校园网与额外认证 |
| 水源搜索/摘要 | `node scripts/shuiyuan_discourse.mjs search "关键词" --max-results 10` | 实时、严格只读，仅允许 GET；需链接授权 |
| 水源读帖 | `node scripts/shuiyuan_discourse.mjs topic <topic_id> --post-limit 5` | 摘要时保留帖子链接、日期和不同观点 |
| 选课搜索 | `python3 scripts/sjtu_course_review.py search <课程或教师>` | jCourse 实时开放 API，需要 key |
| 教师比较 | `python3 scripts/sjtu_course_review.py compare <课程>` | 评分与评论只是参考，需结合培养方案和时间表 |
| 交大 PPT | `python3 scripts/generate_ppt.py --title "标题" --markdown <内容或文件> --template "模板" --output <文件>` | 本地生成；上游含 9 套模板；需要 `python-pptx` |
| 课件提取 | `python3 scripts/file_extractor.py <文件或目录> [输出目录]` | 支持 PPTX/PDF/DOCX/TXT/Markdown；与 Canvas 下载结合生成总结和提纲 |
| SJTU Date | `node scripts/sjtudate.mjs <command>` | 支持登录、匹配、历史、问卷、心动等；仅在用户主动选择后使用 |
| 龙虾广场 | `skills/lobster-square/scripts/discover.sh` 与 `call.sh` | 每次先拉实时 OpenAPI；API key 形如 `lsq_live_...`，禁止回显 |

### 作业辅导流程

1. 用 `canvas_api.py courses` 获取课程 ID。
2. 用 `canvas_api.py ddls` 或 `auto_homework.py scan` 找到作业。
3. 用 `auto_homework.py context` 下载题面与相关材料。
4. 读取生成的上下文，解释知识点、给解题步骤或学习计划。
5. 不代替用户违反独立完成要求；上游普通 CLI 不会自动提交作业。

### 水源摘要流程

1. 首次运行 `node scripts/shuiyuan_discourse.mjs auth init` 生成授权链接。
2. 用户在水源批准后，用 `auth finish --payload <payload>` 完成授权；凭证保存在权限为 `0600` 的本地文件。
3. 先搜索，再按话题 ID 读取必要楼层；不要无目的抓取整个社区。
4. 区分帖子原话、多个用户的共同观点和模型归纳。政策问题回链官方通知。
5. 遇到 429 按提示等待，不进行循环重试。

### SJTU Date 调用说明

上游脚本支持 `login`、`profile`、`match`、`match-history`、`shoot`、`survey`、`submit-survey` 等命令。当前核对提交中，根 `SKILL.md` 引用了 `skills/sjtu-date/SKILL.md`，但仓库实际未包含该文件，仅包含 `scripts/sjtudate.mjs` 与 `skills/sjtu-date/scripts/sjtudate.mjs`。因此：

- 用户咨询恋爱交友渠道时优先介绍水源「鹊桥」。
- 用户明确选择 SJTU Date 时，先查看当前脚本帮助和源码，不依赖缺失的子技能文档。
- 登录、提交问卷、发心动和任何对外消息均视为敏感或写操作；说明数据用途并在执行前确认。

### 龙虾广场流程

1. 只在用户明确提供 key 或要求使用龙虾广场时启用。
2. 先通过 `discover.sh` 或 `https://clawsjtu.com/api/v1/openapi.json` 获取实时规范。
3. 根据意图读取方法、路径、必填字段和响应 schema，不凭记忆构造请求。
4. GET 类读取可直接执行；POST/PATCH/DELETE 前展示目标和内容并二次确认。
5. 401 要求重新签发 key；429 停止重试并报告限流。

## 其他校园能力

| 场景 | 入口 | 数据性质 |
|---|---|---|
| 往年试卷/资料 | `python3 scripts/sjtu_legacy.py search <关键词>` | 需传承 token；列清单不自动下载，下载可能消耗积分 |
| 教务通知 | `python3 scripts/sjtu_news.py jwc 10` | 实时抓取，回答时保留日期与链接 |
| 交大新闻 | `python3 scripts/sjtu_news.py news 10` | 实时抓取 |
| 邮箱 | `python3 scripts/sjtu_mail.py unread --limit 10` | 需邮箱凭证与校园网/VPN；发送邮件是写操作 |
| 生存手册 | `python3 scripts/sjtu_survive.py search <拼音 slug>` | 实时 GitBook；拼音检索通常更准 |
| 食堂 | `python3 scripts/sjtu_canteen.py recommend` | 上游内置静态数据 |
| 图书馆 | `python3 scripts/sjtu_library.py info` | 上游内置静态数据，不含实时座位余量 |
| 空教室 | `python3 scripts/sjtu_classroom.py empty --building <教学楼>` | 静态教室清单，不代表实时空闲 |
| 校车/教学周/校历 | `python3 scripts/sjtu_info.py bus|week|calendar` | 上游内置静态数据，跨学期需核对 |
| 正版软件 | `python3 scripts/sjtu_software.py search <软件>` | 优先实时，失败会回退到内置数据 |
| 校园图片 | `python3 scripts/sjtu_visual.py search <关键词>` | 实时官方图库 |
| 在线工具 | `python3 scripts/sjtu_tools.py list` | 静态目录，详情会探测可达性 |
| 镜像换源 | `python3 scripts/sjtu_mirror.py pip|conda|brew|docker|npm` | 只打印指引，不改本机配置 |

## 已知限制

- 上游把食堂、图书馆、空教室、巴士、教学周、校历和在线工具标为静态数据；不能把输出写成“刚刚查询”。
- 核对提交中 `canvas_api.py ddls-all` 的学期起始日和课表 ICS 的学期区间存在硬编码，跨学期前必须查看当前源码或官方校历。
- 邮箱、水源以及部分物理实验/实验预约平台通常需要校园网或 VPN。
- `handwrite_pdf.py` 需要额外安装 `handright`；`sjtu_mirror.py list` 的上游后端接口曾变化。
- 上游 README 与当前根 `SKILL.md` 对 jCourse 认证方式有过迭代；执行时以当前代码和根 `SKILL.md` 为准。

## 执行安全

- 不输出真实 token、Cookie、邮箱密码、jAccount 密码或龙虾 key；错误日志也需脱敏。
- 优先使用可撤销 token 或链接授权，不建议把 jAccount 主密码作为普通对话内容收集。
- 发邮件、下载消耗积分、提交作业、提交问卷、发心动和龙虾广场写/删操作必须先确认。
- 安装依赖、创建持久化凭证文件或修改用户配置前说明将改变什么；仅做信息咨询时不要擅自安装。
