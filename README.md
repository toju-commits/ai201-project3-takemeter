# TakeMeter: Classifying Generative AI Creator Discourse

## Overview

TakeMeter is a text classification project for AI201 Project 3. The goal is to classify public r/StableDiffusion posts and comments into discourse categories that reflect how people participate in a generative AI creator community.

This project focuses on the difference between reusable creator knowledge, technical troubleshooting, model/news discussion, and low-context showcase or reaction content. The motivation is practical: AI creator communities contain a lot of useful workflow knowledge, but that signal is mixed with hype, memes, brief reactions, and ambiguous posts. A classifier like this could help surface useful workflows, identify technical pain points, and track discussion around new models or tooling.

## Community

The dataset uses public examples from r/StableDiffusion, a Reddit community focused on Stable Diffusion, ComfyUI, open-source/local AI image generation, model releases, workflows, and related creator tools.

I chose this community because it connects directly to generative AI creator workflows and AI content creation. The community has several recurring types of discourse: people share workflows, ask for technical help, discuss new models and licensing, and post showcases or reactions. That made it a good fit for TakeMeter because the community naturally contains several different discourse roles instead of only one type of post.

## Labels

I used four mutually exclusive labels.

| Label | Definition |
|---|---|
| `workflow_resource` | The text shares a repeatable creator process, prompt strategy, model setup, ComfyUI workflow, LoRA, checkpoint, tutorial, benchmark, tool, resource, or practical method that another creator could use, reproduce, download, or adapt. |
| `technical_help` | The text asks for help, troubleshooting, recommendations, settings, installation support, hardware guidance, error diagnosis, or explains a technical problem. |
| `model_news_discussion` | The text discusses model releases, tool updates, model comparisons, licensing, commercial use, platform rules, copyright, AI art debates, or broader community/tooling implications. |
| `showcase_reaction` | The text is mainly an image/video showcase, hype, joke, meme, praise, vague criticism, emotional reaction, or low-detail comment without a clear reusable workflow, technical problem, or substantive model/news discussion. |

## Label Examples

| Label | Example 1 | Example 2 |
|---|---|---|
| `workflow_resource` | "I made a visual Ideogram 4 prompt editor with JSON generation by LLM and vision." | "Subject first, then style, then scene, with lighting and camera last." |
| `technical_help` | "What models and workflows can generate an image in 5s?" | "ComfyUI keeps failing when a checkpoint loads; is this VRAM or a missing custom node?" |
| `model_news_discussion` | "Ideogram 4 is a great model, but the license is very restrictive." | "Replies to the Monet post were confidently wrong about whether it was AI." |
| `showcase_reaction` | "It is still nuts to me how realistic AI is getting." | "Anima-Base is magic and I don't think people realize how good it is." |

## Dataset

The dataset is stored in `data/labeled_data.csv`.

It contains 200 total examples with two required columns:

```text
text,label
```

The label distribution is balanced:

| Label | Count |
|---|---:|
| `workflow_resource` | 50 |
| `technical_help` | 50 |
| `model_news_discussion` | 50 |
| `showcase_reaction` | 50 |

I intentionally kept the classes balanced so that the model would not learn to predict only the majority class. No single label accounts for more than 25% of the dataset.

## Data Collection and Annotation Process

I collected public text examples from r/StableDiffusion posts and comments. I did not include usernames, private messages, or personal metadata. Each row was labeled according to the main purpose of the text.

The hardest part was deciding how to label short or title-like examples. Many posts in AI creator communities are short, and a title such as "Ideogram 4 Turbo LoRA Released" can look like both a resource and model news. To handle this, I used decision rules:

- If the text gives reusable setup or workflow information, label it `workflow_resource`.
- If the text asks how to solve or configure something, label it `technical_help`.
- If the text is mainly about a model/tool release, license, commercial use, or broader community issue, label it `model_news_discussion`.
- If the text is mostly a reaction, showcase, joke, or hype statement, label it `showcase_reaction`.

## Difficult Labeling Decisions

