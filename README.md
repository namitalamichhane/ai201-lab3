# TakeMeter: r/stocks Discourse Classifier

A fine-tuned text classifier that evaluates post quality in r/stocks, 
distinguishing between analysis, speculation, and discussion.

---

## Community & Task

r/stocks is a Reddit community focused on stock market discussion. Discourse 
varies significantly in quality — some posts build structured arguments using 
earnings data and valuations, while others make confident predictions with no 
supporting evidence, or simply share news and ask questions. These distinctions 
matter to the community itself: members frequently call out "baseless" takes 
and praise well-reasoned DD posts.

---

## Label Taxonomy

### `analysis`
Builds an investment argument using specific verifiable evidence (P/E ratios, 
earnings figures, price comparisons) AND reasons from that evidence toward a 
conclusion. Both parts required.

**Example 1:** "$CHWY is trading at a historical low forward P/E, carries 
near-zero debt, and has been cheaper than Amazon on 50-60% of comparable items. 
I believe it's 30-45% undervalued and a likely acquisition target."

**Example 2:** "Lululemon's profit dropped 37% last quarter. The cause isn't 
brand weakness — it's the removal of the de minimis customs exemption, which 
eliminated a key cost advantage on cross-border shipments."

### `speculation`
A confident directional claim or prediction without verifiable supporting 
evidence. Asserts rather than argues.

**Example 1:** "GOOGL is the one stock to buy and hold for 10 years. It owns 
SpaceX, Anthropic, Waymo, and Android. Don't overcomplicate this."

**Example 2:** "Built on decades of cheap money, this market is heading toward 
a forced resolution. Cycles don't disappear — they accumulate excess and 
eventually resolve."

### `discussion`
A question, news share, advice request, or conversational post that makes no 
directional investment claim.

**Example 1:** "SK Hynix overtakes Samsung to become South Korea's most 
valuable company [Reuters repost]"

**Example 2:** "I pulled all my money out of index funds in 2025 and now 
can't bring myself to get back in at all-time highs. Any advice?"

---

## Data Collection

**Source:** r/stocks and r/SecurityAnalysis on Reddit, scraped using Python 
via Reddit's RSS feed and JSON endpoints. Reddit blocked Colab requests (403), 
so scraping ran locally and was supplemented with targeted searches for DD, 
valuation, and earnings posts.

**Labeling:** Posts pre-labeled in batches using Claude with explicit label 
definitions, then reviewed and corrected individually.

**Final distribution:**
| Label | Count | % |
|---|---|---|
| analysis | 75 | 34% |
| speculation | 82 | 38% |
| discussion | 61 | 28% |
| **Total** | **218** | |

**3 difficult-to-label examples:**

1. **Snapchat/Dotmo spinoff** (analysis vs discussion) — Used financial 
terminology in bullet points but cited no actual figures. Structured 
presentation made it look like analysis. Labeled `analysis`, but should be 
`discussion`. Rule: formatting without quantified figures = discussion.

2. **ACN "K-shaped market" post** (analysis vs speculation) — Cited P/E of 
13x and >60% YTD decline but drew no investment conclusion from those figures. 
Evidence present, reasoning step absent. Labeled `speculation`. Rule: evidence 
must argue toward a conclusion, not just be cited decoratively.

3. **"Portfolio suggestions long term"** (discussion vs speculation) — Felt 
like an advice request but made a confident directional claim that space, 
semiconductors, and batteries would outperform the S&P with no data. Labeled 
`speculation`. Rule: directional claim overrides conversational framing.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)  
**Hardware:** Google Colab T4 GPU  
**Split:** 70% train / 15% val / 15% test (152 / 33 / 33)

**Final training settings:**
- Epochs: 5
- Learning rate: 3e-5
- Batch size: 8
- Weight decay: 0.01
- Best model selected by eval loss

**Key hyperparameter decision:** Default settings (3 epochs, lr=2e-5, batch 
size=16) caused the model to collapse — it predicted only `speculation` for 
every input, achieving 42% accuracy. Reducing batch size to 8, increasing 
epochs to 5, and lowering learning rate to 3e-5 fixed this, bringing accuracy 
to 75.8%.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot)  
**Method:** Each test example passed to the model with label definitions and 
one example per label. Model instructed to output only the label name.

---

## Evaluation Report

### Overall Accuracy
| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq) | 60.6% |
| Fine-tuned DistilBERT | 75.8% |
| Improvement | +15.2pp |

### Per-Class Metrics

**Baseline (Groq zero-shot):**
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.67 | 0.67 | 0.67 | 12 |
| speculation | 1.00 | 0.25 | 0.40 | 12 |
| discussion | 0.50 | 1.00 | 0.67 | 9 |
| **macro avg** | **0.72** | **0.64** | **0.58** | 33 |

