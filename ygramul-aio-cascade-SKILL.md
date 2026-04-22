---
name: ygramul-aio-cascade
description: "Use this skill when the user wants to optimize a content page for AI Overview (AIO) and LLM citation capture by generating and evolving a short extraction-optimized summary block through multiple editorial passes. Trigger on phrases like: run the cascade, run Ygramul, AIO cascade, AIO block, AIO summary, AI Overview optimization, optimize for AIO, optimize for Perplexity, make this page cite-worthy, improve AI search capture, rewrite opener for AIO, generate an AIO block, evolve this opener, judge cascade, auto-optimize my blog post. Also trigger when a user pastes a URL and asks to 'run the loop', 'improve this for AI search', 'generate a block for this page', or 'run 20 iterations on this URL'. Do NOT use for full SEO audits (use tech-seo-audit skill instead), for long-form content writing, or for traditional SERP ranking optimization without AI-search framing. Produces a 30-50 word block plus an evolution log showing how 13 specialized editor-judges transformed the block across up to 30 iterations."
---

# Ygramul AIO Cascade

A self-evolving editorial cascade that generates and iteratively improves an AIO-summary block for a target URL using a weighted ensemble of 13 specialized editor-judges.

**Output:** a 30-50 word extraction-optimized block (intended for placement under H1, before first H2) + a full evolution log showing which judges fired, what they changed, and when the block converged.

**Method:** this is the productized Lever 4 content judge system from the Ygramul SEO/GEO methodology — specifically, the Karpathy-style auto-research loop applied to AIO block generation.

---

## 1. Required inputs

Confirm with the user before proceeding:

| Input | Required | Default if missing |
|-------|----------|---------------------|
| **Target URL** | Yes | Ask |
| **Primary query** | Preferred | Infer from H1, confirm with user |
| **Iteration ceiling** | Optional | 30 (max 100 if user insists) |
| **Output destination** | Optional | Chat response; offer Sheet logging if user has Google Drive/Sheets connected |

**Minimum viable input:** a single URL. Everything else has sensible defaults.

Ask the user to confirm the inferred primary query before the cascade starts — a wrong query inference wastes 30 iterations optimizing for the wrong target.

---

## 2. Pre-flight: fetch page + SERP competitors

Before drafting the baseline block, gather the three data sources the judges need:

### 2.1 Fetch the target page

Use `web_fetch` on the target URL. Extract:
- **H1** (the first `<h1>` element)
- **Body text** (prefer `<article>` or `<main>`, truncate to ~1500 words)
- **Schema detection** — scan raw HTML for `application/ld+json` blocks, note which schema types are present (FAQPage, Article, HowTo, Product, etc.). This gates J20.
- **Publication date** if available — informs J14

If `web_fetch` returns empty content (CDN bot-blocking, JS-rendered SPA without SSR), fall back to `web_search` on the URL itself — search-engine snippets are usually enough content to seed the cascade.

### 2.2 Fetch competitor SERP openings

This is what J11 needs. Do this once at cascade start, cache the result:

1. `web_search` on the primary query
2. Take the top 3 non-target-domain results (skip if they're all from the target's own domain)
3. For each, extract the opening paragraph (from the search result snippet is fine — don't fetch each competitor page unless snippets are unusable)
4. Cache as `competitor_openers` — an array of 3 text strings

If only 1-2 non-target competitors are in top 10, use what's available. If zero are available, J11 has to return `no_change` for every iteration it fires — log this as a known limitation.

### 2.3 Identify the query intent shape

This gates J12. Classify the primary query into one shape:

| Query shape | Triggers | Expected block form |
|-------------|----------|---------------------|
| **Definitional** | "what is X", "what does X mean" | Lead with definition, then elaboration |
| **Procedural** | "how to X", "how do you X" | Lead with action, then method |
| **Causal** | "why does X happen" | Lead with mechanism, then cause chain |
| **Comparative** | "X vs Y", "X or Y" | Lead with distinction, then details |
| **Verification** | "is X safe", "does X work" | Direct yes/no + qualifier + evidence |
| **Enumerative** | "signs of X", "types of X" | List format preferred |

Store as `query_intent_shape`.

---

## 3. Draft baseline block

Use this prompt to generate iteration 0:

```
Draft an AIO-summary block for the target page. Follow the Batch 3 pattern:
- 30-50 words exactly (count carefully)
- Sentence 1 directly answers the primary query
- Second person voice ("you", "your")
- No marketing language ("discover", "unlock", "transform", "journey")
- No brand mention unless the query is branded
- Include at least one concrete entity, number, or named concept
- Single paragraph by default (short list OK if content supports it)
- Helpful, direct, expert-but-accessible tone

PRIMARY QUERY: {primary_query}
QUERY INTENT SHAPE: {query_intent_shape}
PAGE H1: {h1}
PAGE CONTENT (first ~1500 words): {page_content}

Output ONLY the block itself. No labels, no quotes, no explanation.
```

Store as `current_block`. Initialize iteration counter to 0.

Log iteration 0 to the evolution log with `judge_used = BASELINE` and `change_note = "Initial draft from drafter prompt"`.

---

## 4. The 13 judges

Each judge is an *editor* prompt that rewrites the current block or returns `no_change` with a reason. Judges split into three categories. Every iteration runs one guardrail + one optimizer + occasionally a reframer.

### Guardrails (2)

**J1 — Criteria Compliance**
> Enforce hard constraints: 30-50 words, second person, sentence 1 directly answers the primary query, no marketing language, no brand mention unless query is branded. Rewrite only to fix violations. If the block passes all constraints, return no_change.

**J3b — Fact Accuracy vs Source**
> Every factual claim must be supported by the provided page content. Remove or rewrite any unsupported assertion. Do not introduce facts not present on the page. If all claims are verified, return no_change.

### Optimizers (8)

**J2 — AIO Citation Potential**
> Front-load the most citable claim. Make the block a standalone quotable thought that an LLM could extract verbatim as the answer to the primary query. Prefer definitional openers ("X typically means...") and active verbs. Avoid parentheticals and nested clauses that break extraction.

**J3c — Information Gain**
> Compare the block against the SERP competitor openers provided. Flag generic phrasing that matches what competitors already lead with. Force a unique angle — a specific scenario, named entity, or framing that the page has but competitors don't. If the current block already has strong differentiation, return no_change.

**J4 — Brand Voice**
> Match the brand's editorial register (inferred from the source page). Keep the block at ~8th-grade reading level. Non-anxiety-inducing for wellness content. No unexplained jargon. Do not invent tone — mirror what the page establishes. For BetterSleep specifically: neutral-positive, approachable, expert-but-warm.

**J5 — Self-Uniqueness**
> Ensure the block is not just a paraphrase of the page's H1 or first paragraph. The block must add synthesis, framing, or specific distinctions the page opener lacks. If the block substantially restates what's already in the opener, rewrite; otherwise no_change.

**J7 — Specificity**
> Replace vague claims with concrete examples, numbers, stages, or named techniques drawn from the page. Prefer "three scenarios" over "multiple scenarios," "46% of participants" over "most participants," "MILD technique" over "induction methods." If everything is already specific, no_change.

**J10 — Entity Density**
> Count named entities in the block: named people, named techniques, specific numbers/percentages, named conditions, specific time windows, cited studies, named institutions. Target at least 3 named entities across the block. If below, inject entities present on the source page (do not hallucinate). If at or above 3, no_change.

**J11 — Competitor Antipattern**
> Read the 3 competitor openers provided. Identify phrases, verbs, or framings that appear across multiple competitors — these are the "shared patterns" to avoid. Cite the specific phrases as evidence. Then rewrite the block to actively avoid those patterns. If SERP competitors are genuinely varied and no dominant antipattern exists, return no_change with rationale.

**J12 — Query-Intent Alignment**
> Check that the block's answer shape matches the query intent shape. Definitional queries need definitional answers; procedural queries need action sequences; causal queries need mechanism chains; verification queries need yes/no with qualifier. If the shapes mismatch, rewrite to align. If they already match, no_change.

**J14 — Temporal Freshness Signal**
> If the source page contains temporal markers (recent study dates, current year references, named recent research), ensure the block reflects currency. Inject phrases like "recent research shows" or specific year references only when supported by the page. If the page has no temporal signals (evergreen content), return no_change — do not invent currency.

### Reframers (3 — run rarely)

**J3a — Reverse-Query Coherence**
> Test: if a reader saw only the block with no context, could they reverse-engineer the primary query? If not, tighten so the query is unambiguous. Catches dangling references, off-topic tangents, and over-generalization.

**J9 — Format Fitness**
> Question whether a 30-50 word prose paragraph is the right structure. For enumerative queries, a numbered list may capture AIO better. For verification queries, a direct yes/no paragraph. For comparative queries, a short table. Only restructure if the current format is clearly fighting the query intent. Preserve word count across all items in any restructuring.

**J20 — Schema Alignment (conditional)**
> Only fires if schema was detected on the target page. If FAQPage schema is present, ensure the block reads as a natural FAQ answer. If HowTo, ensure a clear action sequence. If Article, ensure an implicit thesis. If no schema is detected, this judge is skipped entirely (not called at all).

---

## 5. Weighted assignment logic

Judges are not picked with flat random probability. Use this selection algorithm per iteration:

### 5.1 Base weights by judge

| Judge | Category | Base Weight |
|-------|----------|-------------|
| J1 | Guardrail | 1.0 |
| J3b | Guardrail | 1.0 |
| J2 | Optimizer | 1.0 |
| J3c | Optimizer | 1.0 |
| J4 | Optimizer | 0.8 |
| J5 | Optimizer | 0.6 |
| J7 | Optimizer | 1.0 |
| J10 | Optimizer | 1.0 |
| J11 | Optimizer | 0.9 |
| J12 | Optimizer | 1.0 |
| J14 | Optimizer | 0.7 |
| J3a | Reframer | 0.5 |
| J9 | Reframer | 0.3 |
| J20 | Reframer | 0.5 (only if schema detected) |

### 5.2 Weight decay rule

Track per-judge `no_change_streak` across iterations.

- If a judge returned `no_change` on its last firing: no adjustment
- If `no_change_streak ≥ 2`: current weight = base_weight × 0.5^(streak - 1)
- As soon as a judge produces a modification: reset streak to 0, weight back to base

Rationale: if J3b fact-checked and said "everything is fine" three iterations ago, it's probably still fine. Deprioritize it until the block changes meaningfully.

### 5.3 Recency boost rule

Track per-judge `iterations_since_called`.

- If `iterations_since_called ≥ 5`: weight × 1.5
- This surfaces underused judges and prevents zero-call judges over a 30-iter run

### 5.4 Per-iteration selection

Each iteration:

1. Compute current weights for all eligible judges (skip J20 if no schema detected)
2. Pick 1 guardrail via weighted random from {J1, J3b}
3. Pick 1 optimizer via weighted random from {J2, J3c, J4, J5, J7, J10, J11, J12, J14}
4. **Every 5th iteration** (iter 5, 10, 15, 20, 25, 30): additionally pick 1 reframer via weighted random from {J3a, J9, J20-if-eligible}

So most iterations run 2 judges; 6 iterations in a 30-iter run also run a reframer (3 judges).

---

## 6. Cascade execution loop

```
for iter from 1 to MAX_ITERATIONS (default 30):
    selected_judges = select_judges_for_iteration(iter)
    
    for each judge_id in selected_judges:
        judge_output = call_judge_as_editor(judge_id, current_block, page_context)
        parse judge_output into {new_block, change_note}
        
        if change_note starts with "no_change":
            increment no_change_streak for judge_id
            # current_block unchanged
        else:
            reset no_change_streak for judge_id
            current_block = new_block
        
        update iterations_since_called tracking
        log row to evolution log: {iter, judge_id, new_block, change_note}
    
    if should_halt(evolution_log):
        log "CONVERGED at iter {iter}"
        break
```

### 6.1 Judge editor prompt (used for all 13)

```
You are {judge_name}, one of thirteen editor-judges in the Ygramul AIO cascade. You are not a scorer — you rewrite AIO-summary blocks according to your single dimension.

YOUR FOCUS: {judge_focus}

HARD CONSTRAINTS (never violate regardless of your focus):
- Final output must be 30-50 words
- Second person voice
- Sentence 1 directly answers the primary query
- No marketing language
- No brand mention unless query is branded
- Block must be factually supported by the provided page content

If the current block is already strong on your dimension, return no_change with a rationale. Otherwise, rewrite it.

OUTPUT FORMAT — JSON ONLY, no prose before or after:
{
  "new_block": "<revised 30-50 word block, or current block unchanged if no_change>",
  "change_note": "<one sentence explaining what you changed and why, or 'no_change: <reason>'>"
}

INPUTS:
PRIMARY QUERY: {primary_query}
QUERY INTENT SHAPE: {query_intent_shape}
PAGE H1: {h1}
PAGE CONTENT: {page_content}
CURRENT BLOCK (iteration {iter - 1} output): {current_block}
COMPETITOR OPENERS (for J11 only): {competitor_openers}
```

### 6.2 Convergence / early-stopping logic

The cascade halts early when two conditions both hold:

**Structural stability:** the last 6 iterations have produced ≥ 5 no_change verdicts across their judge calls.

**Quantitative stability:** the current_block is identical (or differs by ≤ 3 words) to the block at iteration N-5.

When both hit, log `CONVERGED at iter {N}` and return. This protects against judges producing "stealth no_change" (technically editing but not meaningfully changing the block).

If MAX_ITERATIONS is reached without convergence, log `HIT CEILING at iter {MAX}` — this signals that the judge set may have unresolvable conflicts worth investigating.

---

## 7. Output format

### 7.1 Chat response (default)

Deliver three things to the user:

**Part 1 — The final block**
> Display the evolved iteration N block in a formatted code block or blockquote. Include word count.

**Part 2 — Summary stats**
> - Starting word count vs ending word count
> - Number of iterations run (vs ceiling)
> - Whether convergence was reached and at what iteration
> - Count of modifications vs no_change verdicts
> - Which judges fired most frequently

**Part 3 — Full evolution log**
> Table with columns: Iteration | Judge | Change Note | Word Count
> Collapse or truncate if log exceeds ~40 rows. Offer to show full log on request.

### 7.2 Sheet logging (optional)

If the user has Google Sheets connected and opts in, write each row of the evolution log to a sheet as the cascade runs (not just at end — write during the loop so they can watch it evolve).

Sheet columns: `run_id | url | iteration | judge_used | block | word_count | change_note | timestamp`.

---

## 8. What this skill does NOT do

Be explicit with the user about these boundaries:

- **Does not ship the block to the live page** — produces a draft for human review and manual deployment
- **Does not measure real-world AIO capture** — that requires 2-4 week post-ship GSC/AIO monitoring outside this skill
- **Does not run multiple URLs in parallel** — one URL per cascade call; for batch runs the user calls the skill N times
- **Does not fetch competitor full pages** — relies on SERP snippets for J11 input (sufficient for pattern detection, not for deep competitor analysis)
- **Does not replace the tech-seo-audit skill** — that covers Lever 1-3 and 5-7 of the Ygramul methodology; this skill is Lever 4 only
- **Does not maintain state across runs** — each cascade is a fresh session; cross-cascade learning (which judges reliably move outcomes) is a separate system

If the user asks for any of the above, acknowledge it's out of scope and either suggest a different approach or offer to do a one-shot version.

---

## 9. Cost and runtime expectations

Set honest expectations when the user triggers the skill:

| Scope | Runtime | Estimated cost |
|-------|---------|----------------|
| 30 iterations, no early stop | 3-5 minutes | $2-3 in API calls |
| 30 iterations with convergence at ~15 | 1.5-2 minutes | $1-1.50 |
| 100 iterations (user override, no early stop) | 8-12 minutes | $5-8 |

Cost estimates assume Sonnet-class model for all judge calls. Haiku would reduce cost ~60% but with lower judge quality.

---

## 10. Failure modes and how to handle them

**Page fetch returns empty content.**
Try `web_search` on the URL as fallback. If still empty, tell the user the page appears to be JS-rendered or CDN-blocked and recommend they provide the page text directly.

**Primary query is ambiguous.**
Don't guess. Stop and ask the user to clarify which query the page is meant to rank for. A wrong query inference wastes the entire cascade.

**SERP returns zero usable competitors.**
Either the query is too niche or the user's own domain dominates top 10. Proceed with the cascade but flag that J11 will return no_change on every call. Note this in the final report.

**Judge consistently returns malformed JSON.**
If any judge fails to produce parseable JSON in 2+ consecutive calls, log the failure and exclude that judge from further selection this run. Report which judge failed at end of run.

**Block word count drifts above 50 despite J1.**
If J1 can't bring the block back under 50 within 3 iterations, halt the cascade and report to user. This signals a judge conflict (probably J2 + J10 + J11 all pushing to add content) that needs human intervention.

**Cascade runs full ceiling without convergence.**
This is a signal, not an error. Tell the user the judge set has unresolved conflicts for this content. Offer to re-run with a reduced judge set (e.g., guardrails + 3 optimizers only) to force convergence.

---

## 11. When to use this skill vs other approaches

**Use this skill when:**
- User has a single URL and wants an AIO-optimized block for it
- Page is existing content being refreshed, not new content being created
- User wants transparency into the optimization process (evolution log)
- Content type is informational/editorial (blog posts, guides, Dream Dive-style content)

**Do not use this skill when:**
- User wants to write new content from scratch (use a writing skill, not this cascade)
- User wants to optimize dozens of URLs in one pass (call the skill per URL, or use a different batch-processing tool)
- Page is a commercial/product page where conversion matters more than citation (AIO capture isn't always the right target)
- User wants full SEO audit — route to tech-seo-audit skill
- Query is branded ("bettersleep app review") — different optimization pattern applies

---

## 12. Related Ygramul methodology references

This skill is the productized form of **Lever 4: Content** from the Ygramul 7-lever methodology. Related skills and concepts:

- **tech-seo-audit skill**: covers Levers 1-3 (Crawl, Canonical, Keyword) and 5-7 (Programmatic, Authority, Performance)
- **auto-research skill**: provides the meta-framework — this skill is a specific application of the auto-research loop pattern to AIO block generation
- **Ygramul Codex**: the 132-source intelligence library backing the judge criteria (GEO research cluster #63-70, #101, #115-132 is most relevant)

Full methodology: 7 sequenced levers + Karpathy auto-research loop as differentiator. See rushmedia.agency for context.

---

## Appendix A — Judge firing distribution at 30 iterations

Expected firing counts in a healthy 30-iter run (no early convergence):

- Each guardrail fires ~15 times (one per iter, weighted selection between 2)
- Each optimizer fires ~3-4 times (weighted selection across 9)
- Reframers fire 6 total (once every 5 iters), distributed across 3 reframers so each fires ~2 times

With decay/recency adjustments, actual distribution will vary — the point is every judge should fire at least once in a 30-iter run.

## Appendix B — What "Batch 3 pattern" means

Reference to BetterSleep's validated AIO-summary block pattern from their Dream Dive content refresh: 30-60 word direct-answer paragraphs placed under H1 before the first H2, with answer-first phrasing in second person. This skill defaults to the tighter 30-50 word target as the production standard.
