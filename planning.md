# Planning: r/socialskills Post Classification

## Community Choice

I chose **r/socialskills**, a public Reddit community where users discuss social confidence, awkward interactions, friendship, workplace communication, loneliness, self-esteem, and everyday social norms. This community is a good fit for a classification task because the posts are text-heavy and vary widely in purpose and quality: some users ask about one specific incident, some describe internal struggles, some ask for general techniques, and some critique the community itself.

This variation makes the discourse interesting because a regular community member would likely judge posts differently depending on whether they are actionable, emotionally supportive, overly vague, too situation-specific, or useful as general social-skills guidance.

## Labels

### 1. Specific Situation Advice Request

A post belongs in this label when the user describes a concrete social situation and asks what they should do, say, or think about it.

Example posts:
- A user accidentally tells their manager “you look tired” and asks whether it was rude or how to recover.
- A user says someone at church is making them uncomfortable and asks how to handle future interactions.

### 2. Internal Struggle / Self-Perception

A post belongs in this label when the main focus is the user’s insecurity, anxiety, loneliness, self-esteem, or feeling socially abnormal.

Example posts:
- A user asks how to raise their self-esteem because they feel socially behind.
- A user asks why everyone else seems “normal” while they feel awkward or disconnected.

### 3. Skill-Building Advice / Guide

A post belongs in this label when it gives or requests generalizable tips, methods, or strategies for improving social skills.

Example posts:
- A user shares a long list of social tips and tricks.
- A user asks for rare social advice that actually helped people become better at socializing.

### 4. Community Meta / Quality Judgment

A post belongs in this label when the user discusses the subreddit itself, the quality of advice, or what kinds of posts belong in the community.

Example posts:
- A user asks whether there is a better alternative to r/socialskills because the subreddit has too many validation posts.
- A user complains that the community focuses too much on personal reassurance instead of general guides or explanations.

## Hard Edge Cases

The hardest anticipated edge case is a post that combines an internal struggle with a request for practical advice. For example, a post like “How do I stop being awkward and become a people person again?” could fit both **Internal Struggle / Self-Perception** and **Skill-Building Advice / Guide**.

I handle this by labeling based on the post’s primary purpose:
- If the post mostly describes emotions, insecurity, loneliness, or identity, I label it **Internal Struggle / Self-Perception**.
- If the post mostly asks for reusable steps, techniques, or strategies, I label it **Skill-Building Advice / Guide**.
- If the post describes one specific real-life incident, even if anxiety is involved, I label it **Specific Situation Advice Request**.
- If the post mainly evaluates the subreddit or the quality of advice, I label it **Community Meta / Quality Judgment**.

The three cases below are real examples from the annotation process that required genuine deliberation.

### Edge Case 1 — “How to not come across as too much” (`self_perception` vs `conversation_skills`)

The post describes a naturally bubbly person who started two new jobs and notices coworkers ignoring or seeming to dislike her. She asks what she can try to stop coming across so strong. On the surface this looks like a conversation skills question — she is asking what to do differently. But the body of the post is dominated by distress about being perceived as annoying, going home crying, and wondering why people seem to hate her. The practical question is thin; the emotional crisis is the weight of the post.

**Decision:** `self_perception`. When the practical request is a thin frame around a deeper anxiety about how others see you, the emotional content determines the label.

### Edge Case 2 — “Distancing from friends, for theirs and my better good.” (`social_anxiety` vs `keeping_friends`)

The post describes a pattern where the author befriends someone, then overthinking kicks in — convinced she is not good enough, that her friend secretly hates her, that she will cause jealousy — and she pulls away. She names this a recurring pattern and asks what it is called and how to understand it. This is about friendship in the sense that friendships keep ending, but the question is not “how do I keep friends” — it is “what is this anxious spiral and why do I do it?”

**Decision:** `social_anxiety`. When the friendship problem is entirely downstream of an internal anxiety mechanism, and the user is asking about the mechanism rather than the friendship strategy, the label goes to the mechanism.

### Edge Case 3 — “What am I doing wrong to deserve this treatment interpersonally?” (`self_perception` vs `conflict_and_boundaries`)

The post describes an isolated woman in her late 20s whose longtime friend casually said something dismissive and hurtful to her. She recounts the exchange and asks what she is doing wrong to receive this kind of treatment. A conflict is present and a specific person behaved badly, but the user is not asking how to confront the friend or set a limit — she is asking whether something about her is causing people to treat her this way.

