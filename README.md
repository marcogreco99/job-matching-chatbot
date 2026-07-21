# Hierarchical Job Matching Chatbot

Chatbot based on a hierarchical multi-agent architecture (CrewAI) for CV analysis, matching with job offers through semantic search, and candidate salary estimation.

The system exposes a conversational interface (Gradio) in which a **manager agent** interprets the user's requests and delegates, when necessary, to a set of **specialist agents**, each responsible for a specific task in the recruiting process.

## Architecture

The project is built with [CrewAI](https://github.com/crewAIInc/crewAI) following a hierarchical process: a manager agent receives the user's request, autonomously decides which specialists to involve, and composes the final response.

### Agents

| Agent | Role |
|---|---|
| **Intelligent Assistant and Manager** | Analyzes the user's request and delegates to specialists only when strictly necessary |
| **Pdf Specialist** | Extracts text and work experience/skills from a CV in PDF format |
| **Mathematical Specialist** | Calculates the CV score and the text readability score |
| **Semantic Search Specialist** | Retrieves the job offers most relevant to the CV through vector search on Qdrant |
| **Soft Skills Specialist** | Evaluates the compatibility between the candidate's soft skills and the job offer |
| **Hard Skills Specialist** | Evaluates the compatibility between the candidate's hard skills and the job offer |
| **Skill Coherence Evaluator** | Measures the lexical and semantic coherence between the CV's soft and hard skills |
| **Salary Estimation Specialist** | Estimates a fair compensation based on compatibility scores and the candidate's profile |

### Custom tools

- `PDFReadTool` - text extraction from PDF (PyPDF2)
- `ScoreCVTool` - CV score based on the number of skills
- `TextReadabilityTool` - readability score (Flesch Reading Ease, via `textstat`)
- `QdrantSearchTool` - semantic search of job offers on Qdrant
- `CoherenceTool` - lexical coherence (via LLM) and semantic coherence (via embeddings + cosine similarity) between soft and hard skills

### Models used

- **Main LLM**: Mistral (`mistral-large-2411`) via API, used by the CrewAI agents
- **Custom local LLM**: `Qwen/Qwen2.5-1.5B-Instruct` (Hugging Face `transformers`), used by the `CoherenceTool` for lexical coherence evaluation
- **Embedding**: `all-mpnet-base-v2` (Sentence Transformers), used for semantic search and semantic coherence calculation

### Tech stack

- [CrewAI](https://github.com/crewAIInc/crewAI) / `crewai_tools` - multi-agent orchestration
- [Qdrant](https://qdrant.tech/) - vector database for job offers
- [Sentence Transformers](https://www.sbert.net/) - text embeddings
- [Hugging Face Transformers](https://huggingface.co/docs/transformers) - local LLM model
- [Gradio](https://www.gradio.dev/) - chat interface
- PyPDF2, textstat, scikit-learn, pandas, numpy

## Requirements

The notebook is designed to run on **Kaggle**, with secrets managed through `kaggle_secrets.UserSecretsClient`. The following secrets/variables are required:

| Name | Description |
|---|---|
| `QDRANT_HOST` | Qdrant cluster endpoint |
| `QDRANT_API_KEY` | Qdrant API key |
| `MISTRAL_API_KEY` | Mistral AI API key |
| `HF_TOKEN` | Hugging Face token (for model download) |


If you want to run the notebook outside of Kaggle, `UserSecretsClient` needs to be replaced with an equivalent mechanism (e.g. environment variables or an `.env` file).

## Installation

Dependencies are installed directly in the first cells of the notebook:

```bash
pip install crewai crewai_tools textstat sentence_transformers qdrant-client PyPDF2 gradio
```

A Qdrant collection populated with job offers (`job_offers_collection`) is also required, indexed with the same embedding model (`all-mpnet-base-v2`).

## Usage

1. Configure the secrets listed above (on Kaggle, via "Add-ons > Secrets").
2. Run all cells of the notebook `chatbot-hierarchical final.ipynb`.
3. A Gradio interface will start, where you can:
   - Attach a CV in PDF (or provide its path in the chat)
   - Ask questions in natural language (e.g. "find me the offers best suited to this CV", "what is the score of this CV?", "how coherent are its soft and hard skills?")
   - Enable the display of intermediate agent outputs ("Tool Outputs" and "Reasoning Outputs")

The manager agent responds directly when possible, or delegates to the individual specialists based on the type of request, indicating at the top of the response which agents were involved.

## Notes

This project was developed in a study/educational context to explore hierarchical multi-agent architectures applied to recruiting and job matching.

## License

⚠️ This is NOT an open source project. The code is published on GitHub for demonstration/portfolio purposes only: no permission is granted for reproduction, redistribution, modification, or commercial use. See the [LICENSE.md](LICENSE.md) file for the complete terms.
