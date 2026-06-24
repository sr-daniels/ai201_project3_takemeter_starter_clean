# r/socialskills Post Classifier

A fine-tuned DistilBERT text classifier trained to categorize Reddit posts from r/socialskills into seven labels based on the post's primary purpose. Built for AI201 Project 3.

---

## What This Project Does

r/socialskills is a public Reddit community where people post about social confidence, friendship, communication, anxiety, and everyday interpersonal situations. The posts vary widely in purpose — some ask for specific advice, some describe internal struggles, some share observations about social norms, and some are just venting. This project builds a classifier that automatically assigns one of seven labels to a post based on what the user is primarily trying to do.

The classifier was trained on 245 hand-labeled posts using DistilBERT fine-tuning, and compared against a zero-shot Llama 3.3 70B baseline via the Groq API.

---

## Labels

| Label | Description |
|---|---|
| `conversation_skills` | Asking how to talk to people, start or continue conversations, avoid awkward silence, or communicate better |
| `making_friends` | Wanting to find new friends, meet people, join groups, or turn acquaintances into friends |
| `self_perception` | Focused on self-image, confidence, insecurity, feeling unlikeable, or feeling socially behind |
| `social_observation` | Asking about social norms, interpreting others' behavior, or whether something is socially acceptable |
| `social_anxiety` | Describing fear, nervousness, avoidance, panic, or anxiety around social interaction |
| `conflict_and_boundaries` | Handling uncomfortable behavior, setting limits, confrontation, or dealing with rude or pushy people |
| `keeping_friends` | Maintaining existing friendships, drifting apart, reciprocity issues, or feeling excluded by current friends |

---

## Dataset

- **Source:** Public posts from r/socialskills collected via Reddit's API
- **Size:** 245 labeled examples
- **Split:** 70% train (171) / 15% validation (37) / 15% test (37), stratified by label
- **Annotation:** All examples labeled by hand. An LLM (Claude) was used to suggest initial labels on all 245 posts, which were then reviewed and corrected individually — see [AI Usage](#ai-usage) for details.

**Label distribution:**

| Label | Count | % |
|---|---|---|
| `conversation_skills` | 51 | 20.8% |
| `making_friends` | 44 | 18.0% |
| `self_perception` | 35 | 14.3% |
| `social_observation` | 33 | 13.5% |
| `social_anxiety` | 32 | 13.1% |
| `conflict_and_boundaries` | 29 | 11.8% |
| `keeping_friends` | 21 | 8.6% |

No label exceeded 70% of the dataset.

---

## Model

- **Architecture:** DistilBERT (`distilbert-base-uncased`) with a classification head
- **Training:** 3 epochs, learning rate 2e-5, batch size 16, weight decay 0.01, 50 warmup steps
- **Hardware:** T4 GPU on Google Colab
- **Baseline:** Zero-shot classification using `llama-3.3-70b-versatile` via Groq API

No hyperparameters were changed from the notebook defaults.

---

## Evaluation Results

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Llama 3.3 70B) | **0.703** |
| Fine-tuned DistilBERT | **0.297** |

Fine-tuning produced a **0.405 regression** relative to the baseline. The fine-tuned model performed substantially worse — this is a negative result and the analysis below explains why.

---

### Per-Class Metrics — Baseline (Llama 3.3 70B)

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `conversation_skills` | 0.78 | 0.88 | 0.82 | 8 |
| `making_friends` | 1.00 | 0.67 | 0.80 | 6 |
| `self_perception` | 0.62 | 0.83 | 0.71 | 6 |
| `social_observation` | 0.40 | 0.40 | 0.40 | 5 |
| `social_anxiety` | 0.67 | 0.40 | 0.50 | 5 |
| `conflict_and_boundaries` | 0.67 | 1.00 | 0.80 | 4 |
| `keeping_friends` | 1.00 | 0.67 | 0.80 | 3 |
| **macro avg** | **0.73** | **0.69** | **0.69** | 37 |

The baseline handled most labels reasonably well. Its weakest label was `social_observation` (F1 0.40), which is the label most dependent on recognizing the *absence* of a personal problem rather than the *presence* of specific vocabulary.

---

### Per-Class Metrics — Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `conversation_skills` | 0.25 | 0.75 | 0.38 | 8 |
| `making_friends` | 0.38 | 0.83 | 0.53 | 6 |
| `self_perception` | 0.00 | 0.00 | 0.00 | 6 |
| `social_observation` | 0.00 | 0.00 | 0.00 | 5 |
| `social_anxiety` | 0.00 | 0.00 | 0.00 | 5 |
| `conflict_and_boundaries` | 0.00 | 0.00 | 0.00 | 4 |
| `keeping_friends` | 0.00 | 0.00 | 0.00 | 3 |
| **macro avg** | **0.09** | **0.23** | **0.13** | 37 |

The fine-tuned model predicted exactly two labels across all 37 test examples: `conversation_skills` and `making_friends`. The remaining five labels have recall of 0.00 — the model never predicted them once.

---

### Confusion Matrix — Fine-Tuned DistilBERT (Test Set)