**Decision:** `self_perception`. The locus of inquiry is inward. Posts that describe a conflict but ask “what is wrong with me?” rather than “how do I handle this?” belong in `self_perception`, not `conflict_and_boundaries`.

## Data Collection Plan

I will collect public posts from r/socialskills. I will start with recent posts and, if needed, include top posts from the past year or all time to get enough examples across all labels. I will collect at least **200 total examples**.

Target distribution:
- Specific Situation Advice Request: about 60 examples
- Internal Struggle / Self-Perception: about 60 examples
- Skill-Building Advice / Guide: about 60 examples
- Community Meta / Quality Judgment: about 20 examples

I expect Community Meta / Quality Judgment to be underrepresented, so I will not force an equal label distribution if that would make the dataset less realistic. If that label has too few examples after 200 posts, I will either collect more posts specifically using subreddit search terms like “subreddit,” “advice,” “quality,” and “alternative,” or merge it into another label if there are not enough real examples to support it.

Each collected example will include the post title, body text when available, URL or ID, date collected, and final human-assigned label.

## Evaluation Metrics

I will use more than accuracy because this is a multi-class classification task with likely class imbalance. Accuracy could look good even if the model performs poorly on smaller labels like Community Meta / Quality Judgment.

I will use:
- **Accuracy** to measure overall correctness.
- **Per-label precision** to see whether the model overuses any label.
- **Per-label recall** to see whether the model misses important examples from a label.
- **Macro F1 score** because it treats each label equally, which matters when some labels are less common.
- **Confusion matrix** to identify which labels are most often mixed up, especially Internal Struggle vs Skill-Building Advice.

These metrics are appropriate because the goal is not just to classify most posts correctly, but to understand whether the classifier can distinguish between different kinds of social-skills discourse in a useful way.

## Definition of Success

A genuinely useful classifier should correctly identify the main purpose of most r/socialskills posts while making relatively few serious boundary mistakes. I would consider the model successful if it reaches:

- At least **75% overall accuracy**
- At least **0.70 macro F1**
- At least **0.65 recall for each major label**
- No repeated severe confusion between **Internal Struggle / Self-Perception** and **Skill-Building Advice / Guide**

For deployment in a real community tool, I would accept “good enough” performance only if moderators or researchers could use the classifier to sort posts for review without trusting it blindly. The classifier would be useful as an organizing tool, not as an automatic moderation system. If the classifier frequently mislabels vulnerable personal posts as low-quality or generic advice posts, it would not be acceptable for real deployment.

## AI Tool Plan

### Label Stress-Testing

Before annotating 200 examples, I will give an AI tool my four label definitions and my edge-case rules. I will ask it to generate 5–10 realistic r/socialskills-style posts that sit near the boundary between two labels, especially:

- Internal Struggle vs Skill-Building Advice
- Specific Situation Advice Request vs Internal Struggle
- Specific Situation Advice Request vs Skill-Building Advice

If I cannot classify the generated boundary examples consistently, I will revise the label definitions before starting annotation. The goal is to discover unclear boundaries early instead of finding out after labeling the full dataset.

### Annotation Assistance

I may use an LLM to pre-label batches of posts before reviewing them myself. If I do, I will treat the LLM labels as suggestions only, not as final labels. I will track this in the dataset with a column such as:

- `ai_suggested_label`
- `human_final_label`
- `ai_used_for_prelabelling`

This will make the AI assistance clear for disclosure in the project’s AI usage section. I will personally review every label before it enters the final dataset.

### Failure Analysis

After model evaluation, I will give an AI tool a list of wrong predictions and ask it to identify patterns in the errors. I will look for patterns such as:

- The model confusing emotional posts with advice-seeking posts
- The model overpredicting the largest label
- The model failing on short posts with little context
- The model struggling with posts that contain both a story and a general question

I will verify any AI-suggested error patterns myself by checking the original posts and the confusion matrix. I will not include a pattern in the final write-up unless I can confirm it with actual examples from the evaluation results.

## Stretch Feature Note

If I add stretch features later, I will update this planning document before implementing them. Possible stretch features could include testing whether using both title and body text improves results, comparing a simple baseline model against an LLM-based classifier, or adding a confidence score for uncertain examples.
