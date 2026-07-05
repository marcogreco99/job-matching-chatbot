# Chatbot Gerarchico per Job Matching

Chatbot basato su un'architettura multi-agente gerarchica (CrewAI) per l'analisi di CV, il matching con offerte di lavoro tramite ricerca semantica e la stima retributiva dei candidati.

Il sistema espone un'interfaccia conversazionale (Gradio) in cui un **agente manager** interpreta le richieste dell'utente e delega, quando necessario, a una serie di **agenti specialisti**, ciascuno responsabile di un compito specifico nel processo di recruiting.

## Architettura

Il progetto è costruito con [CrewAI](https://github.com/crewAIInc/crewAI) secondo un processo gerarchico: un manager agent riceve la richiesta dell'utente, decide autonomamente quali specialisti coinvolgere e compone la risposta finale.

### Agenti

| Agente | Ruolo |
|---|---|
| **Intelligent Assistant and Manager** | Analizza la richiesta dell'utente e delega agli specialisti solo se strettamente necessario |
| **Pdf Specialist** | Estrae testo ed esperienze/competenze da un CV in formato PDF |
| **Mathematical Specialist** | Calcola lo score del CV e il punteggio di leggibilità del testo |
| **Semantic Search Specialist** | Recupera le offerte di lavoro più affini al CV tramite ricerca vettoriale su Qdrant |
| **Soft Skills Specialist** | Valuta la compatibilità tra soft skill del candidato e offerta di lavoro |
| **Hard Skills Specialist** | Valuta la compatibilità tra hard skill del candidato e offerta di lavoro |
| **Skill Coherence Evaluator** | Misura la coerenza lessicale e semantica tra soft e hard skill del CV |
| **Salary Estimation Specialist** | Stima una retribuzione equa in base ai punteggi di compatibilità e al profilo del candidato |

### Strumenti (tool) custom

- `PDFReadTool` — estrazione testo da PDF (PyPDF2)
- `ScoreCVTool` — punteggio del CV basato sul numero di competenze
- `TextReadabilityTool` — punteggio di leggibilità (Flesch Reading Ease, via `textstat`)
- `QdrantSearchTool` — ricerca semantica delle offerte di lavoro su Qdrant
- `CoherenceTool` — coerenza lessicale (via LLM) e semantica (via embedding + cosine similarity) tra soft e hard skill

### Modelli utilizzati

- **LLM principale**: Mistral (`mistral-large-2411`) via API, usato dagli agenti CrewAI
- **LLM locale custom**: `Qwen/Qwen2.5-1.5B-Instruct` (Hugging Face `transformers`), usato dal `CoherenceTool` per la valutazione della coerenza lessicale
- **Embedding**: `all-mpnet-base-v2` (Sentence Transformers), usato per la ricerca semantica e per il calcolo della coerenza semantica

### Stack tecnologico

- [CrewAI](https://github.com/crewAIInc/crewAI) / `crewai_tools` — orchestrazione multi-agente
- [Qdrant](https://qdrant.tech/) — vector database per le offerte di lavoro
- [Sentence Transformers](https://www.sbert.net/) — embedding testuali
- [Hugging Face Transformers](https://huggingface.co/docs/transformers) — modello LLM locale
- [Gradio](https://www.gradio.dev/) — interfaccia chat
- PyPDF2, textstat, scikit-learn, pandas, numpy

## Requisiti

Il notebook è pensato per essere eseguito su **Kaggle**, con i segreti gestiti tramite `kaggle_secrets.UserSecretsClient`. Sono richiesti i seguenti secret/variabili:

| Nome | Descrizione |
|---|---|
| `QDRANT_HOST` | Endpoint del cluster Qdrant |
| `QDRANT_API_KEY` | API key di Qdrant |
| `MISTRAL_API_KEY` | API key di Mistral AI |
| `HF_TOKEN` | Token Hugging Face (per il download del modello) |


Se si desidera eseguire il notebook al di fuori di Kaggle, è necessario sostituire `UserSecretsClient` con un meccanismo equivalente (es. variabili d'ambiente o file `.env`).

## Installazione

Le dipendenze sono installate direttamente nelle prime celle del notebook:

```bash
pip install crewai crewai_tools textstat sentence_transformers qdrant-client PyPDF2 gradio
```

È inoltre necessaria una collezione Qdrant popolata con le offerte di lavoro (`job_offers_collection`), indicizzata con lo stesso modello di embedding (`all-mpnet-base-v2`).

## Utilizzo

1. Configurare i secret elencati sopra (su Kaggle, tramite "Add-ons > Secrets").
2. Eseguire tutte le celle del notebook `chatbot-hierarchical final.ipynb`.
3. Verrà avviata un'interfaccia Gradio in cui è possibile:
   - Allegare un CV in PDF (o indicarne il percorso in chat)
   - Porre domande in linguaggio naturale (es. "trovami le offerte più adatte a questo CV", "qual è lo score di questo CV?", "quanto sono coerenti le sue soft e hard skill?")
   - Abilitare la visualizzazione degli output intermedi degli agenti ("Tool Outputs" e "Reasoning Outputs")

Il manager agent risponde direttamente quando possibile, oppure delega ai singoli specialisti in base al tipo di richiesta, indicando in testa alla risposta quali agenti sono stati coinvolti.

## Note

Questo progetto è stato sviluppato in un contesto di studio/didattico per esplorare architetture multi-agente gerarchiche applicate al recruiting e al job matching.

## Licenza

⚠️ Questo NON è un progetto open source. Il codice è pubblicato su GitHub a solo scopo dimostrativo/di portfolio: non è concesso alcun permesso di riproduzione, redistribuzione, modifica o utilizzo commerciale. Vedi il file [LICENSE.md](LICENSE.md) per i termini completi.
