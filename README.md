# TakeMeter — Magic: The Gathering Post Classifier

Classifies Magic: The Gathering Reddit posts into three categories: **Rules**, **Discussion**, and **Memes**.

---

## Community

I chose the Magic: The Gathering community because I recently started getting into the game and noticed how varied the online discourse is. MTG has anenormous Reddit presence and the posts fall into pretty distinct buckets; people asking about complex card interactions, people sharing memes and jokes about the game, and people having deeper conversations about strategy or their experience with the hobby. That variety made it a good fit for a classification task because the labels are genuinely different from each other and there's enough volume to collect examples without scraping obscure forums.

---

## Labels

- **Rules** — The post asks for clarification on the rules of the game or seeks an answer to a specific rule-related question. These posts usually describe a specific card interaction or game situation and end with a question.
- **Discussion** — The post involves an in-depth conversation about aspects of the game such as strategy, deck building, set reviews, or general opinions about the community or game design.
- **Memes** — The post is intended to be humorous through jokes, image captions, references to MTG culture, or relatable situations framed as comedy.

---

## Dataset

- **Source:** r/magicTCG and related MTG subreddits (Rules and Discussion examples collected from real posts; Memes examples are synthetic due to most real meme posts depending on images)
- **Total examples:** 200 (66 Rules, 66 Discussion, 68 Memes)
- **Split:** 70% train / 15% val / 15% test (stratified)

Real Reddit memes almost always rely on an image with a short caption so the image does most of the work. Since we're doing text classification and can't input images, I manufactured the Memes examples using the format `[Image: description] caption text` to simulate how a meme post reads when you describe what it contains. This was a practical compromise but it introduced a systematic formatting difference between Memes and the other two labels, which ended up affecting the model.

If a label was underrepresented during collection, the plan was to pull from additional MTG communities (Discord servers, MTG forums, Twitter/X). In practice the Rules and Discussion labels were easy to find and the Memes label needed to be filled synthetically.

---

## Models

- **Baseline:** Zero-shot classification using Claude with a prompt that included all three label definitions and instructed the model to output only the label name
- **Fine-tuned:** `distilbert-base-uncased` fine-tuned on the training split for 3 epochs

---

## Evaluation Report

### Overall Accuracy

| Model                       | Accuracy       |
| --------------------------- | -------------- |
| Baseline (zero-shot Claude) | 96.67% (29/30) |
| Fine-tuned DistilBERT       | 83.33% (25/30) |

Honestly, the fine-tuned model doing worse than the baseline was not what I expected. I thought fine-tuning on my own data would help but the baseline crushed it. I think the main reasons are that the training set was pretty small (~140 examples after the split), and DistilBERT is a much smaller model compared to Claude which already has deep language understanding built in. Claude can reason about what a post is trying to do; DistilBERT mostly picks up on surface patterns in the text.

---

### Per-Class Metrics — Fine-Tuned Model

| Label      | Precision | Recall | F1   |
| ---------- | --------- | ------ | ---- |
| Rules      | 0.90      | 0.90   | 0.90 |
| Discussion | 1.00      | 0.60   | 0.75 |
| Memes      | 0.71      | 1.00   | 0.83 |

