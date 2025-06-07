# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands
- Install dependencies: `poetry install`
- Run the agent: `python -u -m agentq`
- Run tests: `python -m test.tests_processor --orchestrator_type fsm`
- Run a specific test: `python -m test.run_tests -min <task_index> -max <task_index+1>`
- Run MCTS generation: `python -m agentq.core.mcts.browser_mcts`
- Start Flask API server: `python server.py` (runs on port 8000)
- Linting: `ruff check .`

## Architecture Overview

### Core Agent System
AgentQ uses a state-driven orchestrator pattern with specialized agents:
- **BaseAgent**: Common LLM interface with structured I/O using Pydantic models
- **Orchestrator**: FSM managing agent transitions and memory state
- **Agent Types**: AgentQ (main), Actor/Critic (action generation/evaluation), Planner, BrowserNav, Vision
- **State Management**: Memory object tracks objectives, completed tasks, current plan, and agent queues

### MCTS Integration
The system combines Monte Carlo Tree Search with agent architectures:
- **BrowserWorldModel**: State representation (DOM + URL + objectives) with async action execution
- **Actor-Critic MCTS**: Actor proposes actions, Critic ranks them for UCB1 selection
- **Skills System**: Modular web interaction capabilities (click, type, navigate, screenshot, etc.)
- **DPO Data Generation**: Automatic preference pair creation for reinforcement learning

### Key Components
- **PlaywrightManager**: Singleton browser management with persistent context
- **Skills Library**: Reusable web automation functions in `agentq/core/skills/`
- **DOM Processing**: Accessibility tree extraction with smart content filtering
- **Memory System**: Long-term memory (LTM) for user preferences and learning

### Agent Communication Flow
1. Orchestrator routes to appropriate agent based on current state
2. Agents use BaseAgent.run() for LLM calls with tool/function support
3. Memory object passed between agents maintains conversation context
4. State transitions trigger different agent workflows (PLAN → BROWSE → AGENTQ_ACTOR → AGENTQ_CRITIC)

## Code Style Guidelines
- Python version: >=3.10
- Use Python type hints for function parameters and return values
- Follow import ordering: standard library, third-party, local modules
- Document functions with docstrings in the format seen in utils/logger.py
- Error handling should use try/except with specific exceptions
- Use Pydantic models for structured data validation
- Async/await for asynchronous operations
- Prefer composition over inheritance where possible

## Development Notes
- All agents inherit from BaseAgent and use structured input/output models
- MCTS implementation is in `agentq/core/mcts/` with browser-specific world model
- Skills are atomic web actions that can be composed into complex behaviors
- Testing framework supports parallel execution with JSON task definitions
- Flask API provides HTTP endpoints for remote agent invocation

