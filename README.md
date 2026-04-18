# Product Research

> Claude Code Skill — 产品调研与公司资本情报分析

对消费级/科技/互联网产品进行全方位拆解，系统挖掘产品背后的公司股权结构、融资历史、实际控制人、投资人网络及资本博弈逻辑。输出结构化调研报告。

## Quick Start

```bash
# 1. 安装：克隆到 Claude Code skills 目录
git clone https://github.com/AsterZephyr/product-research-skill.git
ln -s $(pwd)/product-research-skill ~/.claude/skills/product-research

# 2. 重启 Claude Code

# 3. 使用
> /product-research TapNow
> 调研一下 TapNow 这个产品
> 帮我分析一下这家公司的融资情况
```

## 工作流程

```
输入                    分析                    输出
─────────────────    ─────────────────    ─────────────────
产品/公司名      ──→  确认目标（反混淆）
                      │
已知 URL         ──→  多源信息采集（P0-P3）
                      │                     7 节结构化报告
内部信息（可选）  ──→  公司结构 + 股权穿透  ──→    │
                      │                       ├─→ 本地 Markdown
                      资本分析 + 产品拆解       ├─→ 飞书文档（可选）
                      │                       └─→ Git commit
                      双视角深度分析
                      (投资人 + 产品)
```

## 核心能力

### 1. 反混淆机制

同名产品极为常见。真实案例："TapNow" 同时是日本社交 App（Sango Technologies）和深圳 AI 创作引擎（添科智能科技）。Crunchbase 指向了错误实体。

本技能强制执行验证链：**产品官网 → 运营主体 → 工商信息** ，确认后才展开分析。

### 2. 分级信息源

按可信度分 4 级，每个数据点标注出处：

| 级别 | 类别 | 来源 | 用途 |
|------|------|------|------|
| **P0** | 上市公司文件 | SEC EDGAR、港交所、上交所/深交所 | 财务数据、股权结构 |
| **P0** | 工商数据 | 天眼查、企查查、国家企业信用信息公示 | 股权穿透、实控人、对外投资 |
| **P1** | 投融资数据库 | IT桔子、烯牛、Crunchbase、PitchBook | 融资轮次、估值、投资人 |
| **P1** | 产品数据 | SimilarWeb、Sensor Tower、G2 | 流量、下载量、评价 |
| **P2** | 深度报道 | 36氪、晚点、虎嗅、财新、Bloomberg | 行业分析、内幕 |
| **P3** | 专利数据 | Google Patents、智慧芽 | 技术壁垒验证 |

**原则**：论坛讨论（雪球帖子、股吧）只能作为线索，不能作为事实依据，必须通过 P0-P2 交叉验证。

### 3. 股权穿透

每份报告包含完整的控制链：

```
最终控制人 / 母公司
  └─ 中间控股实体（持股比例）
       └─ 运营主体（注册地、成立时间、注册资本、法人）
            └─ 产品品牌
```

### 4. 双视角分析

**投资人视角**：

| 框架 | 分析内容 |
|------|---------|
| TAM-SAM-SOM | 三层市场规模估算 |
| Business Model Canvas | 商业模式九要素 |
| Porter's Five Forces | 竞争格局五力分析 |
| Bull / Bear Case | 看多 vs 看空论点，均衡呈现 |

**产品视角**：

| 框架 | 分析内容 |
|------|---------|
| User Persona | 3-4 个目标用户画像 |
| AARRR Funnel | 获客 → 激活 → 留存 → 变现 → 推荐 |
| Jobs-to-be-Done | 用户核心需求拆解 |
| Competitive Matrix | 竞品功能/定位对比（至少 3 个） |

详细框架说明见 `references/analysis-frameworks.md`。

### 5. 搜索降级策略

Web 搜索工具经常受限，技能内置级联回退：

```
WebSearch（快） → WebFetch（直连 URL） → Playwright + DuckDuckGo（兜底）
```

中国商业数据库（天眼查、企查查）几乎必触验证码。应对方案：从新闻报道、SEC 文件、港交所披露中提取工商信息。

## 报告结构

```markdown
# [产品名] 产品调研报告

> 调研日期：2026-04-18
> 产品官网：https://example.com

---

## 一、公司概览
   1.1 公司实体（含股权穿透链）
   1.2 创始团队
   1.3 产品演进时间线

## 二、融资与估值
   2.1 公开融资信息 + 母公司财务
   2.2 非公开投资信息（标注来源和置信度）
   2.3 估值推算（融资反推法）
   2.4 待确认信息清单

## 三、产品分析
   3.1 产品定位
   3.2 核心能力矩阵
   3.3 差异化功能
   3.4 技术栈 / 集成

## 四、投资人视角（TAM-SAM-SOM / BMC / Porter / Bull-Bear）
## 五、产品视角（Persona / AARRR / JTBD / 竞品对比）
## 六、市场数据与商业模式（流量、定价、客户案例）
## 七、结论与建议
```

## 使用方式

```bash
# 基本用法
> /product-research TapNow

# 指定保存路径
> /product-research SomeProduct --save-path path/to/output

# 同步到飞书
> /product-research TapNow --feishu

# 提供内部信息（会单独标注置信度）
> /product-research TapNow --known-info "融资 3000-5000 万，红杉领投"

# 自然语言也可以
> 帮我调研一下这家公司的背景和融资情况
> 做个竞品分析
```

## 项目结构

```
product-research-skill/
├── SKILL.md                         # 技能主文件（9步工作流定义）
├── references/
│   └── analysis-frameworks.md       # 分析框架详细规范
├── .gitignore
├── LICENSE
└── README.md
```

## 环境要求

| 依赖 | 用途 | 必需 |
|------|------|------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | 技能运行环境 | Yes |
| Playwright MCP Server | 搜索降级时的浏览器自动化 | No |
| [feishu-cli](https://github.com/nicepkg/feishu-cli) | 飞书文档同步 | No |
| 天眼查 / Crunchbase 账号 | 更深入的数据覆盖 | No |

无需 API Key 或付费服务。技能使用公开可用的数据源。

## 安装

### 方式一：Symlink（推荐，方便更新）

```bash
git clone https://github.com/AsterZephyr/product-research-skill.git
ln -s $(pwd)/product-research-skill ~/.claude/skills/product-research
```

### 方式二：直接复制

```bash
git clone https://github.com/AsterZephyr/product-research-skill.git
cp -r product-research-skill ~/.claude/skills/product-research
```

安装后重启 Claude Code 即可。

## 什么是 Claude Code Skill

Skill 是基于 Markdown 的指令集，用于扩展 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 的能力：

- **YAML frontmatter**（`name` + `description`）始终在上下文中，用于触发匹配
- **SKILL.md 正文** 在技能触发时加载，包含完整工作流
- **references/** 按需加载，提供补充文档

详见 [Claude Code Skills 官方文档](https://docs.anthropic.com/en/docs/claude-code/skills)。

## License

[MIT](LICENSE)
