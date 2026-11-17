# Case Study 4: Simulating Societies with LLM Agents

## Course Guide

> **Working in pairs.** You are expected to work in dyads, but each student submits individually and will be assessed on their own understanding.

---

## Overview

You will run a generative agent simulation — three LLM-powered characters living and working in a café — and investigate how small changes to their personality descriptions and memory architecture produce large, sometimes surprising, changes in behaviour.

You will need to install Go to run the simulation server. **You will not need to write any Go code.** Everything you edit is either natural-language text files (personality descriptions, prompt templates) or Python (analysis scripts).

### What you will submit

1. A completed **answer template** (`answer_template.md`) with your analysis, log excerpts, plots, and written reflections.
2. Your **modified files** (scratch files, prompt edits).
3. Your **analysis code**.

---

## Step 0: Setup

### 0.1 Clone the repository

You are working in this repository. It contains everything you need (simulation server, maze data, the starting agents), so you do not need any other repository.

```
git clone https://github.com/DorotaMoravcikova/agentic-llm-course-excercises
cd agentic-llm-course-excercises
```

Run all commands in this guide from the root of this repository.

### 0.2 Install Go

Download from [go.dev/dl](https://go.dev/dl/). Verify with:

```
go version
```

You need Go 1.24.3 or later. This is only needed to run the simulation server — you will not write any Go code.

### 0.3 Install Python dependencies

You need Python 3.10+ for the analysis exercises.

```
pip install pandas matplotlib scipy
```

### 0.4 Configure your LLM

Copy `.env.example` to `.env` and fill it in. The [README](../README.md#configuration) explains every variable. The `.env.example` defaults already match the layout of this repository, so for a first run you only need to add your keys (and, for a local model, the URLs and model names).

You need a **text model** and an **embeddings model**, and they are configured separately (`TEXT_MODEL_*` and `EMBEDDING_*`). The table below lists text models only. Check that your provider offers an embeddings endpoint; if it does not, use OpenAI or a local server such as Ollama for embeddings only. Use the same embeddings model for every run you want to compare.

You need **two different LLMs** for this assignment. We recommend choosing one commercially hosted and one local/open model so you gain hands-on experience with running both.

| Option | Type | Cost | GPU needed? |
|--------|------|------|-------------|
| GPT-4o-mini (or later) | Commercial API | Low | No |
| Claude Haiku | Commercial API | Low | No |
| Llama 3 8B via Ollama | Local | Free | Yes |
| Qwen 2.5 7B via Ollama | Local | Free | Yes |
| vLLM / LM Studio | Local | Free | Yes |

The system requires an **OpenAI API-compatible endpoint**. When using OpenAI directly, the backend uses the Responses API; for any other provider it falls back to the Chat Completions API.

**No GPU?** Use a commercial API — the cost for all exercises combined should be under €2. Alternatively, use the LIACS DS-lab or the machines in the Gorlaeus computer rooms.

### 0.5 Start the simulation

**A run overwrites the simulation folder it loads.** After every step the server writes the agents' memories, scratch files and the clock back into `simulation/<SIMULATION_NAME>/`. If you start again with the same `SIMULATION_NAME`, the run continues from where the last one stopped, with all its memories. So for every new experiment, start from a fresh copy and keep `simulation/base_cafe_spiral/` untouched as your pristine original:

1. Make a copy with a name for this run (macOS/Linux: `cp -r simulation/base_cafe_spiral simulation/ex1_bernard_v1`; on Windows copy the folder in File Explorer or use `Copy-Item -Recurse`).
2. Edit the scratch files **inside the copy** (see 0.6).
3. In `.env`, set `SIMULATION_NAME=ex1_bernard_v1` (your copy's name).
4. Start the server:

```
go run ./simulation_server
```

Logs are written to `logs/<SIMULATION_NAME>/<timestamp>_<id>/run.jsonl`, a new folder for every start, so runs do not overwrite each other's logs. Copies other than `base_cafe_spiral` are gitignored, so if you submit a repository link, zip your run folders and logs and include them.

**How long to run.** Where this guide says "run for N minutes", it means N minutes of in-game time after the cafe opens. The in-game clock starts at midnight on 13 February 2023. The agents plan their day, the server fast-forwards through the night until about 07:00 (the exact moment depends on the wake-up hour the LLM picks), and the cafe opens at 08:00. If you counted 30 minutes from the start of the run you would stop at about 07:30, before anyone is at the cafe.

The server has no stop-time option and stops by itself only after two in-game days, so you stop it yourself. Watch the console: every step prints a `step_start` line with a `sim_time=...` value. For a **30-minute run, stop it with Ctrl+C when `sim_time` reaches about 08:30**, and for a longer run when it reaches the corresponding later time (for example 12:00 for four hours). State is saved after every step, so stopping is safe; if a run ever fails to load after being interrupted, delete that run folder and start again from a fresh copy. To extend a run, start again with the same `SIMULATION_NAME`; it continues from the last saved step.

### 0.6 Key files

**Personality descriptions (scratch files):**

The originals are in `simulation/base_cafe_spiral/`. Edit the scratch files in your run copy, `simulation/<run_name>/`:
- `simulation/<run_name>/personas/Dolores Abernathy/bootstrap_memory/scratch.json`
- `simulation/<run_name>/personas/Maeve Millay/bootstrap_memory/scratch.json`
- `simulation/<run_name>/personas/Bernard Lowe/bootstrap_memory/scratch.json`

Bernard's `innate` and `learned` fields are empty strings (`""`) on purpose: you fill them in in Exercise 1.

**Agent memory (nodes):**
After running a simulation, each agent's memories are stored in:
- `<SIMULATION_DIR>/<SIMULATION_NAME>/personas/<name>/bootstrap_memory/associative_memory/nodes.json`

Where `<SIMULATION_DIR>` and `<SIMULATION_NAME>` are the values you configured in your `.env` file.

**Planning prompt template (Exercise 4):**
- `simulation_server/llm/openai/v5/task_decomp_v3/prompt.txt`

Read all three scratch files before starting the exercises.

---

## Exercise 1: Shaping Agent Personalities

**Goal:** Discover how different prompting styles affect agent behaviour, and how fragile persona descriptions are.

**Time estimate:** 90–120 minutes.

### Background

Each agent's personality is defined in a `scratch.json` file with two key fields:

* `innate`: a comma-separated list of short trait adjectives (e.g., "warm, empathetic, detail-oriented")
* `learned`: free-text sentences describing the agent's background, habits, and behavioural patterns

Dolores and Maeve (the two baristas) come with pre-written personalities. Do not modify them in this exercise; they serve as your baseline.

Bernard (the café manager) has been left with empty `innate` and `learned` fields. Your task is to fill them in to make Bernard a difficult, confrontational boss.

### Part A: Making Bernard Mean

1. **Write your first version of Bernard's personality.** Choose whatever approach feels natural: adjectives, sentences, backstory, specific example behaviours, or a mix. Run the simulation for 30 minutes (see step 0.5 for what this means and how to stop the run), in a fresh copy of `base_cafe_spiral`, and inspect the conversation logs.

2. **Write at least two more versions**, each using a different prompting style. For example:
   * Abstract trait adjectives only (e.g., "harsh, impatient, critical")
   * Full sentences describing behavioural patterns (e.g., "Bernard rarely offers praise and often interrupts his employees mid-sentence")
   * Concrete example dialogue or actions (e.g., "Bernard says he has never seen an employee underperform like this before")

   Run each version for 30 minutes, each in its own fresh copy of `base_cafe_spiral` (step 0.5), and keep the logs separate.

3. **Compare.** Which version produced the most confrontational Bernard? Which produced the least? Look at the actual conversations in the logs and ask yourself if Bernard's behaviour matches what you wrote?

4. **Report** in your answer template:
   * All versions of Bernard's `innate` and `learned` fields
   * 2–3 conversation excerpts per version
   * Your analysis of which prompting style was most effective and why

### What to look for

LLMs are instruction-tuned to be helpful, harmless, and polite. This training fights against a personality description that asks the agent to be unkind. The gap between the personality you wrote and the behaviour you observe is a direct demonstration of the tension between instruction tuning and persona prompting. Park et al. themselves noted that their agents were "overly cooperative" — you are now seeing why.

### What to save

- All versions of Bernard's scratch file
- 2–3 conversation excerpts per version
- Your written analysis in the answer template

---

## Exercise 2: Comparing Models

**Goal:** See how the choice of LLM affects agent behaviour, independent of architecture.

**Time estimate:** 60–90 minutes.

### What to do

1. **Pick one of your Bernard personalities from Exercise 1.** Use the same version for both models.

2. **Run the simulation with Model A** (e.g., GPT-4o-mini) for 30 minutes of in-game time (step 0.5), in a fresh copy of `base_cafe_spiral` that contains your chosen Bernard. Save the logs to a clearly named folder (e.g., `logs_model_a/`).

3. **Run the same simulation with Model B** (e.g., Llama 3 via Ollama) for 30 minutes, in a second fresh copy with identical scratch files. Do not reuse Model A's folder: it contains Model A's memories. Keep the embeddings model the same, so that only the text model differs. Save logs to `logs_model_b/`.

4. **Read the conversation logs qualitatively.** Note differences in tone, vocabulary, and interaction patterns.

5. **Compare valence trajectories.** Plot valence over time for both models. Does one model produce more emotional variance than the other?

6. **Count JSON validation failures.** Check the simulation server logs for parsing errors. Record the count for each model.

7. **Answer the questions** in the answer template.

### What to look for

The architecture, the personality descriptions, and the world are identical. Any difference in behaviour comes from the LLM itself — its training data, its instruction tuning, its inherent biases. This exercise demonstrates that generative agent simulations are not deterministic systems: the model underneath is not a neutral execution engine, it is an opinionated one.

### What to save

- Logs from both runs
- Output of your analysis (plots, statistics)
- Your written comparison in the answer template

---

## Exercise 3: Enabling the Negativity Bias

**Goal:** Compare baseline vs NEVER-augmented architecture and observe emergent effects.

**Time estimate:** 90–120 minutes (this is the core exercise).

### What to do

1. **Start from a fresh copy** of `simulation/base_cafe_spiral` (step 0.5). Do not continue from an earlier run's folder: it contains that run's memories. In the copy, Dolores and Maeve stay exactly as provided, and Bernard gets the personality you used in Exercise 2 (copy your `innate` and `learned` text into his scratch file). Use this same Bernard for every run in this exercise.

2. **Enable the negativity bias** for Dolores: 
   - Valence-weighted retrieval (the V term in the retrieval score, with β = 1.5 for negative memories)
   - Asymmetric sensory encoding (expanded descriptions for events with valence ≤ −3)

   Maeve stays on the standard architecture.

3. **Run the simulation** for at least 30 minutes of in-game time (step 0.5). Save logs as `logs_negbias/`. If you can, run longer (until about 12:00 to 14:00 in-game, i.e. four to six hours after the cafe opens) — the effects become more pronounced over time.

4. **Quantitative analysis.** Using Python:
   - Plot Dolores's and Maeve's valence trajectories on the same chart.
   - Compute mean valence and standard deviation for each agent.
   - Compute Cohen's d for the difference.

5. **Thought description comparison.** Find 2–3 of Dolores's thought descriptions for negative events and compare their length and content to Maeve's for comparable events. Are Dolores's descriptions longer? Do they contain sensory language (e.g., physical descriptions of how interactions felt)?

6. **Check for memory intrusion.** If your simulation ran long enough to include off-duty hours (the agents leave work at 16:00, so this needs a run that reaches the early evening), inspect Dolores's reflections from that period. Do work memories intrude — references to Bernard, the café, or workplace events? Compare with Maeve's evening reflections. If you find examples, paste them into your answer template.

7. **(Optional but recommended)** Run the simulation twice more to check consistency.

8. **Answer the questions** in the answer template.

### What to look for

You enabled two changes: negative memories get higher retrieval scores, and negative events get stored with more detail. Looking at your logs, can you trace how these two changes could lead to the valence decline you observe? Think about what happens when a negatively-toned reflection gets stored back into the memory stream as a new memory — it is itself negative, so it scores high in retrieval, so it surfaces again in the next reflection, and so on.

### What to save

- Configuration showing the negativity bias is enabled
- Logs from your run(s)
- Valence trajectory plot
- Effect size computation
- Thought description examples
- Any memory intrusion examples you found
- Your written analysis in the answer template

---

## Exercise 4: Daily Plan Quality

**Goal:** Improve agent behaviour through prompt engineering.

**Time estimate:** 45–60 minutes.

### What to do

1. **Inspect daily plans.** From any of your previous runs, find the daily plans generated for each agent at the start of each simulated day. Look at the log entries that show the agent's intended schedule.

2. **Identify problems.** Common issues:
   - Single activities lasting 3–4 hours with no breaks
   - Unrealistic or missing meal times
   - Vague activity descriptions ("work on things")
   - No transitions between locations

3. **Write planning rules.** Create a set of 3–5 natural language rules that should improve plan quality. For example:
   - "No single activity should last longer than 90 minutes without a break."
   - "Include at least two meal breaks (morning, lunch)."
   - "Each activity should specify a location."

4. **Inject your rules.** Open `simulation_server/llm/openai/v5/task_decomp_v3/prompt.txt` and add your rules to the prompt text. You are editing natural language, not code.

   **Important:** The output format specified in the prompt **must stay exactly the same.** Only modify the instructions, not the expected response structure.

   **Tip:** The fields available in the prompt template use Go's text/template syntax (e.g., `{{ .Persona.Name }}`). To see all available fields, check the `TaskDecompV3Input` struct in `simulation_server/llm/openai/types.go` and the `Persona` interface in `simulation_server/llm/llm.go`.

5. **Run the simulation** for 30 minutes (step 0.5, in a fresh copy of `base_cafe_spiral`) with your improved planning prompt. Compare the generated plans to the originals.

6. **Assess downstream effects.** Do agents with better plans have more social interactions? More varied activities? Different valence trajectories?

7. **Answer the questions** in the answer template.

### What to look for

Plan quality cascades through the entire simulation. A plan that has the agent sitting at a desk for four hours straight means four hours without social interaction, without new memories, without the chance for emergent behaviour. Better plans create more opportunities for the architecture to produce interesting dynamics.

### What to save

- Your planning rules (the text you added to the prompt)
- Before/after examples of daily plans
- Your written analysis of downstream effects in the answer template

---

## Optional Extensions

If you want to go further, consider the following:

- **Vary the bias strength.** Run the simulation with different values of the negative memory multiplier (1.0, 1.25, 1.5, 2.0, 3.0). Plot mean valence as a function of the multiplier. Is the relationship linear? Is there a threshold beyond which the agent spirals?

- **Remove Bernard.** Run the negativity bias condition without the stressor agent. Does the negativity bias still produce a valence shift in a benign environment, or does it require negative events to seed the cycle?

- **Scale up.** Increase the number of agents to 5–10. Do group dynamics emerge? Do you see coalition formation, social isolation, or other multi-agent phenomena? How does the negativity bias interact with group size?

- **Implement another cognitive bias.** Choose one (e.g., confirmation bias, availability heuristic, recency bias) and implement it architecturally. Run the comparison. Does it produce the effect you predicted?

---

## Submission Checklist

Before submitting, verify you have:

- [ ] Your name on the answer template
- [ ] Completed `answer_template.md` with all questions answered
- [ ] All versions of Bernard's scratch file (Exercise 1)
- [ ] Modified planning prompt (Exercise 4)
- [ ] Analysis code
- [ ] Log files from your runs
- [ ] The run folders you used (zipped), if you submit a repository link: copies of `simulation/base_cafe_spiral` are gitignored
- [ ] At least one valence trajectory plot (Exercise 3)
- [ ] Effect size computation (Exercise 3)

Submit as a single zip file or as a link to your forked repository.
