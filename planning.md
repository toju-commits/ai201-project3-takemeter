# AI201 Project 3 Planning: TakeMeter

## Project Title

**TakeMeter: Classifying Generative AI Creator Discourse**

## Community

For this project, I am studying public posts and comments from **r/StableDiffusion**, a Reddit community focused on open-source and local AI image/video/software tools such as Stable Diffusion, Flux, PixArt, ComfyUI, LoRAs, workflows, model releases, and related creator tools.

This community is a strong fit for a discourse classification task because its posts are not all the same kind of conversation. Some posts share repeatable workflows or resources, some ask for technical help, some discuss model releases or licensing/commercial use, and some are mainly memes, reactions, or hype. That variety makes it useful for building a classifier that can separate high-signal creator knowledge from lower-signal discussion.

The practical goal of the classifier is to categorize AI creator discourse by intent and usefulness, not to judge whether a post is morally good or bad. A successful model could later support creator tooling by helping surface useful workflows, identify technical pain points, track model/news discussions, and filter low-signal reactions.

## Label Set

I will use four mutually exclusive labels:

1. `workflow_resource`
2. `technical_help`
3. `model_news_discussion`
4. `showcase_reaction`

These labels were chosen after reviewing the r/StableDiffusion front page, visible flairs, community rules, and example post patterns. The subreddit itself emphasizes open-source/local AI tools, encourages posters to explain workflows when relevant, and uses flairs such as Discussion, Meme, News, Resource - Update, Workflow Included, and No Workflow. The labels below translate those real community patterns into a smaller supervised learning taxonomy.

## Label Definitions

### `workflow_resource`

A post or comment should be labeled `workflow_resource` when the main purpose is to share a repeatable creator process, tool, prompt strategy, resource, model setup, node graph, LoRA, checkpoint, tutorial, benchmark, or concrete production method.

This label includes:

- Workflow-included posts
- Tutorials
- Prompting methods
- ComfyUI or Automatic1111 setups
- LoRA/model/resource releases
- Hardware or performance tests when they provide useful setup details
- Practical advice that another creator could copy or adapt

Example:

> "I used this Ideogram workflow with Claude to generate JSON prompts. Not cherry picked."

Decision rule: If the post gives another user something they could directly use, reproduce, download, modify, or learn from, label it `workflow_resource`.

### `technical_help`

A post or comment should be labeled `technical_help` when the main purpose is troubleshooting, asking for help, explaining an error, diagnosing setup issues, or solving technical problems.

This label includes:

- Installation problems
- GPU/VRAM/CUDA issues
- ComfyUI node errors
- Model loading failures
- Bad outputs caused by settings
- Questions about why a workflow or generation process is not working
- Requests for recommended settings or tools when framed as a problem

Example:

> "ComfyUI keeps failing when I load this checkpoint. Is this a VRAM issue or am I missing a custom node?"

Decision rule: If the user mainly wants help fixing something or understanding why something failed, label it `technical_help`.

### `model_news_discussion`

A post or comment should be labeled `model_news_discussion` when the main purpose is discussing a model release, model comparison, license restriction, commercial-use concern, platform change, policy issue, or broader AI art/community debate.

This label includes:

- New model announcements
- Comparisons between models
- License/commercial-use debates
- Copyright or artist-discourse threads
- News about tools, companies, or platform rules
- Opinionated but substantive discussions about the direction of AI image generation

Example:

> "Ideogram 4 is a great model, but the license is too restrictive for commercial work."

Decision rule: If the post is mainly about what a model/tool means for the community, business, licensing, art, or the future of AI generation, label it `model_news_discussion`.

### `showcase_reaction`

A post or comment should be labeled `showcase_reaction` when the main purpose is showing an output, reacting emotionally, joking, memeing, praising, dunking, or making a low-detail statement without a reusable workflow, technical issue, or substantive discussion.

This label includes:

- Memes
- Image/video showcases without enough workflow detail
- Hype posts
- Short reactions
- Jokes
- Low-context praise or criticism
- Vague claims such as "this is insane" or "AI is getting crazy"

Example:

> "It is still nuts to me how realistic AI is getting."

Decision rule: If the post is mostly a reaction to an output or a vibe check, and does not give reusable process details or ask a clear technical question, label it `showcase_reaction`.

## Hard Edge Cases

### Edge Case 1: Showcase with workflow included

Some posts show generated images while also including settings, prompts, model names, or a full ComfyUI workflow. If the workflow is detailed enough for another user to reproduce or learn from, I will label the example `workflow_resource` instead of `showcase_reaction`.

### Edge Case 2: News post with hype language

A post may use hype language such as "this model is insane" while also discussing a new model release, benchmark, license, or comparison. If the main purpose is discussing the model or release, I will label it `model_news_discussion`. If the post is only excitement with no substance, I will label it `showcase_reaction`.

### Edge Case 3: Technical complaint vs technical help

If a user says "this tool is broken" without enough detail, I will label it `showcase_reaction` because it is mostly a low-signal reaction. If they describe the error, setup, symptoms, or what they tried, I will label it `technical_help`.