Rows = true label, columns = predicted label.

| True \ Predicted | conv_skills | making_friends | self_perception | social_obs | social_anxiety | conflict_bounds | keeping_friends |
|---|---|---|---|---|---|---|---|
| **conversation_skills** | **6** | 2 | 0 | 0 | 0 | 0 | 0 |
| **making_friends** | 1 | **5** | 0 | 0 | 0 | 0 | 0 |
| **self_perception** | 4 | 2 | **0** | 0 | 0 | 0 | 0 |
| **social_observation** | 5 | 0 | 0 | **0** | 0 | 0 | 0 |
| **social_anxiety** | 2 | 3 | 0 | 0 | **0** | 0 | 0 |
| **conflict_and_boundaries** | 3 | 1 | 0 | 0 | 0 | **0** | 0 |
| **keeping_friends** | 3 | 0 | 0 | 0 | 0 | 0 | **0** |

Every prediction lands in column 1 (`conversation_skills`) or column 2 (`making_friends`). Columns 3–7 are entirely zero.

---

## Error Analysis

The fine-tuned model got 26 of 37 test examples wrong (70% error rate). Before writing this analysis, the wrong predictions were reviewed with an LLM to surface patterns — see [AI Usage](#ai-usage) for how that was used and what was verified or discarded.

### The Core Failure: Label Space Collapse

This is not a collection of isolated mistakes. The model collapsed onto two labels and stopped distinguishing the other five. Every confidence score on the wrong predictions falls between 0.17 and 0.19 — just above the random-chance floor of 0.14 for a 7-class problem. The model is not confidently wrong; it is uncertain and defaulting to the two most surface-lexically-distinctive classes.

The most likely cause is data size. With 171 training examples split across 7 labels, the least-represented class (`keeping_friends`) had only 15 training examples. Fine-tuning a transformer classification head requires enough examples per label for the model to learn a stable decision boundary — 15–22 examples is generally not enough, and even the largest class (36 examples for `conversation_skills`) is thin. The two labels that survived — `conversation_skills` and `making_friends` — are the ones with the clearest, most consistently-worded surface vocabulary ("how do I talk," "how do I make friends"), making them the easiest for the model to latch onto.

### Wrong Prediction #1 — `self_perception` predicted as `conversation_skills`

> *"For years people have been annoyed by me but I NEVER know what I'm doing wrong. When I go to multiple discord servers people in those servers slowly start getting annoyed at me..."*

The true label is `self_perception`. The post is about a person who cannot identify their own social flaw despite a consistent pattern of rejection across many communities — the question is self-directed: *what is wrong with me?* The model predicted `conversation_skills`, most likely because the post mentions talking, online interaction, and getting responses from people. These are surface features that correlate with communication language.

**Why this boundary is hard:** `self_perception` and `conversation_skills` share a lot of surface vocabulary. Both involve social interaction, talking to people, and interpersonal outcomes. The difference is the locus of the question — *what should I say* (skills) vs. *what is wrong with me as a person* (perception). That distinction requires understanding the sentence's subject and purpose, not just its topic words. A model that learned mostly from word co-occurrence patterns would not reliably make this distinction, especially with limited training data.

**What would help:** More training examples that isolate the *self-directed* framing of `self_perception` posts, and possibly a tighter label definition that flags "what am I doing wrong" / "why don't people like me" phrasings as hard markers for this class.

### Wrong Prediction #2 — `social_observation` predicted as `conversation_skills`

> *"Anyone else have black and white thinking when it comes to liking/not liking people? Idk if I'm just paranoid, but any friend or family member that has talked shit about someone who's supposedly close..."*

The true label is `social_observation`. The post is a reflective question — the author notices a pattern in themselves and wonders if others share it. There is no crisis, no request for a technique, and no specific incident to resolve. The model predicted `conversation_skills`.

**Why this boundary is hard:** `social_observation` is defined by what is *absent* — no urgent personal problem, no request for a skill. Teaching a model to detect the absence of those features is harder than teaching it to detect their presence. All 5 `social_observation` examples in the test set were misclassified (4 → `conversation_skills`, 1 → `making_friends`). The model never learned this label at all. With 23 training examples that each rely on a subtle functional distinction rather than a specific vocabulary set, the class was essentially unlearnable at this scale.

**What would help:** Either more examples (50+) specifically for `social_observation`, or reconsidering whether this label is distinct enough to be worth keeping at this data size. The baseline (zero-shot Llama) only achieved F1 0.40 on this label even with the full label definition in the prompt, which suggests the boundary is genuinely difficult even for a strong model.

### Wrong Prediction #3 — `keeping_friends` predicted as `conversation_skills`

> *"hate being ghosted — anyone else hate being ghosted by those who promise that they'll never do that to you? I guess promises aren't shit nowadays"*

The true label is `keeping_friends`. The post is a short vent about being ghosted — a maintaining-friendships problem. The model predicted `conversation_skills`.

**Why this boundary is hard:** This post mentions communication (ghosting, promises) and has the informal register of a quick reaction post. There is no how-to structure, but short posts give the model very little signal to work with beyond individual words. All 3 `keeping_friends` examples in the test set were predicted as `conversation_skills`. With only 15 training examples for this label, the model never developed a representation for it.

**What would help:** More training examples for `keeping_friends` (aim for at least 40–50) and examples that specifically include short, venting-style posts so the model is not only trained on longer narrative ones.

---

## What the Model Captured vs. What It Was Designed to Capture

The model was designed to capture the *primary purpose* of a post — what the user is fundamentally trying to do. That distinction requires understanding post structure and the relationship between the user's problem and their question.

What the model actually learned was closer to: *does this post use vocabulary associated with talking/communication (→ `conversation_skills`) or vocabulary associated with friends/meeting people (→ `making_friends`)?* It captured topic keywords, not functional intent.

This is a known failure mode of fine-tuned classifiers on small datasets: the model overfits to the most common surface features of the majority classes and fails to develop representations for the nuanced distinctions the labels were designed to encode. The five labels that collapsed to zero recall all require recognizing something about the *structure or stance* of the post (self-directed vs. other-directed, observational vs. help-seeking, anxious vs. strategic) rather than just its topic domain. Those distinctions need significantly more examples and possibly longer texts to learn from than this dataset provided.

---

## Sample Classifications

Five test-set examples run through the fine-tuned model:

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| "How do I actually talk to people? I've spent the longest time pretending to be someone I'm not..." | `self_perception` | `conversation_skills` | 0.17 |
| "How do I make friends with people who will invite me to social events? I'm in high school..." | `making_friends` | `making_friends` ✓ | 0.19 |
| "Micro-habituation for Social Skills Improvement — I've been reading up on this concept of doing one thing a day that makes you feel uncomfortable..." | `social_observation` | `conversation_skills` | 0.18 |
| "How do I make hometown friends after being away for three years? For some context, I just graduated high school..." | `making_friends` | `making_friends` ✓ | 0.21 |
| "My ability to speak is declining. Any Tips? Started a new job in December and I've noticed my ability to form a sentence or articulate myself is becoming non-existent..." | `conversation_skills` | `conversation_skills` ✓ | 0.22 |

The third correct prediction ("My ability to speak is declining") is a good example of where the model is genuinely right for the right reason: the post explicitly asks for tips on articulation and verbal expression, which is the clearest case of `conversation_skills` in the dataset. The model's confidence of 0.22 is slightly above its typical range, which reflects the post's unambiguous structure.

---

## Spec Reflection

**One way the spec helped:** The spec's requirement to document hard edge cases in `planning.md` *before* annotating forced me to think through the label boundaries ahead of time. This discipline caught the `self_perception` vs. `conversation_skills` boundary early — I wrote a decision rule (if the post is about "what is wrong with me" rather than "what should I do differently," it goes to `self_perception`) before seeing the full dataset. That rule turned out to be the most common source of confusion in the actual wrong predictions, which suggests the early stress-testing was worth doing.

**One way the implementation diverged:** The spec's data collection plan called for four labels with roughly equal distribution, targeting ~60 examples each. During collection and annotation, the data naturally produced seven distinct patterns that couldn't be cleanly merged without losing important distinctions — for example, `making_friends` and `keeping_friends` are very different problems that kept appearing separately. I expanded to seven labels, which meant fewer examples per class than the original plan assumed. In hindsight, staying closer to four labels would have given the fine-tuned model a better chance of learning the distinctions, and the divergence is probably the biggest single contributor to the fine-tuning regression.

---

## AI Usage

**1. Pre-labeling the dataset**

All 245 posts were read in full and given to Claude (claude-sonnet-4-6) for initial label suggestions. The LLM was given the label definitions from `planning.md` and the full text of each post. Every suggestion was then reviewed individually — labels were corrected in approximately 20–30% of cases, usually at the `self_perception` / `conversation_skills` boundary and on short posts where the LLM defaulted to the most common class. The `ai_suggested_label`, `human_final_label`, and `ai_used_for_prelabelling` columns in the CSV track this workflow, though in this version the human review was conducted during the labeling session rather than as a separate pass.

**2. Error pattern analysis**

After generating the wrong predictions, they were pasted into Claude and used to surface common themes. The LLM identified two patterns: (a) most wrong predictions involved `conversation_skills` as the predicted class regardless of the true label, and (b) confidence scores were uniformly low across all errors. Both patterns were verified against the confusion matrix and the raw predictions before being included in this report. The LLM also suggested that short post length might be a factor — this was checked against the actual examples and found not to be primary; the "ghosted" post (#7) is short, but most other wrong predictions are medium-length. Short post length was not included as a finding because it could not be confirmed across the full set of errors.

---

## Files

| File | Description |
|---|---|
| `socialskills_labeled.csv` | 245 labeled examples with `text`, `label`, and `notes` columns |
| `planning.md` | Label definitions, edge case rules, data collection plan, annotation decisions |
| `confusion_matrix.png` | Confusion matrix for the fine-tuned model on the test set |
| `evaluation_results.json` | Accuracy and metadata for both models |
| `README.md` | This file |
