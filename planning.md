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

### Case 1: Structured presentation without real numbers
**Post type:** Snapchat/Dotmo spinoff analysis with bullet points  
**Labels it sat between:** `analysis` vs `discussion`

The post used financial terminology (zero-funding structure, equity retention, 
SBC removal, margin impact) arranged as a chain of cause-and-effect reasoning, 
which made it look like analysis. But no actual figures were cited — no cost 
savings quantified, no margin percentages named. The structured presentation 
fooled me into seeing reasoning that wasn't fully there.

**Note:** This case was mislabeled in the final dataset as `analysis` 
and was corrected to `discussion` after reflection.

**Decision:** Labeled `analysis`, but on reflection should be `discussion`. 

**Decision rule going forward:** Structured formatting and financial terminology 
alone do not qualify a post as analysis. There must be at least one specific, 
quantified, verifiable figure doing real argumentative work.

---

### Case 2: Evidence present, reasoning step absent
**Post type:** ACN "K-shaped market" macro post  
**Labels it sat between:** `analysis` vs `speculation`

The post cited ACN's P/E of 13x and a >60% YTD decline — real, checkable 
numbers that pulled toward analysis. But the author explicitly disclaimed 
investing in the stock, drew no investment conclusion from those figures, and 
used them only to float a vague "K-shaped market" narrative. The evidence was 
present but the reasoning step — using that evidence to argue toward a 
conclusion — was absent.

**Decision:** Labeled `speculation`.

**Decision rule going forward:** Both parts of the analysis definition must be 
present — specific verifiable evidence AND a conclusion reasoned from that 
evidence. Evidence used decoratively to support a vague narrative does not 
qualify.

---

### Case 3: Advice request with a directional claim embedded
**Post type:** "Portfolio suggestions long term" advice request  
**Labels it sat between:** `discussion` vs `speculation`

The post felt like a discussion (asking for advice, personal situation), but 
the author made a confident directional claim that space, semiconductors, and 
batteries would outperform the S&P without citing any supporting data. The 
claim tipped it past neutral discussion into an unverifiable directional 
assertion.

**Decision:** Labeled `speculation`. This is the thinnest of the three calls 
and reasonable to argue either way.

**Decision rule going forward:** If a post is primarily an advice request but 
contains a confident directional claim about future performance with no 
supporting evidence, label by the directional claim — `speculation` — not by 
the conversational framing.

---

## 4. Data Collection Plan

**Actual collection method:** Scraped using Reddit's RSS feed and JSON 
endpoints via Python locally, after Reddit's API blocked Colab requests 
with 403 errors. Supplemented with targeted searches for DD, valuation, 
and earnings posts from r/stocks and r/SecurityAnalysis.

**Final distribution achieved:**
- `analysis`: 75 examples (34%)
- `speculation`: 82 examples (38%)
- `discussion`: 61 examples (28%)
- **Total: 218 examples**
  
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

### Annotation assistance
Used Claude to pre-label batches of 20–30 posts at a time. Reviewed and 
corrected every pre-assigned label individually. Pre-labeled examples are 
tracked with "pre-labeled" in the notes column.
