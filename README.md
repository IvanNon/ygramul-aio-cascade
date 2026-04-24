# Ygramul AIO Cascade

A Claude Skill that writes AI-Overview-optimized opening blocks for any article — using 14 specialized editor-judges that iteratively refine the block until it converges. Based on the Karpathy auto-research loop.

---

## The name

The methodology takes its name from Ygramul, the spider in Michael Ende's The Neverending Story — a nod to the fact that search crawlers are, themselves, spiders.

Built by [Ivan Nonveiller](https://www.linkedin.com/in/ivannonveiller/) — enterprise SEO/GEO strategist.

---

## What it does

You give it a URL. It produces a 30–50 word opening block optimized for citation by AI Overviews, Perplexity, and other AI search surfaces.

- Fetches the target page
- Pulls the top 3 SERP competitors for your primary query
- Drafts a baseline block using proven patterns
- Runs up to 30 iterations where 14 specialized judges edit the block — one dimension per judge
- Weighted selection with decay (judges returning `no_change` get deprioritized) and recency boost (unused judges surfaced)
- Halts early when the block stabilizes (typically around iteration 15)
- Returns: final block + full evolution log showing every edit and why it was made

---

## Why this instead of a single optimization prompt

A single prompt can't simultaneously optimize for factual accuracy, query-intent match, extraction-friendliness, entity density, competitor differentiation, brand voice, and format fit. It collapses to whichever dimension it weighs heaviest.

This skill splits the problem into 14 focused judges — each editing for one dimension. The ensemble enforces constraints simultaneously rather than sequentially. That's what gets you a block that survives real-world AIO extraction rather than one that's rubric-compliant in isolation.

---

## The 14 judges

**Guardrails (2)** — enforce hard constraints
- Criteria Compliance (30–50 words, second person, query-answering opener)
- Fact Accuracy vs Source (every claim supported by the page)

**Optimizers (9)** — push the block in a specific direction
- AIO Citation Potential
- Information Gain vs SERP competitors
- Brand Voice
- Self-Uniqueness (block adds what page opener lacks)
- Specificity (concrete entities over vague claims)
- Entity Density (target 3+ named entities)
- Competitor Antipattern (avoid phrasings top 3 SERP results share)
- Query-Intent Alignment (definitional, procedural, causal, verification)
- Temporal Freshness Signal

**Reframers (3)** — question the structure itself, run rarely
- Reverse-Query Coherence
- Format Fitness (prose vs list vs Q&A)
- Schema Alignment (only fires if page has structured data)

---

## Install

1. Download `SKILL.md` from this repo
2. Open [claude.ai](https://claude.ai) → click your profile → **Settings** → **Capabilities** → **Skills**
3. Click **Upload skill** and select the file
4. Done — the skill is now available in your account

**Requires** Claude Pro, Team, or Enterprise. Free tier doesn't support custom skills.

---

## Use

Start a new chat and type:

> Run the Ygramul AIO cascade on https://yourwebsite.com/your-blog-post

Claude will:

1. Ask you to confirm the primary query
2. Run the cascade (2–5 minutes)
3. Return the final block + evolution log

Copy the block and paste it on your page under the H1, before the first H2. **That placement matters** — AI Overviews extract from the block immediately after the H1.

---

## Expected results

- 2–4 weeks after shipping: Google re-crawls, AI Overview starts considering the new opener
- 4–8 weeks: measurable AIO citation rate on your target query
- Cost per cascade: ~$1–3 in Claude API usage (covered by your Pro/Team/Enterprise plan)

---

## When not to use

- Pages under 300 words (not enough source content for the judges to work with)
- Pages that are still in draft (it needs the published version to read)
- Pages where conversion matters more than citation (AIO capture isn't always the right target)
- Queries that are already dominated by your own domain (there's nothing to differentiate against)

---

## Contributing

Found a judge that improved output reliability? Found a convergence edge case? PRs welcome. Open an issue first for anything bigger than a typo fix.

---

## License

MIT. Use it, modify it, ship it in your own tools.

---

## About the author

**Ivan Nonveiller** — enterprise technical SEO and GEO strategist. 10+ years optimizing search for brands including Walmart, AutoTRADER, Ferrari, Maserati, Ritz-Carlton, Sotheby's, and BetterSleep. Based in Montréal.

- LinkedIn: [ivannonveiller](https://www.linkedin.com/in/ivannonveiller/)
- Ygramul methodology: [rushmediaagency.com](https://rushmediaagency.com)

This skill is the open-source version of Ygramul's Lever 4 (Content) system.
