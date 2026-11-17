# Agentic LLM Course — Case Study 4: Simulating Societies with LLM Agents

Course materials for Case Study 4 of the Agentic LLM course at LIACS, Leiden University.

<p align="center">
  <img src="our_cover.png" alt="Good Agent Gone Bad" width="80%">
</p>

This case study draws on the generative agents simulation at [fvdveen/generative_agents](https://github.com/fvdveen/generative_agents), based on the work described in *"Good Agent Gone Bad: When 'Bad is Stronger Than Good' in the Memory of Generative Agents"* (van der Veen, Moravčíková & van Duijn, 2026).

## Getting Started

1. **Read the [Course Guide](course/COURSE_GUIDE.md)** — this is your main reference. It contains setup instructions, all four exercises, and the submission checklist.
2. **Clone this repository.** Everything you need (simulation server, maze data, starting agents, course materials) is in here, so you do not need any other repository:
   ```
   git clone https://github.com/DorotaMoravcikova/agentic-llm-course-excercises
   cd agentic-llm-course-excercises
   ```
3. **Configure your LLM** (see [Configuration](#configuration) below), then follow step 0 of the Course Guide to start the simulation.
4. **Download the [Answer Template](course/answer_template.md)** — fill this in as you work through the exercises. This is what you submit.

## Contents

| Path | Description |
|------|-------------|
| [course/COURSE_GUIDE.md](course/COURSE_GUIDE.md) | Step-by-step exercise instructions, setup guide, and submission checklist |
| [course/answer_template.md](course/answer_template.md) | Template to fill in and submit with your answers, log excerpts, and plots |
| [course/rubric.md](course/rubric.md) | Grading criteria for each exercise |
| `.env.example` | Template for your configuration (copy to `.env`) |
| `simulation/base_cafe_spiral/` | The starting state of the cafe simulation (three agents). Keep it untouched and copy it for every run |
| `maze/the_spiral/matrix/` | The tile map the server loads |
| `simulation_server/` | The simulation server (Go). Prompt templates are in `simulation_server/llm/openai/v5/` |

## Requirements

- **Go 1.24.3+** (to run the simulation server — you will not write any Go code)
- **Python 3.10+** with `pandas`, `matplotlib`, `scipy`
- **Two different LLMs** (at least one commercial API, one local/open recommended)
- **An embeddings endpoint** (see below)

See the [Course Guide](course/COURSE_GUIDE.md) for full setup details.

## Configuration

The server reads its settings from a `.env` file in the repository root.

```
cp .env.example .env
```

Then edit `.env`. The variables are:

| Variable | Meaning |
|----------|---------|
| `SIMULATION_DIR` | Folder containing simulations. Default in `.env.example`: `./simulation` |
| `SIMULATION_NAME` | Which folder in `SIMULATION_DIR` to run. A run **overwrites** this folder, so use a fresh copy per run (Course Guide, step 0.5). Default: `base_cafe_spiral` |
| `MAZE_DIR` | Folder containing the maze. Default: `./maze` |
| `SIMULATION_MAZE` | Name of the maze. Default: `the_spiral` |
| `LOG_DIR` | Where logs are written (`LOG_DIR/SIMULATION_NAME/<timestamp>_<id>/run.jsonl`). Default: `./logs` |
| `TEXT_MODEL_URL` | Base URL of an OpenAI-compatible endpoint. Leave empty for OpenAI itself |
| `TEXT_MODEL_KEY` | API key for the text model |
| `TEXT_MODEL_LLM` | Name of the text model. If empty, `gpt-5-nano` |
| `EMBEDDING_URL` | Base URL of the embeddings endpoint. Leave empty for OpenAI itself |
| `EMBEDDING_KEY` | API key for the embeddings endpoint |
| `EMBEDDING_MODEL` | Name of the embeddings model. If empty, `text-embedding-ada-002` |

Things to know:

- **You need an embeddings endpoint as well as a text model.** The agents embed every memory, and the embeddings settings are separate from the text model settings: the text model's URL and key are not reused. If your text model is a local model or a provider without an embeddings API, point `EMBEDDING_*` at OpenAI or at a local server that offers embeddings (for example Ollama).
- **Use the same embeddings model for every run you want to compare.**
- With `TEXT_MODEL_URL` empty the server talks to OpenAI through the Responses API. With any URL set it uses the Chat Completions API, so the endpoint must support it, including JSON-schema structured output.
- Never commit your `.env`. It is gitignored.

Start the server from the repository root:

```
go run ./simulation_server
```
