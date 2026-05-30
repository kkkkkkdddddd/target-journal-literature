# target-journal-literature
 (找目标期刊论文)

A Claude Code skill for searching related academic literature in target journals and generating structured knowledge cards.

## What It Does

Searches for papers related to your research topic within a specific target journal, then generates:

- **Individual knowledge cards** for each paper (with abstract, citation format, relevance analysis)
- **An index page** summarizing all results with topic grouping and relevance ratings

## Trigger Phrases

- "找相关论文" / "搜几篇相近的文章"
- "在XX期刊里找方向接近的"
- "搜5篇 Applied Energy 上和主论文方向相近的文章"
- "找参考文献" / "相近引用"

## How It Works

1. **Understand your research direction** — reads your current paper summary to extract keywords and methodology
2. **Multi-round Tavily Search** — performs 2-3 rounds of search with different keyword combinations
3. **Generate knowledge cards** — one `.md` file per paper, including:
   - Bilingual title (Chinese + English)
   - Authors, journal, volume/issue/pages
   - Chinese abstract summary (original, not copied)
   - Relevance analysis to your main paper
   - Citation format (both Chinese and English)
4. **Create index page** — grouped by topic with relevance ratings (high/medium/low)
5. **Update wiki index** — integrates with your existing research wiki structure

## Output Structure

```
wiki/summaries/{Journal Name}/
├── {Abbr}-{Journal}相近方向文献索引（{English Index Title}）.md
├── {Author}-{Year}-{Chinese Title}（{English Keywords}）.md
├── {Author}-{Year}-{Chinese Title}（{English Keywords}）.md
└── ...
```

## Card Format

Each knowledge card contains:

```yaml
---
type: summary
title: "中文标题（English Title）"
authors: [Author1 (Affiliation), Author2, et al.]
journal: "Journal Full Name"
year: 2024
doi: "https://doi.org/..."
relevance: high|medium|low
tags: [keyword1, keyword2]
---

# Summary
[200-300 word Chinese abstract]

# Relevance to Main Paper
[2-3 sentences on methodology comparison]

# Citation
> Author. Title[J]. Journal, Year, Vol(No): Pages.
```

## Search Strategy

- **Round 1**: Broad search with journal name + core keywords
- **Round 2**: Precise ISSN-based search (for Elsevier journals)
- **Round 3**: Author page / citation network search for high-quality supplements

## Supported Journals

Works with any journal, including:
- English: Applied Energy, IEEE Transactions, Electric Power Systems Research, etc.
- Chinese: 电力系统自动化, 电力自动化设备, 中国电机工程学报, etc.

## Requirements

- Claude Code with Tavily Search MCP server configured
- An existing research wiki structure (`wiki/summaries/`)

## Installation

Copy `SKILL.md` to your Claude Code skills directory:

```bash
# For project-level skill
mkdir -p .claude/skills/zhaolunwen
cp SKILL.md .claude/skills/zhaolunwen/

# For user-level skill
mkdir -p ~/.claude/skills/zhaolunwen
cp SKILL.md ~/.claude/skills/zhaolunwen/
```

## License

MIT
