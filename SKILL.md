---
name: zhaolunwen
description: >
  在指定期刊中搜索与主论文方向相近的学术文献，为每篇文章生成独立的知识卡片（含摘要、链接、引用格式、与主论文关联分析），
  并创建索引页汇总所有结果。适用于：用户说"找相关论文"、"搜几篇相近的文章"、"在XX期刊里找方向接近的"、"引用格式"、
  "找参考文献"、"搜N篇"、"生成文献卡片"、"相近引用"时触发。也适用于用户提到特定期刊名（如 Electric Power Systems Research、
  电力自动化设备、IEEE Trans、Applied Energy 等）并要求搜索相关方向文献的场景。
---

# 找论文 — 特定期刊相近文献检索与卡片生成

## 这个 Skill 做什么

在用户指定期刊中搜索与主论文方向相近的学术文献，将每篇文献做成独立的知识卡片文件，再创建一个索引页汇总。最终产出物是 wiki 中可直接点击跳转的文献卡片集合。

## 核心流程

### Step 1: 理解主论文方向

读取用户当前正在看的论文 summary（通常在 `summaries/主论文/` 下），提取：
- 核心研究方向关键词（中英文）
- 使用的方法论（如 VCG 拍卖、PTDF-DLMP、DistFlow 等）
- 关注的技术（如区块链、智能合约、ADMM 等）

这些关键词将用于后续搜索。

### Step 2: 确定搜索目标

向用户确认（如果用户已在消息中说明则跳过）：
1. **目标期刊**：全名 + 缩写 + ISSN（如需查 ISSN 可用 Tavily 搜索）
2. **搜索数量**：默认 5-10 篇
3. **语言偏好**：中文期刊用中文关键词搜索，英文期刊用英文关键词

### Step 3: 多轮 Tavily 搜索

使用 Tavily Search 进行多轮搜索，策略如下：

**第一轮：宽泛搜索**
```
query: "{期刊名}" + "{核心关键词1}" + "{核心关键词2}" + "{年份范围}"
search_depth: advanced
max_results: 10
```

**第二轮：精确 ISSN 搜索**（如果第一轮结果不够精确）
```
query: "site:sciencedirect.com S{ISSN号}" + "{细分关键词}"
```

**第三轮：作者页面/引用网络搜索**（补充高质量文献）
```
query: "{已找到的高引作者名}" + "{期刊名}" + "{相关关键词}"
```

**搜索技巧**：
- 英文期刊：组合 `site:sciencedirect.com` + ISSN 号 + 关键词
- 中文期刊：直接用中文关键词搜索期刊名
- 如果 `tavily_extract` 失败（ScienceDirect 反爬），改用 `tavily_search` 搜摘要
- 通过 Google Scholar 作者页面交叉验证论文信息（期刊、卷号、页码、DOI）

### Step 4: 创建文件夹

在 `wiki/summaries/` 下按期刊创建新文件夹：
```
wiki/summaries/{期刊中文名或缩写}/
```

### Step 5: 为每篇文献生成独立卡片

每张卡片是一个独立 `.md` 文件，遵循以下格式：

**文件命名**：`{第一作者姓}-{年份}-{中文标题}（{English Title Key Words}）.md`

**文件结构**：

```yaml
---
type: summary
title: "中文标题（English Title）"
authors:
  - 作者1 (单位)
  - 作者2
  - et al.
journal: "期刊全名（English Name）"
volume: "卷号"
issue: "期号"
pages: "页码"
year: 年份
doi: "DOI链接"
link: "ScienceDirect/期刊链接（期刊链接不仅限于sciencedirect就是搜索到这篇文章的链接 保证用户一点这个链接就可以看到文章）"
source_files:
  - pom/raw/articles/{主论文源文件}
tags:
  - keyword1
  - keyword2
  - 期刊缩写
related_concepts:
  - "[[concepts/相关概念页]]"
relevance: high|medium|low
---

# 作者姓 等 (年份)

## 基本信息

| 字段 | 内容 |
|------|------|
| **标题** | 中文标题 |
| **英文标题** | English Title |
| **期刊** | 期刊名 |
| **卷号/期号/页码** | vol. X, no. Y, pp. Z |
| **出版日期** | 年份 |
| **作者** | 作者列表 |

## 摘要

[基于搜索结果撰写的中文摘要，200-300字，覆盖：研究问题、方法、关键结果]

## 与主论文关联

[2-3句话说明本文与主论文的关系：方法论互补性、研究对象重叠度、可借鉴之处]

## 引用格式

> 作者. 标题[J]. 期刊, 年份, 卷(期): 页码.
>
> Author. Title[J]. Journal, Year, Vol(No): Pages.

## 来源

- (source: pom/raw/articles/{主论文源文件})
```