| Text | Final Label | Why This Was Difficult | Decision |
|---|---|---|---|
| "I'm looking for local models and workflows that can generate images in under five seconds on average CPU." | `technical_help` | It contains the words "models" and "workflows," which sound like resource-sharing language. | I labeled it `technical_help` because the intent is asking for recommendations, not providing a reusable workflow. |
| "Big update to the LTX Trainer: One framework, many conditioning modes." | `model_news_discussion` | It could be treated as a resource because it names a tool/framework. | I labeled it `model_news_discussion` because the main purpose is announcing/discussing a tool update rather than teaching a specific reusable workflow. |
| "Subject first, then style, then scene, with lighting and camera last." | `workflow_resource` | It is short and does not explicitly say "tutorial" or "workflow." | I labeled it `workflow_resource` because it gives a repeatable prompt-ordering rule that another creator can apply. |
| "Ideogram 4 can produce great stuff sometimes." | `showcase_reaction` | It names a specific model, which could make it look like model discussion. | I labeled it `showcase_reaction` because it is mostly a vague reaction and does not discuss release details, licensing, or broader implications. |

## Model and Training Setup

The fine-tuned model used was:

```text
distilbert-base-uncased
```

The dataset was split approximately as:

- 70% training
- 15% validation
- 15% test

The final test set contained 30 examples.

The training setup used the starter Colab notebook with the following hyperparameters:

| Hyperparameter | Value |
|---|---:|
| Epochs | 3 |
| Learning rate | 2e-5 |
| Train batch size | 16 |
| Eval batch size | 32 |
| Weight decay | 0.01 |
| Model | `distilbert-base-uncased` |

I kept the default 3 epochs, learning rate 2e-5, and batch size 16 because the dataset was small. My goal was to run a stable fine-tuning experiment without over-tuning hyperparameters to a tiny validation set. Three epochs is enough for the model to adapt to the labels, but not so many that I would intentionally overfit the 200 examples.

## Baseline Model

The baseline was a zero-shot Groq/Llama classifier. The prompt included the community description, the four label definitions, examples for each label, and decision rules. The model was instructed to return only one valid label name.

The baseline was evaluated on the same 30-example test set as the fine-tuned DistilBERT model. I collected the baseline results by running the same test examples through the prompt-based classifier and parsing the returned label names.

The baseline prompt used this structure:

```text
You are classifying posts/comments from r/StableDiffusion.
Assign each post to exactly one of the following four labels:
workflow_resource, technical_help, model_news_discussion, showcase_reaction.
[Definitions and examples for each label]
Decision rules:
- If the text gives reusable process or resource details, choose workflow_resource.
- If the text asks how to fix or do something technical, choose technical_help.
- If the text is mainly about model releases, licensing, commercial use, platform rules, or broader AI community debate, choose model_news_discussion.
- If the text is mostly a reaction, showcase, joke, or vague hype, choose showcase_reaction.
Respond with ONLY one label name.
```

## Results

| Model | Accuracy |
|---|---:|
| Zero-shot baseline (Groq/Llama) | 0.7333 |
| Fine-tuned DistilBERT | 0.3333 |

Fine-tuning did not improve performance. It caused a regression of 0.4000 accuracy compared with the zero-shot baseline.

The exported metrics are in `evaluation_results.json`.

## Fine-Tuned Model Per-Class Metrics

| Label | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| `workflow_resource` | 0.22 | 0.25 | 0.24 | 8 |
| `technical_help` | 0.29 | 0.71 | 0.42 | 7 |
| `model_news_discussion` | 0.00 | 0.00 | 0.00 | 8 |
| `showcase_reaction` | 0.75 | 0.43 | 0.55 | 7 |
| **Accuracy** |  |  | **0.33** | 30 |
| **Macro avg** | 0.32 | 0.35 | 0.30 | 30 |
| **Weighted avg** | 0.30 | 0.33 | 0.29 | 30 |

## Baseline Per-Class Metrics

| Label | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| `workflow_resource` | 0.78 | 0.88 | 0.82 | 8 |
| `technical_help` | 0.70 | 1.00 | 0.82 | 7 |
| `model_news_discussion` | 1.00 | 0.38 | 0.55 | 8 |
| `showcase_reaction` | 0.62 | 0.71 | 0.67 | 7 |
| **Accuracy** |  |  | **0.73** | 30 |
| **Macro avg** | 0.78 | 0.74 | 0.71 | 30 |
| **Weighted avg** | 0.78 | 0.73 | 0.71 | 30 |

## Confusion Matrix

The confusion matrix below shows the fine-tuned DistilBERT model's predictions on the test set.

![Fine-tuned model confusion matrix](results/confusion_matrix.svg)

The most important pattern is that the fine-tuned model never correctly predicted `model_news_discussion`. It also over-predicted `technical_help`, especially for true `workflow_resource` and `model_news_discussion` examples.

## Sample Classifications from the Fine-Tuned Model

