[English](README.md) | [简体中文](README.zh-CN.md)

<div align="center">

# ieee-paper-search

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform: Claude Code](https://img.shields.io/badge/Platform-Claude%20Code-blueviolet.svg)](https://claude.com/claude-code)
![Type: Agent Skill](https://img.shields.io/badge/Type-Agent%20Skill-blue.svg)
![Scope: IEEE only](https://img.shields.io/badge/Scope-IEEE%20only-brightgreen.svg)

<br>

**一句"帮我查 X 方向的论文"，换回一份筛好序的 IEEE 阅读清单。摘要翻成中文，确认一声 PDF 就到手。**

<br>

[前提条件](#️-前提条件先读这个) · [它做什么](#它做什么) · [安装](#安装) · [用法](#用法) · [工作原理](#工作原理) · [常见问题](#常见问题)

</div>

---

## ⚠️ 前提条件（先读这个）

三个条件缺一不可，装之前先确认。

1. **你的网络必须能访问 IEEE Xplore 全文。**
   PDF 下载靠机构 IP 识别，你机器的 IP 必须落在学校/单位向 IEEE Xplore 注册的 IP 段内。实际上就是**连着校园网或机构 VPN**。没有订阅时搜索和筛选仍可用，但全文下载会返回 HTTP 502。
   快速自检，`curl -sS https://ifconfig.me` 显示的 IP 要在机构网段内，浏览器里打开 `ieeexplore.ieee.org` 的付费论文应能看到解锁的 PDF 按钮。

2. **推荐在 [Claude Code](https://claude.com/claude-code) 上使用。**
   这是一个 Agent Skill（`SKILL.md` + references），在 Claude Code 里设计并日常使用。其他能读 SKILL.md 的框架或许能跑，但未测试。

3. **[paper-search-mcp](https://github.com/openags/paper-search-mcp) + [uv](https://docs.astral.sh/uv/)** 提供搜索后端（OpenAlex + Semantic Scholar）。安装步骤见下。

---

## 它做什么

你说，

> "查一下 DAB converter ZVS modulation 方面的论文"

skill 跑完整管线，**搜索 → IEEE-only 过滤 → 去噪 → 排序 → 卡片呈现 → 你挑 → 下 PDF**，回给你这样的卡片（摘要按你的会话语言翻译），

```
### [1] Optimal ZVS Modulation of Single-Phase Single-Stage Bidirectional DAB AC–DC Converters

🔗 https://doi.org/10.1109/tpel.2013.2292026
📅 2014 · 📍 IEEE TPEL · 📊 引用 471
👤 Jordi Everts; ... ; Johann W. Kolar (ETH/KU Leuven)

**中文摘要**：提出全工况 ZVS 调制方案推导流程，用电流相关电荷分析替代
传统电流/能量分析，考虑开关寄生输出电容充电需求。3.7 kW 双向 EV
充电器原型实验验证。

💡 判断：必读（领域 anchor，471 引用，charge-based ZVS 分析奠基论文）
```

你回一句"1 和 4"，它就用 `ieee-fetch` 只下这两篇。`ieee-fetch` 是专为 IEEE 写的 session-warmed curl，纯靠机构 IP 认证，约 6 秒一篇，不需要 IEEE 账号，不存任何 cookie。

### 为什么只搜 IEEE

列出来的每篇都保证能下到全文。DOI 前缀不是 `10.1109/` 或 `10.23919/` 的论文一开始就被丢掉，列一篇下不动的 Elsevier 论文只会浪费你时间。过滤掉超过 50% 候选时 skill 会主动告诉你，建议换通用搜索。

### 内置的设计取舍

- 搜索源锁死 **OpenAlex + Semantic Scholar** —— Crossref 对 IEEE 论文摘要全空（实测 0/6 vs 6/6），`-s all` 会混进大量误命中
- **排序** = 引用数 + venue 层级 + 年份 + 作者团队，带明确反模式（TPEL 上不满 1 年的 0 引用论文保留，不丢）
- **卡片不用表格** —— 每篇 2-4 句工程腔翻译摘要 + 一词判断（必读 / 代表 / 新方法 / 背景 / 可选）
- **绝不擅自批量下载**，被剔除的论文一律列出原因

## 安装

```bash
# 1. skill 本体
git clone https://github.com/Zane456/ieee-paper-search.git ~/.claude/skills/ieee-paper-search

# 2. ieee-fetch 下载器
mkdir -p ~/.local/bin
cp ~/.claude/skills/ieee-paper-search/scripts/ieee-fetch ~/.local/bin/
chmod +x ~/.local/bin/ieee-fetch
# 确认 ~/.local/bin 在 PATH 里

# 3. 搜索后端
git clone https://github.com/openags/paper-search-mcp.git ~/paper-search-mcp
cd ~/paper-search-mcp && uv sync
```

paper-search-mcp 如果克隆到别处，改一下 `SKILL.md` 里那行 `uv run --directory` 的路径。

## 用法

在 Claude Code 里直接自然语言提问，

- "查一下 DAB converter ZVS modulation 方面的论文"
- "find papers on model predictive control for three-level inverters"
- "近 3 年 GaN 门极驱动设计的论文调研一下"

论文调研类措辞会自动触发本 skill。PDF 默认落在 skill 文件夹的 `downloads/` 里（已 gitignore），带自动维护的 `INDEX.md` 索引。用 `$IEEE_FETCH_DIR` 可改位置。

下载器也可单独使用，

```bash
ieee-fetch 10374220
ieee-fetch "https://ieeexplore.ieee.org/document/10374220"
ieee-fetch 10374220 /tmp/paper.pdf
```

## 工作原理

| 阶段 | 工具 | 细节 |
|---|---|---|
| 搜索 | paper-search-mcp CLI | `-s openalex,semantic`，每源 8 条，可加年份过滤 |
| 过滤 | DOI 前缀 | 只留 `10.1109/` 和 `10.23919/`，按 DOI 去重 |
| 筛选 | [references/triage.md](references/triage.md) | 引用数 → venue 层级 → 年份 → 作者团队 |
| 呈现 | [references/output-template.md](references/output-template.md) | 卡片 + 翻译摘要 + 判断词 |
| 下载 | [scripts/ieee-fetch](scripts/ieee-fetch) | 先访问 `/document/<id>` 预热会话，再取 `stampPDF`，IP 认证 |
| 阅读 | Claude Code `Read` | PDF 直接进上下文做摘要、抽方法 |

`references/` 里还有[数据源选择依据](references/sources.md)、[8 条实战踩坑](references/failure-modes.md)、[完整金案例](references/golden-case-dab-zvs.md)。

## 常见问题

**报 HTTP 502 / 下来个 4 KB 的文件。**
你的 IP 不在机构 IEEE 网段内。连上校园 VPN，用 `curl ifconfig.me` 复查。

**HTTP 200 但 content-type 是 text/html。**
你的机构订阅不含这本期刊。

**能下 Elsevier / Springer / Wiley 吗？**
不能，这是设计取舍。这几家出版商的访问机制各不相同。非 IEEE 论文在你看到列表前就被过滤了。

**这算绕过付费墙吗？**
不算。它用的正是你机构已付费的访问权限，和你在校园网里用浏览器下论文是同一套 IP 认证机制。不共享账号、不存凭据、不做任何绕过。使用时请遵守你所在机构和 IEEE 的服务条款，下载的 PDF 仅限自己科研使用，不得二次分发。

## License

[MIT](LICENSE) © [Zane456](https://github.com/Zane456)
