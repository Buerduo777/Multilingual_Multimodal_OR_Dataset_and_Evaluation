# Building a Multilingual Dataset to Benchmark Over-Refusal in LLMs
## USYD  Group CS62-2

# Important Notes
- The project is covered by a Non-Disclosure Agreement (NDA); implementation details must remain confidential per client requirements.

- All testing API keys will be removed from the final delivery. To run the project locally, place your own credentials in `call_api/api_config.py` and `call_api/api_keys.json`.

- Runtime requirements:
python 3.10

CPU: 12 cores

GPU: NVIDIA RTX2000 8G

Additional platforms: LMStudio, Ollama

Models exercised in testing: qwen3-4b, llama3.1-8b, gemma3-4b, gemini-2.5-pro, gemini-2.5-flash, deepseek-v3.2

- Final project deliverables (for the customer)
    - A high-quality data generation pipeline and the resulting datasets (see the `data` module)
    - A model evaluation framework (see the `evaluation` module)


# Overview

This project targets harmful-content detection and prompt generation through an end-to-end pipeline that spans data acquisition, cleaning, labeling, rewriting, and evaluation. Multi-model voting, rewriting strategies, and automated testing accelerate the creation of high-quality safety datasets and analysis.

# Project Flowchart

![Project Flowchart](md_images/14_Flow_Chart_FInal1.png)


## Directory Structure and Module Overview

- `main_pipeline_v1.py`: Orchestrates extraction, voting, rewriting, and balance checks in sequence; serves as an end-to-end entry point.
- `call_api/`: Centralizes large-model invocation logic, covering key management, model configuration, and helpers for Gemini, DeepSeek, OpenAI, and related services.
- `extract/`: Handles data ingestion and preprocessing by merging ten base datasets, deduplicating via similarity, supplementing with crawler data, and producing the base prompt pool.
- `rewrite/`: Houses 11 rewriting strategies across task forms, expression structures, and lexical variations to broaden prompt diversity.
- `labeller/`: Provides automated labeling utilities that blend rule sets, LLM voting, and local models to produce final annotations.
- `voting/`: Implements multi-model consensus workflows, monitors rewritten sample counts, and triggers crawler-based replenishment when imbalance is detected.
- `model_response_test/`: Contains offline and online test harnesses—both local and remote templates—to evaluate refusal behavior of candidate models.
- `evaluation/`: Offers notebooks, charts, and cached features for visualization, metric analysis, and retrospective studies.
- `web_crawler/`: Supplies crawling and data-cleaning scripts that source external prompts for extraction and replenishment stages.
- `human_label/`: Stores manual labeling utilities, guides, and historical records to compare against automated outputs.
- `data/`: Central repository for raw inputs, rewritten results, model-test outputs, and supporting CSV/JSON/notebook artifacts exchanged across modules.
- `tester_v1.py`, `tester_v2.py`: Standalone test runners for quick regression or targeted flow execution.

## Basic Usage Flow

1. Prepare the required input data—for example, base prompts under `data/source_data/` or newly crawled samples from `web_crawler/`.
2. Configure the necessary model API keys in `call_api/api_keys.json` (see below) and adjust switches in `call_api/api_config.py` to match your environment.
3. Run `main_pipeline_v1.py` or `voting/voting_pipeline.py` to execute extraction, voting, rewriting, and balance-check stages as instructed.
4. Use scripts or notebooks under `model_response_test/` and `evaluation/` to validate outcomes and generate visual analytics.

## API Key Configuration

The project loads credentials for external services from `call_api/api_keys.json` by default. On first use:

1. Copy the template `call_api/api_keys.json` (or edit the file directly) and replace placeholders with your secrets, such as:
   - `openai_api_key`
   - `gemini_api_key_*`
   - `deepseek_api_key_*`
2. If you prefer environment variables, set them before runtime and extend the loading logic in `call_api/api_config.py` accordingly.
3. Ensure the credentials file is never committed to public repositories. Add it to `.gitignore` or adopt local secret-management practices.

Once credentials are in place, follow the module guidelines to run the scripts and notebooks required for dataset construction and evaluation.

## Project Overview （Report）
### Motivation & objective of the work
#### Motivation​

LLMs are widely used, but over-refusal occurs reducing usability due to sensitive keywords, ambiguous phrasing, defense mechanism varies, etc.​

Datasets of Existing studies lack multilingual coverage, semantics, expression diversity, automation, large scale and standardized evaluation.​

#### Objectives​

A robust and high-fidelity pipeline for constructing a multilingual over-refusal assessment dataset was developed.​

Generated a representative benchmark dataset covering various real-world cueing scenarios.​

