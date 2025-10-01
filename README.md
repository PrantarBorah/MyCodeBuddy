## Code Builder — AI-Powered Coding Assistant (LangGraph)

An AI-powered coding assistant that operates like a small multi-agent development team. It takes a natural-language request ("Build a simple To‑Do app") and transforms it into a concrete engineering plan and implementation tasks, designed to generate a complete project file-by-file using real developer workflows.

### Architecture at a Glance
- **LangGraph StateGraph**: Orchestrates a graph of agents (nodes) and their transitions.
- **Planner Agent (`planner`)**: Converts a user prompt into a high-level project plan using a structured schema (`Plan`).
- **Architect Agent (`architect`)**: Breaks the plan into detailed implementation tasks (`TaskPlan`) with step-by-step, file-oriented actions.
- **LLM Backend**: `ChatGroq` configured with `openai/gpt-oss-120b` (modifiable). Prompts live in `agent/prompts.py`.
- **Structured Schemas**: Pydantic models in `agent/states.py` enforce reliable, typed outputs from the LLM.

### Repository Structure
- `agent/`
  - `graph.py`: Builds the LangGraph `StateGraph`, defines `planner` and `architect` nodes, compiles the agent, and shows an example invocation.
  - `prompts.py`: Prompt templates for planner and architect roles.
  - `states.py`: Pydantic models: `Plan`, `File`, `ImplementationTask`, `TaskPlan`.
- `main.py`: Example entry-point to invoke the compiled agent from the project root.
- `pyproject.toml`: Dependencies and project metadata.
- `.env`: API keys and config (not checked in).

### How It Works
1. User provides a natural-language request (`user_prompt`).
2. `planner` calls the LLM with `planner_prompt(user_prompt)` and expects a structured `Plan`:
   - App name, description, tech stack, feature list
   - Proposed files with paths and purposes
3. `architect` takes the `Plan` and emits a `TaskPlan` of `ImplementationTask`s:
   - Concrete file-level tasks; variable/function/class names; integration/ordering details
4. The graph currently wires `planner -> architect` and compiles into a callable `agent` you can `.invoke`.

### Setup
1. Python 3.10+
2. Create a virtual environment and install deps:
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip
python -m pip install -e .
```

3. Environment variables (`.env` in project root):
```
GROQ_API_KEY=your_key_here
```

### Running
Prefer running via `main.py` from the project root so module imports under `agent/` resolve consistently.

```bash
python main.py
```

If you want to change the model or prompt, edit `agent/graph.py` (`ChatGroq(model=...)`) and `agent/prompts.py`.

### PyCharm AI Debugger Tips
- Configure the run target as `main.py` (not `agent/graph.py`).
- Working directory should be the project root: `/Users/prantarborah/Downloads/PyCharm_Projects/Code_Builder`.
- If you must run `agent/graph.py` directly, add `agent/` to `sys.path` in `main.py` and import `graph` from there; otherwise `from prompts import *` / `from states import *` can fail.

Example `main.py` pattern to ensure imports resolve:
```python
from dotenv import load_dotenv
load_dotenv()

import sys
from pathlib import Path

def main():
    project_root = Path(__file__).resolve().parent
    agent_dir = project_root / "agent"
    if str(agent_dir) not in sys.path:
        sys.path.insert(0, str(agent_dir))

    from graph import agent  # compiled graph from agent/graph.py

    user_prompt = "Create a simple To-do list Web Application"
    result = agent.invoke({"user_prompt": user_prompt})
    print(result)

if __name__ == "__main__":
    main()
```

### Common Issues & Fixes
- **ModuleNotFoundError: `langchain_groq`**
  - Install deps: `python -m pip install -e .` (ensures packages from `pyproject.toml` are installed)
- **ModuleNotFoundError: `prompts` / `states` when running `agent/graph.py` directly**
  - Run via `main.py` from the project root, or adjust `sys.path` as shown above.
- **Missing `GROQ_API_KEY`**
  - Create `.env` with the key and reload environment.

### Roadmap / Next Steps
- Add an execution agent to turn `TaskPlan` steps into real file edits and generate the codebase in an output directory.
- Persist and visualize intermediate state and artifacts for debugging.
- Add retries/validation to handle partial structured outputs.

### Debugging Output File Generation (next task)
If the final output files are not being written to the output directory, we will:
- Verify where the write-to-disk logic lives (executor/writer agent or utility).
- Ensure the `TaskPlan` contains file paths and content directives.
- Add logging around file creation and directory preparation.
- Confirm permissions and existence of the target output directory.
- Re-run with PyCharm AI Debugger to inspect state transitions and outputs.

---

Built with LangGraph + LangChain + Groq. Structured outputs via Pydantic ensure reliable multi-agent plans and tasks.


