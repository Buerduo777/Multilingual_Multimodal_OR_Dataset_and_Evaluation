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


# Project Overview

This project targets harmful-content detection and prompt generation through an end-to-end pipeline that spans data acquisition, cleaning, labeling, rewriting, and evaluation. Multi-model voting, rewriting strategies, and automated testing accelerate the creation of high-quality safety datasets and analysis.

# Project Flowchart

![Project Flowchart](14_Flow_Chart_FInal1.png)


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
