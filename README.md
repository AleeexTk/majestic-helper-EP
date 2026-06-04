# Majestic Helper

**A cognitive agent-consultant for website and application development.**

Majestic Helper is an LLM-powered agent that provides business consultation, not code delivery. Built on the **ep-osa-core 10-layer architecture** for scalability, governance, and cross-environment adaptability.

## 🎯 Core Vision

- **Majestic** — scalable 10-layer architecture capable of handling complex projects, GitHub integration, market analysis
- **Helper** — humble, advisory-only role: clarifies, advises, never decides
- **Protective** — converts consultation into paid implementation services

## ✨ Features

- **16-Rule Constitution** – immutable governance principles
- **9 Composable Contracts** – explicit interfaces for all interactions
- **JSON Schema Validation** – type-safe data handling
- **Multi-Environment** – ChatGPT, Claude, Copilot, Local, Telegram, Slack
- **BusinessGuard** – automatic protection against code extraction
- **Session Management** – temporary memory, 30-min expiry
- **GitHub Integration** – public repo analysis (README + file structure)
- **JSONL Tracing** – audit logs without personal data storage

## 🚀 Quick Start

### Installation

```bash
git clone https://github.com/AleeexTk/majestic-helper-EP
cd majestic-helper-EP
pip install -r requirements.txt
```

### Configuration

```bash
cp config.example.yaml config.yaml
# Edit config.yaml with your API keys
```

### Run

```bash
# ChatGPT environment
python run.py --env chatgpt

# Local environment (Ollama)
python run.py --env local

# Telegram bot
python run.py --env telegram
```

## 📁 Repository Structure

```
majestic-helper-EP/
├── docs/                          # Layer 1: Constitution & Framework
│   ├── CONSTITUTION.md            # 16 immutable rules
│   ├── AGENT_CONCEPT.md           # Vision & principles
│   └── BUSINESS_LOGIC.md          # Protection mechanisms
│
├── src/
│   ├── contracts/                 # Layer 2: Explicit Interfaces
│   │   ├── CONTRACTS.md           # 9 contract definitions
│   │   └── contract_registry.py   # Contract implementations
│   │
│   ├── schemas/                   # Layer 3: Data Definition
│   │   ├── SCHEMAS.md             # JSON Schema definitions
│   │   └── validator.py           # Schema validation
│   │
│   ├── runtime/                   # Layer 4: Safe Execution
│   │   ├── RUNTIME.md             # Execution environment
│   │   ├── sandbox.py             # Resource limits
│   │   └── error_handler.py       # Error isolation
│   │
│   ├── skills/                    # Layer 5: Composable Capabilities
│   │   ├── SKILLS.md              # Skill definitions
│   │   ├── clarify_project.py
│   │   ├── analyze_requirements.py
│   │   ├── suggest_roadmap.py
│   │   ├── identify_risks.py
│   │   ├── generate_artifacts.py
│   │   ├── prepare_meeting.py
│   │   ├── evaluate_idea.py
│   │   ├── consult_on_process.py
│   │   ├── handle_github_url.py
│   │   └── business_guard.py      # Protection layer
│   │
│   ├── orchestration/             # Layer 6: Session Management
│   │   ├── ORCHESTRATION.md       # Routing logic
│   │   ├── orchestrator.py        # Session & intent classification
│   │   └── intent_classifier.py   # Intent detection
│   │
│   ├── tracing/                   # Layer 7: Observability
│   │   └── logger.py              # JSONL trace logging
│   │
│   ├── memory/                    # Layer 8: Session Memory
│   │   ├── MEMORY_TRACING.md      # Memory architecture
│   │   └── session_store.py       # In-memory session storage
│   │
│   ├── adapters/                  # Layer 10: Environment Integration
│   │   ├── ADAPTATION.md          # Adapter pattern
│   │   ├── CROSS_ENV_ADAPTATION.md # Cross-environment consistency
│   │   ├── base_adapter.py        # Base adapter class
│   │   ├── chatgpt_adapter.py
│   │   ├── claude_adapter.py
│   │   ├── copilot_adapter.py
│   │   ├── local_adapter.py
│   │   ├── telegram_adapter.py
│   │   └── slack_adapter.py
│   │
│   ├── environments/              # Environment configurations
│   │   ├── chatgpt/
│   │   │   └── config.yaml
│   │   ├── claude/
│   │   │   └── config.yaml
│   │   ├── local/
│   │   │   └── config.yaml
│   │   └── telegram/
│   │       └── config.yaml
│   │
│   └── main.py                    # Entry point
│
├── tests/                         # Test suite
│   ├── test_contracts.py
│   ├── test_skills.py
│   ├── test_orchestration.py
│   ├── test_business_guard.py
│   └── test_cross_env.py
│
├── logs/                          # Trace logs (JSONL)
│   └── trace.jsonl
│
├── config.example.yaml            # Configuration template
├── requirements.txt               # Python dependencies
├── LICENSE                        # Commercial license
└── README.md                      # This file
```