The over-refusal behavior of mainstream LLMs (Llama3.1-8b, Qwen3-4b, Gemma3-4b, Gemini2.5 and Deepseek-V3.2) was systematically evaluated and compared, leading to a set of constructive and actionable conclusions.​

### Related work & background study
#### Existing Findings:​

Cross-linguistic: over-refusal rate varies significantly across languages.​

Structural factors: complex syntax and high-risk semantics increase refusal likelihood.​

Safety alignment: multi-stage alignment amplifies conservative bias.​

#### Existing Benchmarks:
OR-Bench, SORRY-Bench, HarmBench, PH Test.​

Limitation: lack of unified indicators, limited language coverage, poor cross-benchmark comparability;​

#### Research Gaps:​

Lack of multi-lingual unified evaluation framework;​

Existing mainstream benchmarks lack of examine the relationship between prompt linguistic structures and over-refusal behavior in LLMs;​

### Automatic Extraction​
#### Approach / Method​

Built a multi-stage pipeline to automatically extract high-quality prompts based on 300 manually screened base prompts.​

internal deduplication (Sentence-BERT) → Unsafe filtering → deduplication with 300 base prompts(＞0.7) → semantic filtering(≥0.5) → Gemini 2.5 classification → Structured CSV output.​

#### Key Results​

Automatically extracted high-quality, diverse, category-relevant prompts with a scalable pipeline that removes the need for manual screening.​

### Rewriter

#### Approach / Method​

Three-level rewriting pipeline: Adopting the architecture of "base prompt→seed prompt→8 types of rewriting", the progressive relationship is clearly defined to form a standardized chain.​

Phased invocation of Gemini: First, expand the base prompt to generate the seed prompt, and then complete 8 types of rewriting based on the seed prompt. The entire process is model-driven.​

Efficient processing framework: Build a batch processing and parallel processing framework, reuse multiple API keys to achieve parallel processing, and improve efficiency.​

Instruction principle: Strictly retain core semantics, unify the length standard, provide examples to assist Gemini in precise execution, and ensure quality standards​

#### Key Results​
Completed the rewriting of 600 base prompts for known datasets and 600 prompts for real scenarios of web crawler.​

### Minimal Toxic Rewriter​
#### Approach / Method​

Built a minimal-change rewriting pipeline aligned to harmless seeds.​

Instruction template with detailed rules + keyword-hint dictionary (EN/ZH) to ensure correct category selection and category-consistent minimal-change rewriting.​

Flow: rewritten harmless prompts → template + hint → model rewrite → pair with harmless.​

#### Key Results:​

Tightly aligned harmless–harmful pairs that accurately match the target category, read naturally, and consistent in Chinese and English.​

Cleaner refusal-boundary analysis across models and languages;  pipeline is reusable and extensible.​

### LLMs Testing & Assign Labels
#### Approach / Method​

Under a unified testing framework and API, batch-test both local and online LLMs using a shared system role prompt and consistent evaluation pipeline.​

Outputs are mapped to binary labels (answer/refuse) via a rule + LLM fusion algorithm: ​

#### Key Results​

Established a reproducible automated testing and labeling pipeline that supports large-scale over-refusal evaluation.​

Produced consistent refusal labels for comparing over-refusal behaviors across models and languages.​

​
### Main results
- Automated and reliable rewrite pipeline​

- Representative benchmark dataset ​

- Systematic evaluation pipeline ​

- Constructive and feasible conclusions​


![Sample](md_images/Sample.png)


### Results of the visualization
- Figure1: Heatmap of semantic similarity among eight rewriting methods
​![Figure1: Heatmap of semantic similarity among eight rewriting methods](md_images/avg_similarity_heatmap.png)

​- Figure2: Over-Refusal Rates of Mainstream LLMs Across Languages
​![Figure2: Over-Refusal Rates of Mainstream LLMs Across Languages](md_images/over_refusal_rate_by_model_language.png)

​- Figure3: Trend of Model Over-Refusal Across Safety Risk Categories
​![Figure3: Trend of Model Over-Refusal Across Safety Risk Categories](md_images/over_refusal_rate_trend.png)

​- Figure4: Over-Refusal by Prompt Rewriting Method
​![Figure4: Over-Refusal by Prompt Rewriting Method](md_images/over_refusal_rate_by_rewrite_method.png)

- Figure5: Radar chart of 5 models
​![Figure5: Radar chart of 5 models](md_images/refusal_ratio_radar_grid_shortlabel.png)

- Figure6: Other
​![Other1](md_images/over_refusal_rate_by_category_language.png)
​![Other2](md_images/over_refusal_rate_by_rewrite_language.png)
​![Other3](md_images/over_refusal_rate_by_category.png)



​
​