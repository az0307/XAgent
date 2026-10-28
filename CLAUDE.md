# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**XAgent** is an open-source autonomous LLM agent. It decomposes a task with an outer **plan** loop and executes subtasks with an inner **ReAct**-style tool loop, running tools inside an isolated Docker **ToolServer**. It can run headless from the CLI or serve a web UI backed by a FastAPI server, SQL database, and WebSocket interaction layer.

Core loop: a **Dispatcher** routes work to specialized agents — plan-generate, plan-refine, tool (execution), and reflect — over a plan tree (`data_structure/`), searching with the inner-loop algorithms (`ReACT`, base search).

## Tech Stack & Components

- **Language**: Python (packaged via `setup.py`, deps in `requirements.txt`)
- **Server**: FastAPI + WebSocket (`XAgentServer/`), SQL database (`XAgentServer/database`), nginx config for deploy
- **ToolServer**: Dockerized tool execution sandbox (`ToolServer/`)
- **XAgentGen**: model/serving components for a fine-tuned XAgent model
- **XAgentWeb**: front-end web client
- **Orchestration**: `docker-compose.yml` + `dockerfiles/` + `build_images.sh`

## Repository Structure

```text
XAgent/                     # Core agent library
├── core.py                 # main orchestration entry
├── config.py               # CONFIG / ARGS loading (assets/config YAMLs)
├── agent/                  # agent implementations
│   ├── dispatcher.py       # routes tasks to the right agent
│   ├── base_agent.py
│   ├── plan_generate_agent/  plan_refine_agent/  reflect_agent/  tool_agent/
├── workflow/               # plan_exec, task_handler, reflection, working_memory, base_query
├── inner_loop_search_algorithms/  # ReACT.py, base_search.py
├── data_structure/         # node.py, plan.py, tree.py (plan tree)
├── ai_functions/           # function_manager + request backends (openai, xagent)
├── function_handler.py, toolserver_interface.py, vector_db.py, message_history.py, ...
XAgentServer/               # FastAPI app, DB, models, enums, loggers, nginx, server.py
XAgentGen/                  # fine-tuned model serving
XAgentWeb/                  # web front end
ToolServer/                 # Dockerized tool execution backend
Markdown_Docs/              # generated per-module documentation (mirrors the source tree)
run.py                      # CLI entry
start_server.py             # web server entry
command.py / command_input.py
assets/                     # config templates + static assets
tests/
```

## Commands

```bash
pip install -e .                      # install (Python; see requirements.txt)

# CLI (headless) run — task is required
python run.py --task "your task description" [--model MODEL] [--mode auto|manual] \
              [--upload-files f1 f2] [--record-dir DIR] [--quiet]

# Web server
python start_server.py

# Full stack (ToolServer + server + web) via Docker
bash build_images.sh
docker compose up
```

`run.py --mode` defaults to `auto`; `manual` enables step-through. Flags exist to bound the search: `--max-subtask-chain-length`, `--max-plan-refine-chain-length`, `--max-plan-tree-depth`, and `--enable-ask-human-for-help`.

## Configuration

- Runtime config is loaded through `XAgent/config.py` (`CONFIG`, `ARGS`) from the YAML config under `assets/` — set API keys / model endpoints there (or via env), never commit real keys.
- The ToolServer must be running/reachable for tool execution; the agent talks to it via `toolserver_interface.py`.

## Development Conventions

- This is an upstream open-source project — preserve the existing module layout and the outer-plan / inner-ReAct separation. New agent types belong under `XAgent/agent/`; new search strategies under `inner_loop_search_algorithms/`.
- `Markdown_Docs/` mirrors the source tree as documentation; keep it in sync when changing public module behavior.
- Follow `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md`. Record notable changes in `CHANGELOG.md`.
- Run `tests/` after changes to the core loop or workflow.