## 🏗️ Architecture Layers (ep-osa-core 9+1)

| Layer | Component | Purpose |
|-------|-----------|----------|
| 1 | Constitution | 16 immutable rules |
| 2 | Contracts | 9 explicit interfaces |
| 3 | Schemas | JSON Schema validation |
| 4 | Runtime | Safe execution sandbox |
| 5 | Skills | 9 composable capabilities |
| 6 | Orchestration | Session & intent routing |
| 7 | Tracing | JSONL observability |
| 8 | Memory | Short-term session storage |
| 9 | Research | (Reserved for future) |
| 10 | Adaptation | Multi-environment support |

## 📋 Contracts (9 Skills)

1. **clarify_project** – Ask 3-5 clarifying questions (PM/Analyst/Consultant roles)
2. **analyze_requirements** – Extract functional/non-functional requirements
3. **suggest_roadmap** – Generate phase names + task types
4. **identify_risks** – SWOT analysis + probability/impact/mitigation
5. **generate_artifacts** – User stories, use cases, acceptance criteria
6. **prepare_meeting** – Meeting agenda + participant questions
7. **evaluate_idea** – Idea evaluation with complexity estimate
8. **consult_on_process** – Answer process questions (not technical implementation)
9. **handle_github_url** – Extract README + file structure from public repos

## 🛡️ BusinessGuard Protection

Automatic mechanisms prevent code extraction:

- ❌ No full code generation (>10 lines rejected)
- ❌ No complete architectures
- ❌ No ready-made solutions
- ✅ Redirects to company services after 3 technical questions
- ✅ Appends company contact info to technical responses

## 🔄 Supported Environments

| Environment | Status | Adapter |
|-------------|--------|----------|
| ChatGPT | ✅ | `chatgpt_adapter.py` |
| Claude | ✅ | `claude_adapter.py` |
| GitHub Copilot | ✅ | `copilot_adapter.py` |
| Local (Ollama) | ✅ | `local_adapter.py` |
| Telegram | ✅ | `telegram_adapter.py` |
| Slack | ✅ | `slack_adapter.py` |

## 📝 Documentation

Full documentation in `/docs` and `/src`:

- **Constitution** – `docs/CONSTITUTION.md` (16 rules)
- **Agent Concept** – `docs/AGENT_CONCEPT.md` (vision & capabilities)
- **Contracts** – `src/contracts/CONTRACTS.md` (9 interfaces)
- **Skills** – `src/skills/SKILLS.md` (implementations)
- **Business Logic** – `docs/BUSINESS_LOGIC.md` (protection mechanisms)
- **Orchestration** – `src/orchestration/ORCHESTRATION.md` (routing)
- **Runtime** – `src/runtime/RUNTIME.md` (execution)
- **Memory** – `src/memory/MEMORY_TRACING.md` (session storage)
- **Adaptation** – `src/adapters/ADAPTATION.md` (multi-environment)
- **Cross-Environment** – `src/adapters/CROSS_ENV_ADAPTATION.md` (consistency)

## 🧪 Testing

```bash
# Run all tests
pytest tests/

# Run specific test suite
pytest tests/test_business_guard.py

# Run with coverage
pytest --cov=src tests/
```

## 📊 Usage Example

```
$ python run.py --env chatgpt
(no greeting - direct processing)

> https://github.com/EvoPyramidini/ep-osa-core

EvoPyramidini/ep-osa-core
README: Adaptive Orchestration & Governed Agent Execution...
Files: README.md, src/, docs/, contracts/, skills/

Specific question?

> What roadmap would you suggest?

1. Design Phase – Define data model and API contracts
2. Development Phase – Implement core orchestration engine
3. Integration Phase – Connect all 9 layers
4. Testing Phase – Validate governance rules
5. Deployment Phase – Set up monitoring and logging

Example timeline: 3-5 weeks per phase
*This is a high-level roadmap. We build reliable solutions.*
*Contact us for exact planning and implementation.*
```

## 🔐 Security & Privacy

- ✅ No personal data storage
- ✅ Sessions expire after 30 minutes of inactivity
- ✅ JSONL logs contain only metadata (no message content)
- ✅ GitHub API calls read public repos only
- ✅ No file system access beyond configuration

## 📜 License

Commercial. Contact for licensing and support.

## 👥 Contact

**Majestic Helper Support**
- Email: contact@example.com
- GitHub: [AleeexTk/majestic-helper-EP](https://github.com/AleeexTk/majestic-helper-EP)

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
