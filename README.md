# 🧠 ResearchMind

**ResearchMind** is a multi-agent AI research system that automatically searches the web, reads relevant sources, generates a structured research report, and reviews the generated report using a critic agent.

Instead of relying on a single LLM call to perform the entire research process, ResearchMind divides the task among specialized agents.

```text
User Research Topic
        ↓
   Search Agent
        ↓
   Reader Agent
        ↓
     Writer
        ↓
     Critic
        ↓
 Research Report + Feedback
```

---

## 🎯 Why ResearchMind?

Researching a topic manually usually requires several steps:

1. Search for relevant information.
2. Find reliable sources.
3. Open and read multiple webpages.
4. Extract important information.
5. Combine the information.
6. Write a structured report.
7. Review the report for quality and completeness.

ResearchMind attempts to automate this workflow using a **multi-agent architecture**.

Instead of asking one AI model:

> "Research this topic and give me a report."

ResearchMind gives different responsibilities to different components.

The Search Agent focuses on discovering information, the Reader Agent performs deeper reading, the Writer converts the collected information into a report, and the Critic evaluates the generated report.

This separation makes the research workflow easier to understand, extend, debug and improve.

---

# ✨ Features

- 🔎 Web research using Tavily
- 🤖 Multiple specialized AI agents
- 🌐 Real-time web information retrieval
- 📖 Automatic webpage scraping
- 🧹 HTML content cleaning
- ✍️ AI-generated research reports
- 🔍 AI-based report review
- 🔗 LangChain tool integration
- 🧠 LangGraph-powered agent execution
- ⚡ Groq-hosted LLM inference
- 🔐 Environment-variable-based API key management

---

# 🏗️ System Architecture

ResearchMind follows a sequential multi-agent research pipeline.

```text
                         ┌─────────────────┐
                         │      USER       │
                         │ Research Topic  │
                         └────────┬────────┘
                                  │
                                  ▼
                       ┌────────────────────┐
                       │    Search Agent    │
                       │                    │
                       │ Finds recent and   │
                       │ relevant sources   │
                       └─────────┬──────────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │ Tavily Search │
                         │     Tool      │
                         └───────┬───────┘
                                 │
                                 ▼
                         Search Results
                                 │
                                 ▼
                       ┌────────────────────┐
                       │    Reader Agent    │
                       │                    │
                       │ Selects a useful   │
                       │ source and reads   │
                       │ deeper content     │
                       └─────────┬──────────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │  scrape_url   │
                         │     Tool      │
                         └───────┬───────┘
                                 │
                         Requests + BS4
                                 │
                                 ▼
                         Scraped Content
                                 │
                                 ▼
                       ┌────────────────────┐
                       │       Writer       │
                       │                    │
                       │ Search Results +   │
                       │ Scraped Content    │
                       └─────────┬──────────┘
                                 │
                                 ▼
                         Research Report
                                 │
                                 ▼
                       ┌────────────────────┐
                       │       Critic       │
                       │                    │
                       │ Reviews quality,   │
                       │ accuracy and       │
                       │ completeness       │
                       └─────────┬──────────┘
                                 │
                                 ▼
                         Critic Feedback
```

---

# ⚙️ How It Works

## 1. User provides a research topic

The application starts from `pipeline.py`.

For example:

```text
Enter a research topic: Future of AI Agents
```

The topic is passed into the research pipeline.

---

## 2. Search Agent

The Search Agent receives the research topic and determines that it needs current information from the web.

It has access to the `web_search` tool.

```text
Research Topic
      ↓
Search Agent
      ↓
web_search()
      ↓
Tavily API
      ↓
Search Results
```

The search results contain information such as:

```text
Title
URL
Snippet
```

This allows the LLM to work with current information instead of depending entirely on its pretrained knowledge.

---

## 3. Reader Agent

Search results usually contain only short snippets.

Therefore, ResearchMind uses a separate Reader Agent for deeper research.

The Reader Agent receives the search results and selects a relevant URL.

It then calls:

```python
scrape_url(url)
```

The scraper uses:

- `requests`
- `BeautifulSoup`

to download and clean the webpage.

Elements such as:

```html
<script>
<style>
<nav>
<footer>
```

