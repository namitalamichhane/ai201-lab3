# TakeMeter: Planning Document
**Community:** r/stocks  
**Date:** June 2026

---

## 1. Community

r/stocks is a Reddit community focused on stock market discussion, ranging from 
individual stock analysis to broad market commentary. The discourse varies 
significantly in quality — some posts build structured arguments using earnings 
data, valuations, and sector comparisons, while others make confident predictions 
with no supporting evidence, or simply share news and ask questions. This variety 
makes it well-suited for a classification task because the distinctions between 
post types are meaningful to the community itself (members frequently call out 
"baseless" takes or praise well-reasoned DD posts).

---

## 2. Label Taxonomy

### `analysis`
A post that builds an investment argument using specific, verifiable evidence 
(P/E ratios, earnings figures, price comparisons, ownership data) AND reasons 
from that evidence toward a conclusion. Both parts are required: evidence alone 
is not analysis, and a conclusion without evidence is not analysis.

**Example 1:**
> "$CHWY is trading at a historical low forward P/E, carries near-zero debt, 
> and has been cheaper than Amazon on 50-60% of comparable items. I believe 
> it's 30-45% undervalued and a likely acquisition target for a major retailer."

**Example 2:**
> "Lululemon sold more last quarter but profit dropped 37%. The cause isn't 
> brand weakness — it's the removal of the de minimis customs exemption, which 
> quietly eliminated a key cost advantage on cross-border shipments."

---

### `speculation`
A confident directional claim or prediction about a stock or market without 
verifiable supporting evidence. The post asserts rather than argues. May use 
financial language or frameworks but doesn't anchor claims to checkable data.

**Example 1:**
> "GOOGL is the one stock to buy and hold for 10 years. It owns a piece of 
> SpaceX, Anthropic, Waymo, and Android. Don't overcomplicate this."

**Example 2:**
> "Built on decades of cheap money and policy backstops, this market is heading 
> toward a forced resolution. Cycles don't disappear — they accumulate excess 
> and eventually resolve. And when they do, it's rarely orderly."

---

### `discussion`
A question, news share, advice request, or conversational post that makes no 
directional investment claim. Includes news reposts, personal finance situations, 
debate-openers, and requests for explanation.

**Example 1:**
> "SK Hynix overtakes Samsung to become South Korea's most valuable company 
> [Reuters article repost]"

**Example 2:**
> "I pulled all my money out of index funds in 2025 based on crash predictions 
> and now I can't bring myself to get back in at all-time highs. Any advice?"

---

## 3. Hard Edge Cases

### The framework-without-metrics problem
**The ambiguous post type:** Posts that describe a real investment thesis using 
general frameworks ("capital rotation," "sector strength," "undervalued relative 
to peers") but cite no specific verifiable metrics. These sound like analysis 
but lack checkable evidence.

**Real example from browsing:**
> "When I first discovered TSEM, it was in a post-rally pullback. Based on 
> earnings season dynamics and capital flow patterns, I started building a 
> position. My view was that capital would rotate into undervalued names that 
> hadn't yet moved. That was the setup I positioned for."

**Decision rule:** If I can fact-check the claims made, it's `analysis`. If the 
reasoning is a framework or narrative with no specific numbers attached, it's 
`speculation`. The test: would the argument collapse if I removed the numbers? 
If there are no numbers to remove, it's speculation.

### The news-with-a-take problem
**The ambiguous post type:** Posts that share a news article but add a brief 
opinion or prediction. Is it `discussion` (because it's primarily news) or 
`speculation` (because it adds a directional claim)?

**Decision rule:** Label by the dominant content. If the post is primarily 
sharing news and the opinion is a sentence or two of reaction, label it 
`discussion`. If the opinion is the main point and the news is cited as 
supporting context, label it `speculation`.

---

## 4. Data Collection Plan

**Source:** r/stocks on Reddit, collected manually by browsing posts sorted 
by Hot, Top (This Month), and Top (This Year).

**Target distribution:**
- `analysis`: 70 examples (35%)
- `speculation`: 65 examples (32.5%)
- `discussion`: 65 examples (32.5%)

**Where to find each label:**
- `analysis`: Search "DD" posts, sort by Top All Time, look for long-form posts 
  with specific metrics
- `speculation`: Hot feed, posts with strong directional claims, macro posts
- `discussion`: Daily discussion thread, advice request posts, news reposts

**If a label is underrepresented after 150 examples:** I will specifically 
search for that label type before continuing general collection. For `analysis` 
specifically, I will search r/stocks for "DD" or "due diligence" to find 
concentrated examples.

**Format:** Single CSV with columns: text, label, notes

---

## 5. Evaluation Metrics

**Primary metric:** Per-class F1 score for all three labels.

Accuracy alone is insufficient because my dataset may have moderate class 
imbalance (discussion tends to be overrepresented in natural r/stocks posts). 
A model that predicts `discussion` 60% of the time would get decent accuracy 
but would be useless.

Per-class F1 captures both precision (when the model predicts a label, is it 
right?) and recall (does the model catch all real examples of that label?). 
I need both — a model that only predicts `analysis` when it's extremely 
confident (high precision, low recall) would miss most analysis posts.

**Secondary metrics:**
- Confusion matrix to identify which label pairs are most confused
- Overall accuracy for comparison with the baseline

---

## 6. Definition of Success

The classifier is useful if all three of the following are true:
1. Overall accuracy exceeds the zero-shot baseline by at least 10 percentage points
2. Per-class F1 ≥ 0.65 for all three labels (no label is being systematically 
   ignored)
3. The analysis/speculation confusion rate is below 30% (this is the most 
   important boundary for the task)

A classifier that hits these thresholds could realistically be used to 
auto-tag r/stocks posts, helping readers filter for substantive content.

---

## 7. AI Tool Plan

### Label stress-testing
I will paste my three label definitions into Claude and ask it to generate 
8–10 posts that sit at the boundary between analysis and speculation — the 
hardest pair. If I can't classify them cleanly, I'll tighten the definitions 
before annotating 200 examples. I'll do this before starting data collection.

### Annotation assistance
I will use Claude to pre-label batches of 20–30 posts at a time by providing 
my label definitions and asking for one label per post. I will review and 
correct every pre-assigned label myself — I will not accept pre-labels without 
reading each post. I will track which examples were pre-labeled by adding 
"pre-labeled" in the notes column of my CSV.

### Failure analysis
After fine-tuning, I will paste my full list of misclassified test examples 
into Claude and ask it to identify common patterns (post length, vague language, 
sarcasm, specific label pairs). I will then verify each suggested pattern by 
re-reading the examples myself before including it in my evaluation report.