### Edge Case 4: Hardware benchmark vs workflow/resource

Posts about GPU generation time, VRAM use, or low-VRAM setups can be ambiguous. If they include useful details such as hardware, model, steps, sampler, generation time, or setup advice, I will label them `workflow_resource`. If they only say "my GPU is slow" or "this runs great" with little detail, I will label them `showcase_reaction`.

### Edge Case 5: Market/ethics discussion vs model/news discussion

I considered making a separate label for market/ethics discussion, but that would likely create too many boundary issues with model release and licensing conversations. Instead, I will fold licensing, commercial use, copyright, artist-discourse, and platform policy conversations into `model_news_discussion`.

## Data Collection Plan

I will collect at least **200 public Reddit examples** from r/StableDiffusion. Each example will be a post title, post body, comment, or title/body combination that contains enough text to classify. I will avoid collecting usernames, private information, direct links to user profiles, or unnecessary metadata.

The dataset will be stored as a CSV file at:

```text
data/labeled_data.csv
```

The required columns will be:

```text
text,label
```

Optional columns may include:

```text
source_type,notes
```

I will try to keep the class distribution balanced. My target distribution is approximately:

- `workflow_resource`: 50 examples
- `technical_help`: 50 examples
- `model_news_discussion`: 50 examples
- `showcase_reaction`: 50 examples

If one class is much easier to collect than the others, I will intentionally search for underrepresented examples before finalizing the dataset. No single label should dominate the dataset.

## Annotation Rules

While labeling, I will follow these rules:

1. Label the main purpose of the text, not every topic mentioned.
2. Prefer `workflow_resource` when the text gives reusable technical or creative process information.
3. Prefer `technical_help` when the text is centered on solving a specific problem.
4. Prefer `model_news_discussion` when the text is centered on model releases, licensing, commercial use, platform changes, or broader community debate.
5. Prefer `showcase_reaction` when the text is mostly emotional, humorous, visual, hype-based, or low-detail.
6. Exclude examples that are too short, unclear, deleted, or dependent on an image without enough text context.

## Evaluation Metrics

I will evaluate both the zero-shot baseline model and the fine-tuned model using:

- Overall accuracy
- Per-class precision
- Per-class recall
- Per-class F1-score
- Confusion matrix
- Failure examples

Accuracy will show the general performance of the classifier, but macro F1 will be especially important because the classes may not be perfectly balanced. The confusion matrix will help identify whether the model is mixing up similar categories, such as `workflow_resource` and `model_news_discussion`, or `showcase_reaction` and hype-heavy news posts.

## Baseline Model Plan

For the baseline, I will use a zero-shot or prompt-based LLM classifier through Groq/Llama. The baseline prompt will include the four label definitions and ask the model to return one label for each example.

The baseline will be evaluated on the same held-out test set as the fine-tuned model. This makes the comparison fair: both models will classify the same examples using the same labels.

## Fine-Tuned Model Plan

For the fine-tuned model, I plan to use the starter notebook and fine-tune a lightweight transformer model such as `distilbert-base-uncased` on the labeled dataset.

The dataset will be split approximately as:

- 70% training
- 15% validation
- 15% test

The model will be evaluated on the test set after training. I will compare its performance against the zero-shot baseline to see whether task-specific fine-tuning improves classification.

## Definition of Success

A successful project will produce a classifier that performs meaningfully better than the zero-shot baseline and shows that it learned the discourse categories in r/StableDiffusion.

My target is:

- At least 70% overall accuracy
- Macro F1 of at least 0.65
- No class with extremely poor recall
- Clear analysis of common failure cases

If the fine-tuned model does not beat the baseline, that will still be useful. I will analyze whether the issue came from noisy labels, overlapping categories, too few examples, or the baseline model already being strong at this task.

## Expected Failure Modes

I expect the model to struggle with:

- Hype-heavy model discussions that look like reactions
- Showcases that mention a model but do not provide workflow details
- Technical posts that are short and lack enough context
- Resource posts that include both practical workflow information and broader commentary
- Sarcasm, memes, and community-specific slang

These failures are important because they reflect real ambiguity in AI creator communities.

## AI Tool Usage Plan

I will use AI tools to assist with planning, annotation consistency, baseline comparison, and error analysis.

Specifically, I may use AI to:

1. Help refine label definitions before collecting the full dataset.
2. Suggest labels for small batches of examples.
3. Identify ambiguous examples that need manual review.
4. Generate the zero-shot baseline predictions.
5. Help interpret the confusion matrix and failure cases.

However, I will manually review all final labels. The dataset labels will be my responsibility, and I will not blindly accept AI-generated annotations.

## Project Relevance

This project is relevant to AI product development, creator tools, NLP, and content intelligence. A classifier like this could help identify useful creator workflows, surface technical pain points, track model/news discussions, and filter low-signal hype from high-signal AI creator knowledge.

From a portfolio perspective, this project demonstrates practical NLP work: dataset design, label taxonomy, supervised fine-tuning, baseline comparison, evaluation metrics, and error analysis on a real online community.