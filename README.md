# Product Research — Claude Code Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for deep product analysis and company capital intelligence. Systematically dissects consumer/tech/internet products while uncovering the company's equity structure, funding history, ultimate controllers, investor networks, and capital dynamics.

## What It Does

```
1. Confirm target  -->  2. Web research  -->  3. Company structure
      |                      |                       |
      v                      v                       v
4. Capital analysis  -->  5. Product deep-dive  -->  6. Generate report
      |                                                    |
      v                                                    v
7. Save to vault  -->  8. (Optional) Feishu sync  -->  9. Git commit
```

Given a product or company name, this skill:

1. **Confirms the target** — anti-confusion mechanism for same-name products
2. **Researches from 20+ data sources** with credibility-ranked priority (P0-P3)
3. **Maps the complete equity chain** — product -> operating entity -> holding company -> ultimate controller
4. **Analyzes capital structure** — funding rounds, investors, valuation estimates
5. **Deep-dives the product** from two perspectives:
   - **Investor view**: TAM-SAM-SOM, Business Model Canvas, Porter's Five Forces, Bull/Bear cases
   - **Product view**: User personas, AARRR funnel, Jobs-to-be-Done, competitive comparison
6. **Generates a structured 7-section report** in Markdown
7. **Optionally syncs to Feishu** (Lark) documents

## Usage

```bash
# Natural language
> 调研一下 TapNow 这个产品
> 帮我分析一下这家公司的融资情况

# Slash command with options
> /product-research TapNow --save-path work/AIMO-TeCH --feishu

# With insider info
> /product-research TapNow --known-info "融资 3000-5000 万，红杉领投"
```

## Report Structure

The skill outputs a structured Markdown report with 7 sections:

```
# [产品名] 产品调研报告

## 一、公司概览
   1.1 公司实体（含股权穿透链）
   1.2 创始团队
   1.3 产品演进时间线

## 二、融资与估值
   2.1 公开融资信息 + 母公司财务状况
   2.2 非公开投资信息（来源 + 置信度标注）
   2.3 估值推算（融资反推法）
   2.4 待确认信息清单

## 三、产品分析
   3.1 产品定位
   3.2 核心能力矩阵（Capability-Modality Matrix）
   3.3 差异化功能
   3.4 集成/技术栈

## 四、投资人视角分析
   4.1 TAM-SAM-SOM 市场规模
   4.2 商业模式画布（BMC）
   4.3 竞争格局（Porter's Five Forces）
   4.4 投资亮点与风险（Bull/Bear Case）

## 五、产品视角分析
   5.1 用户画像（User Persona）
   5.2 AARRR 漏斗
   5.3 Jobs-to-be-Done
   5.4 竞品对比矩阵

## 六、市场数据与商业模式
   6.1 用户与流量数据
   6.2 定价策略
   6.3 知名客户案例

## 七、结论与建议
```

## Data Source Priority

Sources are ranked by credibility and structure. The skill annotates every data point with its source.

| Priority | Category | Sources | Use For |
|----------|----------|---------|---------|
| **P0** | Public filings | SEC EDGAR, HKEX, SSE/SZSE | Financial data, equity structure |
| **P0** | Business registry | Tianyancha, Qichacha, NECIPS | Equity penetration, controllers, investments |
| **P1** | VC databases | IT Juzi, Xiniudata, Crunchbase, PitchBook | Funding rounds, valuations, investors |
| **P1** | Product data | SimilarWeb, Sensor Tower, G2, Product Hunt | Traffic, downloads, user reviews |
| **P2** | Media reports | 36Kr, LatePost, Huxiu, Caixin, Bloomberg, Reuters | Industry analysis, insider reporting |
| **P3** | Patents | Google Patents, Zhihuiya | Technology moat verification |

Forum discussions (Xueqiu posts, Eastmoney) are **never** used as sole evidence — only as leads that must be cross-verified through P0-P2 sources.

## Key Design Decisions

### Anti-Confusion Mechanism

Same-name products are extremely common. Real example: "TapNow" is simultaneously a Japanese Gen-Z social app (Sango Technologies) and a Chinese AI creation engine (Shenzhen Tianke Intelligent Technology). Crunchbase/Dealroom pointed to the wrong entity.

The skill forces identity confirmation through: **product URL -> operating entity -> business registry** before starting any analysis.

### Search Degradation Strategy

Web research tools are frequently blocked. The skill uses a cascading fallback:

```
WebSearch (fast) -> WebFetch (direct URL) -> Playwright + DuckDuckGo (fallback)
```

Chinese business databases (Tianyancha, Qichacha, Aiqicha) almost always trigger CAPTCHAs. The skill works around this by extracting registry data from secondary sources: news articles citing Tianyancha, SEC filings, HKEX disclosures.

### Non-Public Information Handling

When users provide insider info (funding amounts, investor names from private sources), the skill:
- Tags it separately with source and confidence level
- Never mixes it with verified public data
- Uses it as a lead for targeted verification
- Maintains a "pending confirmation" checklist

### Equity Chain Template

Every report includes a complete ownership chain:

```
Ultimate Controller / Parent Company
  └─ Intermediate Holding Entity (shareholding %)
       └─ Operating Entity (registration, capital, legal rep)
            └─ Product Brand
```

## Analysis Frameworks

Detailed framework specifications are in `references/analysis-frameworks.md`:

| Framework | Section | Purpose |
|-----------|---------|---------|
| TAM-SAM-SOM | Investor view | Market size estimation (3 layers) |
| Business Model Canvas | Investor view | 9-element business model analysis |
| Porter's Five Forces | Investor view | Competitive landscape assessment |
| Bull/Bear Case | Investor view | Balanced investment thesis |
| User Persona | Product view | 3-4 target user archetypes |
| AARRR Funnel | Product view | Growth metrics analysis |
| Jobs-to-be-Done | Product view | Core user need decomposition |
| Competitive Matrix | Product view | Feature/positioning comparison |
| Unit Economics | Optional | CAC, LTV, margins (if data available) |

## Project Structure

```
product-research-skill/
├── SKILL.md                           # Main skill definition (9-step workflow)
├── references/
│   └── analysis-frameworks.md         # Detailed analysis framework specs
├── LICENSE
└── README.md
```

## Installation

### Option 1: Copy

```bash
git clone https://github.com/YOUR_USERNAME/product-research-skill.git
cp -r product-research-skill ~/.claude/skills/product-research
```

### Option 2: Symlink (recommended for development)

```bash
git clone https://github.com/YOUR_USERNAME/product-research-skill.git
ln -s $(pwd)/product-research-skill ~/.claude/skills/product-research
```

Restart Claude Code after installation. The skill triggers on "调研一下这个产品", "公司背景调查", `/product-research`, etc.

## Prerequisites

- **Claude Code** with WebSearch, WebFetch, and Bash tools
- Optional: **Playwright MCP server** for degraded search scenarios
- Optional: **[feishu-cli](https://github.com/nicepkg/feishu-cli)** for Feishu document sync

No API keys or paid services are required. The skill uses publicly available data sources. For deeper analysis, having accounts on Tianyancha, Crunchbase, or PitchBook will improve data coverage.

## How Claude Code Skills Work

Skills are Markdown-based instruction sets that extend Claude Code:

1. **Metadata** (`name` + `description` in YAML frontmatter) is always in context — used for trigger matching
2. **SKILL.md body** loads when the skill triggers — contains the full workflow
3. **References** (`references/`) load on-demand when referenced by the skill

See [Claude Code Skills Documentation](https://docs.anthropic.com/en/docs/claude-code/skills) for details.

## License

[MIT](LICENSE)