**Fine-tuned DistilBERT:**
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.71 | 0.83 | 0.77 | 12 |
| speculation | 0.82 | 0.75 | 0.78 | 12 |
| discussion | 0.75 | 0.67 | 0.71 | 9 |
| **macro avg** | **0.76** | **0.75** | **0.75** | 33 |

### Confusion Matrix (Fine-Tuned Model)
| | Pred: analysis | Pred: speculation | Pred: discussion |
|---|---|---|---|
| **True: analysis** | 10 | 2 | 0 |
| **True: speculation** | 0 | 9 | 3 |
| **True: discussion** | 2 | 1 | 6 |

### 3 Wrong Predictions

**1. analysis predicted as speculation**
> "Snapchat spins off Dotmo AI... Snap will provide zero direct funding, 
> with CTO Bobby Murphy acting as lead investor using personal capital."

True: `analysis` | Predicted: `speculation`

This is the Snapchat/Dotmo hard case from annotation. It uses structured 
bullet points and financial terminology but cites no specific figures. The 
model's speculation prediction is arguably more correct than my original 
label — this is a labeling error on my part, not a model failure.

---

**2. speculation predicted as discussion**
> "$MSFT is nowhere near a bottom. This stock is headed to $300 soon. 
> Copilot is a failure, OpenAI is sketchy... What's the bull case here?"

True: `speculation` | Predicted: `discussion`

The closing question ("What's the bull case here?") likely triggered the 
discussion prediction. The model hasn't learned that question framing can 
contain a strong directional claim. The confident price target and negative 
thesis are clear speculation signals the model missed.

---

**3. discussion predicted as analysis**
> "How long do you expect the AI gold rush to last? People are pouring money 
> into TSMC, SK Hynix, and Samsung..."

True: `discussion` | Predicted: `analysis`

The model keyed on specific company names and an article citation as analysis 
signals. But the post makes no investment argument — it's an open question 
about a trend. The model learned "named entities + external source = analysis" 
rather than "evidence argued toward a conclusion = analysis."

### Sample Classifications
| Post (truncated) | Predicted | Confidence | Correct? |
|---|---|---|---|
| "$CHWY near-zero debt, historical low P/E, 30-45% undervalued..." | analysis | 0.89 | ✓ |
| "GOOGL one stock to buy 10 years. Don't overcomplicate this." | speculation | 0.91 | ✓ |
| "Pulled money out in 2025, can't get back in at highs. Help?" | discussion | 0.84 | ✓ |
| "Snapchat spins off Dotmo AI, zero direct funding..." | speculation | 0.73 | ✗ (true: analysis) |
| "$MSFT headed to $300 soon... What's the bull case here?" | discussion | 0.67 | ✗ (true: speculation) |

The first prediction is reasonable: specific debt levels and a named valuation 
metric combined with a clear conclusion correctly signals analysis with high 
confidence.

### Reflection: What the Model Learned vs. What I Intended

I intended the model to learn reasoning structure — evidence-backed argument 
vs. confident assertion vs. neutral exchange. What it actually learned is a 
reasonable approximation with two gaps:

1. **Tone as proxy for label.** Assertive language → speculation. Correct 
often, but fails when analysis posts state conclusions confidently.

2. **Named entities as proxy for analysis.** Specific company names and 
citations → analysis. Fails when those appear in news reposts with no argument.

These gaps mirror my hardest annotation decisions — the same boundaries that 
confused me during labeling are the ones the model gets wrong.

---

## Spec Reflection

**How the spec helped:** Requiring a zero-shot baseline before fine-tuning 
gave the improvement real meaning. Without that 60.6% floor, the 75.8% result 
would be uninterpretable.

**How implementation diverged:** The spec assumes smooth data collection. 
Reddit blocked all Colab requests and most local requests with 403 errors. 
I had to combine RSS scraping, browser-saved JSON files, and searches across 
r/stocks and r/SecurityAnalysis to reach 218 examples — a significant effort 
the spec didn't anticipate.

---

## AI Usage

**1. Batch pre-labeling:** Provided Claude my label definitions and pasted 
batches of 20-30 posts asking for one label per post. Reviewed and corrected 
every label individually. Corrections were most common on the 
analysis/speculation boundary, where Claude over-labeled framework posts as 
analysis when no specific figures were present.

**2. Failure pattern analysis:** Pasted misclassified test examples into Claude 
and asked for common patterns. Claude identified tone and named entities as 
model proxies — verified by re-reading errors myself. Discarded Claude's 
suggestion that post length was a factor after finding no consistent pattern.

**3. Label stress-testing:** Before annotating, asked Claude to generate 
boundary posts between analysis and speculation. Several used real metrics 
but drew no conclusion — exactly the gap my ACN hard case later revealed. 
This prompted me to add "evidence must do argumentative work" to my analysis 
definition before labeling began.