The per-class metrics show the problem clearly. Discussion has perfect precision (when the model says Discussion it's right) but terrible recall it only catches 6 out of 10 actual Discussion posts and the other 4 get pulled into Memes. Meanwhile Memes has perfect recall but lower precision, meaning the model was calling too many things Memes that weren't.

Per-class metrics for the baseline are not available since the notebook only logged overall accuracy for that model.

---

### Confusion Matrix — Fine-Tuned Model

|                      | Predicted: Rules | Predicted: Discussion | Predicted: Memes |
| -------------------- | ---------------- | --------------------- | ---------------- |
| **True: Rules**      | 9                | 0                     | 1                |
| **True: Discussion** | 1                | 6                     | 3                |
| **True: Memes**      | 0                | 0                     | 10               |

The matrix makes the problem obvious. Memes was almost never misclassified, the model basically nailed that label. The real damage is in the Discussion row: 4 out of 10 Discussion posts were classified wrong, and 3 of those went to Memes. That one pair of labels (Discussion → Memes) accounts for most of the model's failures.

---

### Sample Classifications

These are example posts run through the fine-tuned model with their predicted label and confidence score.

| Post                                                                                                                                                                                                                                            | True Label | Predicted  | Confidence |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | ---------- | ---------- |
| "Can i tap attacking creatures to make them not attack?"                                                                                                                                                                                        | Rules      | Rules      | 0.97       |
| "[Image: Drake meme] No: Reading the card to explain the card. Yes: Just trusting your opponent when they say it works."                                                                                                                        | Memes      | Memes      | 0.99       |
| "Any thoughts on Thanos tech? I'm curious what everyone thinks about building this. I know blink should absolutely be included, but I can't help but wonder whether to build this as a voltron deck or fill it with creatures and protections." | Discussion | Discussion | 0.81       |
| "Every spell is better when you copy it"                                                                                                                                                                                                        | Discussion | Memes      | 0.78       |
| "[image of commander] So does this mean if I attack with 5 faeries I can goad 5 creatures?"                                                                                                                                                     | Rules      | Memes      | 0.91       |

The Drake meme prediction (0.99 confidence) makes a lot of sense — the post leads with a clear image description and uses the "No: / Yes:" meme format, which is one of the most recognizable structures in the Memes class. The model was probably very decisive because almost every Memes example in the training set followed a similar image-description-first format.

The Thanos deckbuilding post (0.81 confidence, correctly Discussion) shows the model working as intended, the post is longer, mentions specific card names and deck archetypes, and directly asks for community opinions, which are all features of genuine discussion posts.

The two wrong predictions at the bottom of the table connect directly to the issues in the error analysis below.

---

### Analysis of Wrong Predictions

The model made 5 errors total. The dominant pattern is **Discussion being called Memes** (3 cases), plus one **Discussion called Rules** and one **Rules called Memes**. I pasted the misclassified examples into Claude to look for common themes. It identified two patterns: posts that were very short with no contextual signals, and posts with image references. Both matched what I found when I went back and re-read the examples myself.

---

**Example 1 — True: Discussion, Predicted: Memes**

> _"Every spell is better when you copy it"_

This is a one-line opinion about a deckbuilding concept. It's Discussion because the person is making a strategic point, but it reads exactly like a meme caption. There's no question, no context, nothing that distinguishes it from text on a meme image. The model pattern-matched on the short, punchy format and flipped it to Memes.

The boundary issue here is real — I labeled several posts like this as Discussion when I probably should have been more strict. A one-liner observation with no follow-up question isn't really "in-depth conversation" by my own definition, but I kept classifying them as Discussion because they felt more like opinions than jokes. That inconsistency likely hurt the model.

---

**Example 2 — True: Discussion, Predicted: Memes**

> _"Why do Plains look dirty?"_

Same problem but even shorter. This is a question about card art and aesthetics, which I put in Discussion because it's a community conversation topic. But the model had no way to know that — it's 5 words, it's a playful observation, and it could easily be the caption under a meme pointing at a Plains card. The model made a reasonable mistake.

This one revealed something I hadn't thought about before: my Discussion examples ranged from multi-paragraph posts about Marvel set pricing all the way down to 5-word observations like this one. That inconsistency in length and depth made it hard for the model to learn what Discussion actually looks like.

---

**Example 3 — True: Rules, Predicted: Memes**

> _"[image of commander] So does this mean if I attack with 5 faeries I can goad 5 creatures?"_

This one is my fault. I labeled it Rules because the person is genuinely asking a rules question, but the text starts with `[image of commander]` — and every single Memes example in my dataset starts with `[Image: ...]`. The model learned that posts starting with an image description are Memes, which is a totally logical pattern to pick up given how I formatted the data. The one Rules post with an image reference was an outlier and there weren't enough examples like it for the model to learn differently.

This is a data problem more than a label definition problem. If I'd included 5-10 Rules posts that reference images, the model would have learned that image references aren't exclusive to Memes.

---

### What Would Need to Change

1. **More Discussion examples with consistent depth.** A lot of my Discussion posts are very short and casual. I should have been stricter. If a post is one sentence with no question and no context, it probably shouldn't be Discussion by my own definition. More examples that are clearly conversational and longer would help the model learn what actually separates Discussion from a short meme-like post.

2. **More Rules posts with image references.** One example wasn't enough. A model can't generalize from a single outlier. I'd need at least 5-10 Rules posts that include image references to break the `[image] = Memes` pattern.

3. **Either collect real Memes examples or be more careful with synthetic ones.** Manufacturing all 66 Memes examples with the `[Image: ...]` prefix format meant the model had an easy shortcut to classify Memes that didn't generalize well. Real meme posts from Reddit don't all follow that same structure.

4. **More training data overall.** ~140 training examples is very small for fine-tuning. The model didn't have enough examples to learn the subtler distinctions, especially at the Discussion/Memes boundary.

---

## What the Model Captured vs. What I Intended

This is worth thinking about separately from just the wrong predictions. The model learned some things that were different from what I was trying to teach it.

For **Rules**, what I intended was: posts seeking specific mechanical clarifications about how cards interact. What the model actually learned seems to be: posts with question marks that name multiple specific cards or game actions in the same sentence. That worked pretty well because those two things overlap a lot. Rules questions almost always name specific cards and end with a question mark.

For **Memes**, what I intended was: humorous posts relying on joke formats, image captions, or MTG cultural references. What the model actually learned was: posts that start with `[Image: ...]`. That worked perfectly on the test set (10/10 correct) but it's a surface-level trick. If a real meme post came in without that prefix format say, just a one-liner joke with no image tag the model would probably misclassify it as Discussion or Rules. The model found a formatting shortcut rather than actually learning what humor looks like.

For **Discussion**, what I intended was: in-depth conversations about strategy, opinions, and community topics. What the model actually learned seems to be: longer posts that don't start with `[Image: ...]` and don't have the structure of a direct rules question. That's sort of correct by elimination but it fails on short Discussion posts because those can look like meme captions. The model never learned what makes something a real discussion — it just learned what Discussion posts are not.

The bigger issue is that my label definitions assumed more signal in the text than actually exists for short posts. A 5-word sentence can be Discussion, a Meme caption, or even the start of a rules question. The model can't tell them apart with that little text, and honestly neither could I, I just made a judgment call when labeling.

---

## Spec Reflection

**One way the spec helped:** The planning document's hard edge cases section pushed me to define tie-breaking rules before I started annotating specifically for the Discussion/Memes boundary. That forced me to think about primary intent as the deciding factor, which kept my Rules and Memes labels fairly consistent. Without that section I probably would have been even more inconsistent on the borderline cases.

**One way the implementation diverged:** The spec assumed I'd collect real examples for all three labels. For Rules and Discussion that worked, but real Reddit memes are almost entirely image-based the text is usually just the caption, which doesn't work in isolation. So I manufactured all 66 Memes examples synthetically, using the `[Image: description] caption` format. This was the right call practically, but it introduced a systematic formatting difference that the model overfit to. Real meme posts in the wild don't follow that pattern as consistently, which means the model's Memes performance would likely drop significantly on data it hasn't seen before. If I were doing this again I'd either find a way to use real image post captions with their surrounding thread context, or be more careful about varying the format of the synthetic examples.

---

## AI Usage

### Instance 1 — Label stress-testing and boundary-case generation

Before annotating, I directed Claude to generate 9 posts that sit at the boundary between two labels. 3 for each pair (Rules/Discussion, Memes/Rules, Memes/Discussion). The prompt was: _"Generate 8 Magic: The Gathering Reddit post titles that a human annotator might struggle to classify as [Label A] vs. [Label B]. Each post should plausibly belong to either category. Do not label them."_

What it produced was a set of posts like _"I know deathtouch + trample means you only need to assign 1 damage to blockers — but does that actually make trample good in EDH or is it just a trap keyword?"_ (Rules/Discussion) and _"POV: you finally built a 'budget' deck [image of a $300 pile]. At what point do we admit budget is a myth in this game?"_ (Memes/Discussion). These were genuinely useful for pressure-testing the definitions. I added them to the dataset and used them to refine my tie-breaking rules, specifically for the Discussion/Memes boundary, where I landed on: if the informational content could stand alone as a discussion post, classify as Discussion; if the joke is the whole point, classify as Meme.

What I changed: Claude generated some posts that were easier to classify than I expected (not actually on the boundary), so I discarded about 3 of the 9 and kept the ones that genuinely gave me pause. I also adjusted the tie-breaking rule it implied. Claude tended to suggest "primary intent" as the deciding factor, which I kept, but I added a more specific test for the Memes/Discussion case rather than leaving it as a pure judgment call.

---

### Instance 2 — Synthetic Memes data generation

Because real Reddit meme posts are almost entirely image-based, I directed Claude to generate all 66 Memes examples synthetically. I provided the label definition, the `[Image: description] caption` format I wanted to use, and a list of common MTG meme formats (Drake meme, expanding brain, Gru's plan, etc.) and asked it to generate a diverse set of posts across those formats. I also asked it to include some posts near the Memes/Discussion boundary for the dataset.

What it produced was the full set of 66 examples, including posts like _"[Image: Gru's plan meme, 4 panels] Step 1: Play Cyclonic Rift. Step 2: Bounce everyone's board. Step 3: Win. Step 4: Get uninvited to game night."_ The outputs were mostly good and recognizable as memes. I reviewed each one before including it.

What I changed or overrode: I flagged 3 posts as boundary cases in the notes column (rows 145, 173, 182) because they used meme formats that doubled as real opinions Claude had generated them as straightforwardly Memes but I thought they were genuinely ambiguous. I also removed a few that felt too on-the-nose or repetitive (several Rhystic Study jokes in a row). In retrospect I should have varied the format more aggressively most examples used `[Image: ...]` as a consistent prefix, which the fine-tuned model learned as a shortcut for classifying Memes rather than learning what humor actually looks like.

---

### Instance 3 — Misclassification pattern analysis

After seeing the confusion matrix, I pasted the likely misclassified examples into Claude and asked it to identify common themes across the wrong predictions. The prompt was roughly: _"Here are examples a classifier predicted incorrectly. Identify any common patterns — similar post length, format, label pair confusion, etc."_

What it produced: two main patterns — (1) very short posts with no contextual signals, and (2) posts containing image references. Both were accurate observations that matched what I found when I went back and re-read the examples myself.

What I overrode: Claude also suggested that sarcasm might be a factor, and framed it as a possible explanation for some Discussion → Memes errors. When I re-read the specific examples, I didn't think sarcasm was actually the issue — the posts that got misclassified were just genuinely short and ambiguous, not sarcastic. I left sarcasm out of the analysis because I couldn't confirm it from the examples I had.

---

### Annotation disclosure

The Rules and Discussion examples were collected from real Reddit posts and labeled entirely by me. No AI tool was used to pre-label those examples. The Memes examples were fully AI-generated (see Instance 2 above) and reviewed by me before being included in the dataset. I did not use any AI tool to assign or suggest labels during annotation of the Rules or Discussion classes.

---

## Spec Reflection

1. Clone the repo and open `takemeter.ipynb` in Google Colab
2. Upload `data_template.csv` when prompted
3. Set `LABEL_MAP = {"Rules": 0, "Discussion": 1, "Memes": 2}` in the config cell
4. Run all cells in order
5. Evaluation results are saved to `evaluation_results.json` and `confusion_matrix.png`