These examples are written out as text so the predicted label and confidence are visible without relying on screenshots.

| Text | True Label | Fine-Tuned Prediction | Confidence | Result | Explanation |
|---|---|---|---:|---|---|
| "ComfyUI keeps failing when a checkpoint loads; is this VRAM or a missing custom node?" | `technical_help` | `technical_help` | 0.28 | Correct | This is reasonable because the post directly asks for troubleshooting help around a checkpoint error. |
| "Anima-Base is magic and I don't think people realize how good it is." | `showcase_reaction` | `showcase_reaction` | 0.27 | Correct | This is mostly a hype/reaction post, not a workflow or help request. |
| "I made a visual Ideogram 4 prompt editor with JSON generation by LLM and vision." | `workflow_resource` | `technical_help` | 0.27 | Incorrect | The model likely focused on technical words instead of recognizing the post as a shared tool/resource. |
| "Replies to the Monet post were confidently wrong about whether it was AI." | `model_news_discussion` | `workflow_resource` | 0.26 | Incorrect | This is broader AI art discourse, but it lacks clear model-news keywords. |
| "Subject first, then style, then scene, with lighting and camera last." | `workflow_resource` | `showcase_reaction` | 0.27 | Incorrect | The post gives a reusable prompting rule, but it is short and lacks obvious workflow markers. |

## Error Analysis

The fine-tuned DistilBERT model made 20 incorrect predictions out of 30 test examples. Many errors came from short examples where the text did not contain enough context for a small fine-tuned model to infer the label boundary.

### Failure 1

Text:

```text
I made a visual Ideogram 4 prompt editor with JSON generation by LLM and vision.
```

True label: `workflow_resource`  
Predicted label: `technical_help`  
Confidence: 0.27

Analysis: The example describes a tool/resource, but the model likely focused on technical words like "prompt editor," "JSON," and "LLM," which often appear in help requests. The model did not learn the difference between sharing a tool and asking for help with a tool.

### Failure 2

Text:

```text
I'm looking for local models and workflows that can generate images in under five seconds on average CPU.
```

True label: `technical_help`  
Predicted label: `workflow_resource`  
Confidence: 0.27

Analysis: The text asks for a recommendation, so it should be `technical_help`. The model likely focused on the words "models" and "workflows," which are common in resource-sharing examples.

### Failure 3

Text:

```text
Replies to the Monet post were confidently wrong about whether it was AI.
```

True label: `model_news_discussion`  
Predicted label: `workflow_resource`  
Confidence: 0.26

Analysis: This is a broader AI art/community discussion, but it lacks obvious keywords like "license," "release," or "model." The model had trouble recognizing community discourse when it did not contain explicit model-news vocabulary.

### Failure 4

Text:

```text
Ideogram 4 can produce great stuff sometimes.
```

True label: `showcase_reaction`  
Predicted label: `workflow_resource`  
Confidence: 0.27

Analysis: The text is a vague reaction to model output. The model may have over-associated the model name "Ideogram 4" with workflow/resource content.

### Failure 5

Text:

```text
Subject first, then style, then scene, with lighting and camera last.
```

True label: `workflow_resource`  
Predicted label: `showcase_reaction`  
Confidence: 0.27

Analysis: This is a practical prompt-ordering rule, so it is a workflow/resource example. However, because it is short and lacks explicit words like "workflow," "tutorial," or "settings," the model treated it as a low-context reaction.

## Error Pattern Analysis

The systematic error pattern was not simply "the model is bad." The model confused labels that share the same surface vocabulary. The most consistent boundary problems were:

1. **`workflow_resource` vs. `technical_help`**: Posts with words like "workflow," "model," "prompt," or "tool" were hard to classify because those words appear in both shared resources and help requests.
2. **`model_news_discussion` vs. `workflow_resource`**: Posts about model/tool updates were often pushed into resource/help categories because they named specific models or tools.
3. **Short low-context examples**: Very short titles or comments did not provide enough structure for DistilBERT to infer intent.

The strongest supporting evidence is `model_news_discussion` having 0.00 recall and 0.00 F1. This means the fine-tuned model never successfully learned that class on the test set.

## What the Model Learned vs. What I Intended

I intended the fine-tuned model to learn discourse intent: whether a post was sharing reusable creator knowledge, asking for technical help, discussing broader model/community issues, or simply reacting/showcasing.