**卡片质量要求**：
- 摘要必须是基于搜索结果的原创中文总结，不是直接复制
- 每篇必须标注与主论文的具体关联（方法论对比、互补性等）
- 引用格式同时提供中英文版本
- `relevance` 字段基于与主论文方向的接近程度判定：
  - **high**: 直接涉及相同核心方法或问题
  - **medium**: 方法论互补或同领域不同角度
  - **low**: 间接相关，提供背景知识

### Step 6: 创建索引页

在同文件夹下创建索引页：`{期刊缩写}-{期刊中文名}相近方向文献索引（{English Index Title}）.md`

索引页包含：
1. **文献卡片列表**：表格形式，带 wikilink 和相近度评级（★）
2. **按主题分类**：将文献按研究主题分组
3. **与主论文的距离评估**：high/medium/low 三级
4. **方法论对比表**：文献 vs 主论文的方法论对照

### Step 7: 更新 INDEX.md 和 log.md

**INDEX.md**：在 `## Summaries` 下新增一个 `### 相近引用（{期刊名}）` 板块，列出索引页和所有卡片的 wikilink。

**log.md**：追加一行：
```
## [YYYY-MM-DD] query | {期刊名}相近方向文献检索 | 1 index + N individual summary cards + INDEX updated
```

## 关键注意事项

### 搜索策略

1. **多轮搜索优于单轮**：用不同关键词组合搜 2-3 轮，覆盖面更广
2. **作者页面是金矿**：找到一篇高度相关的论文后，搜其作者的其他 EPSR/EPAA 论文
3. **中文期刊用中文搜**：《电力自动化设备》等中文期刊直接用中文关键词，不要翻译成英文
4. **ISSN 号过滤**：对 Elsevier 期刊，用 `S{ISSN号去掉横杠}` 作为 ScienceDirect 的 PII 前缀来精确过滤

### 信息完整性

5. **DOI 优先验证**：搜索结果中的 DOI 不一定准确，需要交叉验证
6. **无法获取的信息标记待确认**：如果卷号、页码、DOI 不确定，在卡片中注明"待确认"
7. **摘要质量**：如果搜索结果只有标题没有摘要，基于标题和引用上下文推断研究内容，并注明"基于标题推断"

### Wiki 集成

8. **wikilink 用中英双语文件名**：确保 Obsidian 可以正确解析链接
9. **related_concepts 字段**：尽量链接到 wiki 中已有的概念页
10. **source_files 保持一致**：所有卡片的 source_files 都指向主论文的原始文件

## 输出示例

用户说："搜 5 篇 Applied Energy 上和主论文方向相近的文章"

→ 产出：
```
wiki/summaries/Applied Energy/
├── AE-Applied Energy相近方向文献索引（AE Related Papers Index）.md
├── Zhang-2024-基于ADMM的P2P能源交易分布式优化（ADMM P2P Energy Trading）.md
├── Li-2023-区块链赋能微电网群电力交易（Blockchain Microgrid Group Trading）.md
├── Wang-2024-考虑网络约束的产消者交易机制（Prosumer Trading Network Constraints）.md
├── Chen-2023-智能合约驱动的去中心化电力市场（Smart Contract Decentralized Market）.md
└── Liu-2024-基于博弈论的多微网能源共享（Game Theory Multi-Microgrid Sharing）.md
```
