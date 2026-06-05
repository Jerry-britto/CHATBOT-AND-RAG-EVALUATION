# LLM and RAG Evaluation

In this repository I have covereed my learnings with respect to LLM and RAG Evaluation using Langmsith. The evaluation approach used here centers on automated evaluation with an LLM acting as a judge, tracked and managed through LangSmith.

**Goals:**
- Compare different LLM responses.
- Evaluation a RAG system based on various factors.
- Use an LLM-as-judge to score relevance, factuality, helpfulness and hallucinations.
- Track experiments, prompts, and evaluation artifacts with LangSmith for reproducibility.

**Key artifacts in this workspace:**
- [experiment_results.csv](experiment_results.csv) — aggregated experiment outputs and scores.
- [chatbot_evaluation.ipynb](chatbot_evaluation.ipynb) — interactive analysis and plotting.
- [rag_evaluation.ipynb](rag_evaluation.ipynb) — notebook with RAG-specific experiment steps.

**What I learned**
- LLM-as-judge is a practical way to scale evaluation: a strong LLM can compare or score outputs on qualitative axes (helpfulness, accuracy, hallucination) and generate consistent annotations.
- LangSmith provides experiment tracking, run metadata, and a structured place to store prompts, model outputs, and evaluation judgments — which aids reproducibility and comparison across runs.
- RAG often improves factuality and groundedness but must be evaluated for retrieval errors and hallucinations introduced by poor snippet selection.

Methodology
-----------

1. Data & prompts
	- Prepare a set of prompts/questions and (optionally) reference answers or expected facts.
	- For RAG runs, retrieve top-k documents per query using the chosen retriever.

2. Generate responses
	- Produce responses from the base LLM (no retrieval) and from the RAG pipeline.

3. LLM-as-judge evaluation
	- Use a high-quality LLM to compare pairs of outputs or to score single outputs along predefined axes:
	  - Helpfulness / Answer completeness
	  - Factuality / Correctness
	  - Hallucination presence (binary or graded)
	  - Citation adequacy (does the RAG answer cite supporting docs when appropriate)
	- Implement scoring via structured prompts (rubrics) so the judge LLM returns machine-readable judgments (JSON or CSV-friendly rows).

4. Metrics & aggregation
	- Aggregate per-question judgments into metrics such as percent correct, average helpfulness score, and hallucination rate.
	- Optionally compute pairwise win-rate (RAG vs base), and calibration measures.

	Evaluation types
	----------------

	This project contains two distinct, non-competitive evaluation tracks:

	- Chatbot/LLM evaluation
		- Focus: conversational behavior, user intent handling, turn-level coherence, and safety.
		- Procedure: simulate or use recorded conversational turns, evaluate responses for helpfulness, dialogue appropriateness, and consistency across turns using the LLM-as-judge rubric.
		- Goal: measure chat quality and user-facing interaction metrics rather than direct factual recall.

	- RAG evaluation
		- Focus: grounding, factuality, citation adequacy, and retrieval quality when using external documents.
		- Procedure: run retrieval + generation pipelines, capture retrieved snippets, and judge answers for factual correctness, citation presence, and hallucination.
		- Goal: assess how retrieval affects correctness and grounding; does RAG reduce hallucinations and improve answers?

	Note: these tracks are complementary analyses of different model behaviors — there is no competition between them. Each track uses a dedicated rubric and metrics tailored to its objectives.

LangSmith integration
---------------------

- Use LangSmith to log runs, prompts, models, and evaluation artifacts. Each run should include:
  - Input prompt and metadata
  - Retrieved documents (for RAG runs)
  - Model outputs
  - Judge outputs (LLM-as-judge JSON)
- LangSmith's UI makes it easy to compare runs, inspect examples, and export artifacts for analysis.

Reproducibility & setup
-----------------------

1. Create and activate a virtual environment (example):

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

2. Run the main experiment driver (examples vary by how you configured `main.py`):

```bash
python main.py --config config.yaml
```

3. Open the notebooks for interactive exploration:

```bash
jupyter lab
```

Notes on configuration
----------------------
- Keep API keys and secrets out of the repo. Use environment variables for model and LangSmith keys.
- Fix random seeds where appropriate and record retriever parameters (top-k, embedding model, index type).

Working with the LLM-as-judge
-----------------------------

- Design rubric prompts that instruct the judge LLM to return only structured outputs (for example, JSON with fields `helpfulness`, `factuality`, `hallucination`, and `notes`).
- Use few-shot examples in the rubric to improve judge consistency.
- Validate the judge on a small human-annotated subset to estimate judge reliability and bias.

Interpreting results
--------------------
- Compare aggregated metrics across conditions (base LLM vs RAG).
- Inspect failure cases (high judged helpfulness but low factuality, missing citations, or retrieval mismatches).
- Use LangSmith traces to link a judged score back to the exact prompt, retrieved documents, and model output.