are removed before the text is returned.

The result is cleaner textual information that can be passed to the LLM.

---

## 4. Writer

The Writer receives two major sources of context:

```text
Search Results
       +
Detailed Scraped Content
```

These are combined into research context.

```text
research_combined
       │
       ▼
   Writer Chain
       │
       ▼
Research Report
```

The Writer then generates a structured report about the requested topic.

---

## 5. Critic

Generating a report does not necessarily mean that the report is good.

Therefore, ResearchMind includes another AI component called the **Critic**.

The generated report is passed to the Critic.

```text
Research Report
      ↓
Critic Chain
      ↓
Quality Review
```

The Critic can evaluate aspects such as:

- clarity
- completeness
- structure
- weaknesses
- missing information
- overall research quality

The pipeline therefore produces both a report and feedback about that report.

---

# 🧩 Low-Level Design (LLD)

The major internal components can be represented as:

```text
┌──────────────────────────────────────────────────────────────┐
│                       pipeline.py                            │
│                                                              │
│  run_research_pipeline(topic)                               │
│                                                              │
│  state = {                                                   │
│      search_result,                                          │
│      scraped_content,                                        │
│      report,                                                 │
│      feedback                                                │
│  }                                                           │
└───────────────┬──────────────────────────────────────────────┘
                │
                │ imports
                ▼
┌──────────────────────────────────────────────────────────────┐
│                        agents.py                             │
│                                                              │
│  build_search_agent()                                        │
│  build_reader_agent()                                        │
│  writer_chain                                                │
│  critic_chain                                                │
│                                                              │
│                     ChatGroq LLM                             │
└───────────────┬──────────────────────────────────────────────┘
                │
                │ uses
                ▼
┌──────────────────────────────────────────────────────────────┐
│                         tools.py                             │
│                                                              │
│  ┌────────────────────────┐  ┌────────────────────────────┐ │
│  │      web_search()      │  │       scrape_url()         │ │
│  │                        │  │                            │ │
│  │ TavilyClient           │  │ requests                   │ │
│  │       ↓                │  │      ↓                     │ │
│  │ Tavily Search API      │  │ BeautifulSoup              │ │
│  └────────────────────────┘  └────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Component Relationship

```text
ResearchPipeline
      │
      ├── SearchAgent
      │       │
      │       └── WebSearchTool
      │                 │
      │                 └── TavilyClient
      │
      ├── ReaderAgent
      │       │
      │       └── ScrapeURLTool
      │                 │
      │                 ├── Requests
      │                 └── BeautifulSoup
      │
      ├── WriterChain
      │
      └── CriticChain
              │
              ▼
          ChatGroq
```

---

# 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Python | Core programming language |
| LangChain | LLM chains, tools and AI application components |
| LangGraph | Agent execution and tool-calling workflow |
| Groq | Fast LLM inference |
| GPT-OSS | Language model used through Groq |
| Tavily | Real-time web search |
| BeautifulSoup | HTML parsing and webpage cleaning |
| Requests | Fetching webpage content |
| python-dotenv | Loading environment variables |
| Pydantic | Data validation used throughout the AI ecosystem |
| uv | Python environment and dependency management |

---

# 📁 Project Structure

```text
ResearchMind/
│
├── agents.py
├── tools.py
├── pipeline.py
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

### `pipeline.py`

Controls the complete research workflow.

```text
Search → Read → Write → Critique
```

It also maintains the state generated during each stage.

### `agents.py`

Contains the AI components responsible for reasoning and report generation.

It includes components such as:

```text
build_search_agent()
build_reader_agent()
writer_chain
critic_chain
```

### `tools.py`

Contains tools that agents can call.

Currently:

```text
web_search()
scrape_url()
```

### `.env`

Stores private API keys.

Example:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
```

> Never commit your `.env` file to GitHub.

---

# 🚀 Installation

## 1. Clone the repository

```bash
git clone <your-repository-url>