The fine-tuned model learned some surface-level distinctions but did not learn the full taxonomy. It became especially biased toward `technical_help` and failed completely on `model_news_discussion`. The confidence scores for many wrong predictions were around 0.26 to 0.28, which is close to random uncertainty for a four-class classifier. This suggests the model was not strongly separating the categories.

The zero-shot baseline performed much better because it could use the full written label definitions and general language reasoning at inference time. The baseline could interpret subtle intent better than a small model fine-tuned on only 200 short examples.

## Why Fine-Tuning Underperformed

There are several likely causes:

1. **Small dataset size**: 200 examples is enough for a course project, but it is small for learning subtle discourse boundaries.
2. **Short text examples**: Many examples were titles or short comments. They often lacked enough context for DistilBERT to infer intent.
3. **Overlapping labels**: `workflow_resource`, `technical_help`, and `model_news_discussion` all contain model/tool/workflow vocabulary.
4. **Weak context for model news**: `model_news_discussion` sometimes required broader community context that was not obvious from the text alone.
5. **Strong baseline**: The Groq/Llama baseline had access to detailed label definitions and stronger general reasoning.

## Reflection

The biggest lesson from this project is that label design matters as much as the model. The taxonomy made sense to me as a human annotator, but several labels shared the same vocabulary. For example, model names, workflow names, and tool names appeared in multiple classes. That made the task harder for a small fine-tuned model.

The zero-shot baseline's stronger performance shows that for subjective discourse classification, a large instruction-following model can sometimes outperform a smaller fine-tuned classifier when the dataset is small. The fine-tuned model needed more examples, longer text context, and possibly cleaner class boundaries.

If I improved this project, I would collect more examples, include post body text along with titles, and possibly merge or redefine the most confused labels. One improvement would be to split the task into two stages: first classify whether the post is high-signal or low-signal, then classify the high-signal posts into workflow, help, or news/discussion.

## Spec Reflection

The planning document helped force early decisions about the community, labels, edge cases, and evaluation criteria. Specifically, it made me define what "good enough" performance meant before seeing results, and it made me think about edge cases like short posts and model/tool update posts.

The main place my implementation diverged from the original intention was in the final model performance: I expected fine-tuning to improve over the baseline, but it performed significantly worse. Instead of hiding that, I treated it as the main evaluation finding. The divergence happened because the dataset was small, many examples were short, and several labels shared the same AI-tool vocabulary.

## AI Usage

I used AI assistance to help design the label taxonomy, draft the planning document, create an initial labeled dataset, prepare the baseline prompt, interpret the confusion matrix, and write the evaluation report. I manually reviewed the labels, outputs, and final analysis rather than blindly accepting model suggestions.

Specific AI usage examples:

1. **Label stress-testing**: I asked an AI tool to help refine the four-label taxonomy after choosing r/StableDiffusion. I revised the labels to make them more specific and to avoid vague categories like "good" or "bad."
2. **Annotation assistance**: I used AI help to organize an initial set of labeled examples. I reviewed the label definitions and made labeling decisions based on the intended discourse role of each example.
3. **Failure pattern analysis**: After getting wrong predictions from the fine-tuned model, I used AI to identify likely failure patterns such as short examples, overlapping vocabulary, and confusion between `workflow_resource`, `technical_help`, and `model_news_discussion`. I then checked those patterns against the confusion matrix and wrong predictions.
4. **Report drafting**: I used AI assistance to turn the metrics and error analysis into a readable README. I revised the language to be honest about the fine-tuned model underperforming instead of claiming it worked better than it did.

## Files

| File | Purpose |
|---|---|
| `planning.md` | Project plan, community choice, label taxonomy, edge cases, evaluation plan, and AI usage plan. |
| `data/labeled_data.csv` | Labeled dataset with 200 examples. |
| `evaluation_results.json` | Exported evaluation summary from the Colab notebook. |
| `results/confusion_matrix.svg` | Fine-tuned model confusion matrix visualization. |
| `README.md` | Final project report and evaluation writeup. |
| Completed Colab notebook | Fine-tuning and evaluation notebook committed to the repo. |

## Demo Video Notes

The demo video should show:

1. The GitHub repo and dataset.
2. The four labels and what they mean.
3. The Colab notebook running or showing 3-5 fine-tuned predictions with label and confidence visible.
4. At least one correct prediction, with an explanation of why it is reasonable.
5. At least one incorrect prediction, with an explanation of what went wrong.
6. The results comparison showing the zero-shot baseline outperforming the fine-tuned model.
7. The confusion matrix and failure analysis.
