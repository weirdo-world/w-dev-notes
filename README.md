# 开发笔记（w-dev-notes）

记录日常开发过程中遇到的问题、环境搭建、常用命令与解决方案，方便随时查阅与复用。

## 目录结构

```text
w-dev-notes/
├── README.md
├── .gitignore
└── server/                  # 服务器与运维：Linux 操作、服务安装部署、网络配置等
    ├── xray-install.md
    ├── mysql-install.md
    ├── mysql-uninstall.md
    ├── docker-install.md
    └── ubuntu-commands.md
```

后续笔记增多时，可按需新建以下顶层分类目录，并同步更新本文档：

| 规划目录 | 说明 |
| --- | --- |
| `server/` | 服务器与运维（Linux、服务部署、网络、防火墙等） |
| `database/` | 数据库使用、SQL、配置与优化 |
| `frontend/` | 前端开发相关 |
| `backend/` | 后端开发相关 |
| `tools/` | 开发工具与效率软件 |
| `troubleshooting/` | 问题排查与报错解决 |

### 目录扩展规则（内容变多后怎么办）

- 现阶段 `server/` 保持**扁平结构 + 软件名前缀**，文件按名字自然分组（如 `mysql-install.md`、`mysql-uninstall.md`），文件名即关键词，编辑器内快速打开（Ctrl+P）和全文搜索都很方便。
- 当**同一软件的笔记达到 3 篇及以上**，或**单目录文件超过 15 个**时，再为该软件建二级目录，避免过早分层：

```text
server/
├── mysql/
│   ├── install.md
│   ├── uninstall.md
│   └── config.md
├── docker/
│   └── install.md
└── linux-commands.md       # 跨软件的通用知识留在外层
```

## 笔记索引

### 服务器运维（server）

- [Xray 安装与配置（VLESS + REALITY）](server/xray-install.md)
- [Ubuntu 安装 MySQL](server/mysql-install.md)
- [Ubuntu 卸载 MySQL（含一键卸载脚本）](server/mysql-uninstall.md)
- [Ubuntu 安装 Docker（一键脚本 + 手动方式）](server/docker-install.md)
- [Ubuntu 系统常用操作（vi、文件操作、SSH 端口、ufw 防火墙）](server/ubuntu-commands.md)

## 编写规范

- 目录名、文件名统一使用**英文小写**，单词间用连字符分隔（kebab-case），例如 `mysql-install.md`
- 文档正文使用中文记录，每篇文档以一个 `#` 一级标题开头，一篇笔记只记录一个主题
- 文件名体现内容与类型：
  - 安装部署：`xxx-install.md`
  - 卸载清除：`xxx-uninstall.md`
  - 配置说明：`xxx-config.md`
  - 问题排查：`xxx-issue.md`
  - 常用命令：`xxx-commands.md`
- **安装/卸载类文档优先采用「方式一：一键脚本 + 方式二：手动逐步」两部分结构**（顺序：先脚本，后手动）：
  1. **一键脚本**：自包含、可直接保存为 `.sh` 执行，带分步进度输出
  2. **手动逐步命令**：按「第 N 步」拆分的单条/单组命令，每步可独立执行和排错
- **无法一键化的场景**（官方安装脚本、需要人工交互确认的流程）：脚本部分给出官方命令并说明原因，逐步命令仍必须完整可复制执行
- 代码块标注真实语言（`shell`/`sql`/`ini`/`json`/`text`），配置文件不要用 `shell`
- 命令必须真实可执行，删除/覆盖类操作前要有备份或警告提示；不写无法复制执行的伪命令
- **禁止提交真实凭据**（密钥、密码、Token、UUID 等），一律用 `<你的xxx>` 占位符
- 文件统一使用 **LF** 换行（仓库已通过 `.gitattributes` 约束），Windows 编辑器不要改成 CRLF
- 标题、代码块前后各留一个空行，避免 Markdown 渲染与 lint 告警

## 如何添加笔记

1. 在对应分类目录下新建 `.md` 文件；没有合适分类时新建目录
2. 按命名规范命名文件，按编写规范组织内容（可脚本化操作必须「一键脚本 + 手动步骤」双写）
3. 在本文档「笔记索引」中追加链接，保持索引完整