cd ResearchMind
```

## 2. Create a virtual environment

Using `uv`:

```bash
uv venv
```

## 3. Activate the environment

### Git Bash on Windows

```bash
source .venv/Scripts/activate
```

### PowerShell

```powershell
.venv\Scripts\Activate.ps1
```

## 4. Install dependencies

```bash
uv pip install -r requirements.txt
```

---

# 🔑 Environment Variables

Create a `.env` file in the root directory:

```env
GROQ_API_KEY=your_groq_api_key
TAVILY_API_KEY=your_tavily_api_key
```

Add `.env` to `.gitignore`:

```gitignore
.env
.venv/
__pycache__/
```

---

# ▶️ Running ResearchMind

Run:

```bash
uv run pipeline.py
```

The application will ask:

```text
Enter a research topic:
```

For example:

```text
Enter a research topic: Impact of Generative AI on Software Engineering
```

ResearchMind will then execute:

```text
==================================================
Step 1 - Search agent is working...
==================================================

Search Results...

==================================================
Step 2 - Reader agent is scraping top resources...
==================================================

Scraped Content...

==================================================
Step 3 - Writer is drafting the report...
==================================================

Final Report...

==================================================
Step 4 - Critic is reviewing the report...
==================================================

Critic Feedback...
```

---

# 🔄 Research Pipeline State

ResearchMind maintains a simple state dictionary while executing the pipeline.

Conceptually:

```python
state = {
    "search_result": "...",
    "scraped_content": "...",
    "report": "...",
    "feedback": "..."
}
```

Each stage contributes information to the state.

```text
{}
 │
 ▼
{ search_result }
 │
 ▼
{ search_result,
  scraped_content }
 │
 ▼
{ search_result,
  scraped_content,
  report }
 │
 ▼
{ search_result,
  scraped_content,
  report,
  feedback }
```

This makes it easy to inspect what each stage produced.

---

# 🤖 Why Multi-Agent Architecture?

A single LLM could theoretically perform all of these operations.

ResearchMind intentionally separates responsibilities.

```text
                  ResearchMind
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    Research        Writing        Evaluation
        │              │              │
 Search + Reader      Writer         Critic
```

This provides several advantages.

**Separation of responsibilities:** each component has a specific task.

**Modularity:** individual agents or tools can be changed without redesigning the entire application.

**Debugging:** failures can be traced to a particular stage.

**Extensibility:** additional agents can be introduced later.

**Better workflow design:** the project resembles a real research process rather than a single prompt-response interaction.

---

# 🔮 Future Improvements

ResearchMind can be extended significantly.

Possible improvements include:

- multiple-source webpage reading
- source credibility scoring
- citation generation
- parallel research agents
- structured report export to PDF
- persistent research history
- Streamlit or React frontend
- database integration
- vector database support
- RAG-based document research
- human approval before report generation
- automatic retry and rate-limit handling
- report revision based on critic feedback

A particularly useful next improvement would be:

```text
Search
   ↓
Reader
   ↓
Writer
   ↓
Critic
   ↓
Revision Agent
   ↓
Final Improved Report
```

Currently, the Critic provides feedback. A revision stage could send that feedback back to the Writer so that the final report is automatically improved.

---

# 📌 Example Use Cases

ResearchMind can be useful for:

- academic topic exploration
- technology research
- market and industry research
- software engineering research
- AI trend analysis
- preliminary literature exploration
- gathering information for reports
- learning multi-agent AI system design

---

# ⚠️ Limitations

ResearchMind is an experimental AI research project.

Generated reports may contain incorrect or incomplete information. Web sources can also vary in quality.

For academic, financial, medical, legal, or other high-stakes research, information should always be verified against authoritative sources.

The system may also encounter API rate limits depending on the Groq or Tavily usage tier.

---

# 🎓 Learning Goals

ResearchMind demonstrates several important concepts in modern Generative AI development:

- LLM integration
- AI agents
- tool calling
- multi-agent workflows
- web search integration
- web scraping
- prompt engineering
- state management
- LLM chains
- agent orchestration

The project is designed not only to generate research reports but also to demonstrate how multiple AI components can collaborate to solve a larger problem.

---

# 📄 License

This project is intended for educational and experimental purposes.

---

# ⭐ ResearchMind

```text
Search the web.
Read deeper.
Write intelligently.
Critique the result.
```

**ResearchMind — A Multi-Agent AI Research Assistant.**