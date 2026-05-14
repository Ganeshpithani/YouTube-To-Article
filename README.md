# YouTube-To-Article
Turn any YouTube video into a publication-ready article webpage. Paste a URL, get a complete HTML/CSS/JS site — auto-routes short &amp; long transcripts, strips intros &amp; sponsors, and runs fully local with LangChain, Ollama (Gemma3) &amp; Streamlit. No OpenAI key needed.


# 🎬 YouTube → Article Generator

> Transform any YouTube video into a **publication-ready article webpage** using LangChain, Ollama, and Streamlit.

---

## ✨ What It Does

Paste a YouTube URL → get a fully designed **HTML / CSS / JS webpage** styled like Medium or Dev.to, generated end-to-end by a local LLM — no OpenAI key required.

The pipeline:
1. **Fetches** the YouTube transcript automatically
2. **Routes** to the right summarization strategy based on transcript length
3. **Summarizes** the content into a clean, structured article (ignoring intros, sponsors, subscribe CTAs)
4. **Generates** a complete, responsive webpage with dark/light theme toggle

---

## 🏗️ Architecture

```
YouTube URL
     │
     ▼
YoutubeLoader  ──►  Transcript
                        │
           ┌────────────┴────────────┐
           │  RunnableBranch Router  │
           └────────────┬────────────┘
                        │
          Short (<1000 tokens)    Long (≥1000 tokens)
                  │                       │
          base_summarizer         recursive_summarize()
          (single LLM call)       (chunked + agent loop)
                  │                       │
                  └──────────┬────────────┘
                             ▼
                    web_dev_template
                             │
                             ▼
                   ChatOllama (gemma3)
                             │
                             ▼
              index.html + style.css + script.js
                             │
                             ▼
                        website.zip
```

---

## 🚀 Quick Start

### 1. Prerequisites

- Python 3.9+
- [Ollama](https://ollama.com) installed and running
- The `gemma3` model pulled

```bash
ollama pull gemma3:latest
ollama serve          # keep this running in a terminal
```

### 2. Clone & Install

```bash
git clone https://github.com/your-username/yt-article-generator.git
cd yt-article-generator

pip install -r requirements.txt
```

### 3. Environment Setup

Create a `.env` file in the project root:

```env
# Optional – only needed if switching to OpenAI or Gemini
# OPENAI_API_KEY=sk-...
# GOOGLE_API_KEY=...
```

### 4. Run

**Streamlit UI (recommended):**
```bash
streamlit run app.py
```

**CLI / Script:**
```bash
python RAG.py
```

---

## 📦 Project Structure

```
yt-article-generator/
├── RAG.py          # Core pipeline (LangChain chains, agent, router)
├── app.py          # Streamlit frontend
├── .env            # API keys (git-ignored)
├── requirements.txt
├── README.md
│
└── outputs/        # Generated on run
    ├── index.html
    ├── style.css
    ├── script.js
    └── website.zip
```

---

## 📋 Requirements

```txt
langchain
langchain-community
langchain-ollama
langchain-text-splitters
langchain-core
python-dotenv
streamlit
yt-dlp
youtube-transcript-api
```

Install all at once:

```bash
pip install langchain langchain-community langchain-ollama \
            langchain-text-splitters langchain-core \
            python-dotenv streamlit yt-dlp youtube-transcript-api
```

---

## 🔧 Configuration

| Setting | Location | Default |
|---|---|---|
| LLM model | `RAG.py` | `gemma3:latest` |
| Chunk size | `get_text_chunks()` | `5000` chars |
| Chunk overlap | `get_text_chunks()` | `200` chars |
| Short/long threshold | `estimate_transcript_length()` | `1000` chars |
| Agent summarize trigger | `SummarizationMiddleware` | `1000` tokens |
| Agent context kept | `SummarizationMiddleware` | `200` tokens |

---

## 🤖 How the Routing Works

```python
smart_summarizer = RunnableBranch(
    # Long transcript  → recursive chunked summarization via agent
    (RunnableLambda(estimate_transcript_length), long_summarizer),
    # Short transcript → single-shot summarization
    base_summarizer
) | web_dev_template | llm | StrOutputParser()
```

The router calls `estimate_transcript_length()` which fetches the transcript and checks its character count. Videos under the threshold go through the fast `base_summarizer`; longer ones are chunked and processed iteratively by the agent with `SummarizationMiddleware` to stay within the context window.

---

## 📄 Output Format

The LLM is prompted to return output with strict delimiters:

```
--html--
<!DOCTYPE html> ...
--html--

--css--
body { ... }
--css--

--js--
// dark mode toggle etc.
--js--
```

The Streamlit app parses these blocks and gives you:

- **Live preview** rendered in the browser
- **Per-file download** buttons (HTML / CSS / JS)
- **One-click ZIP** of the full website

---

## 🧩 Switching LLM Providers

The model is swappable — just replace the `llm` line in `RAG.py`:

**OpenAI:**
```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4o")
```

**Google Gemini:**
```python
from langchain_google_genai import ChatGoogleGenerativeAI
llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash")
```

---

## ⚠️ Troubleshooting

| Problem | Fix |
|---|---|
| `Connection refused` on Ollama | Run `ollama serve` in a separate terminal |
| `Model not found` | Run `ollama pull gemma3:latest` |
| No transcript available | The video may have captions disabled; try another video |
| Empty HTML/CSS/JS tabs | Check **Raw Output** tab — the model may have deviated from the delimiter format |
| Slow generation | Normal for large transcripts on CPU; use a GPU or switch to a smaller model |

---

## 🗺️ Roadmap

- [ ] Support for multiple LLM backends via dropdown in UI
- [ ] Custom article tone/style selector
- [ ] Export to Medium-compatible Markdown
- [ ] Batch processing for playlists
- [ ] Vector store integration for Q&A on transcript

---

## 📝 License

MIT License — free to use, modify, and distribute.

---

## 🙌 Acknowledgements

- [LangChain](https://github.com/langchain-ai/langchain) — pipeline orchestration
- [Ollama](https://ollama.com) — local LLM inference
- [Streamlit](https://streamlit.io) — frontend UI
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) — YouTube transcript extraction
