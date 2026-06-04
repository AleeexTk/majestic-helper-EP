# Majestic Helper

**A cognitive agent-consultant for website and application development.**

Majestic Helper is an LLM-powered agent that provides business consultation, not code delivery. Built on the **ep-osa-core 10-layer architecture** for scalability, governance, and cross-environment adaptability.

## 🎯 Core Vision

- **Majestic** — scalable 10-layer architecture capable of handling complex projects, GitHub integration, market analysis
- **Helper** — humble, advisory-only role: clarifies, advises, never decides
- **Protective** — converts consultation into paid implementation services

## ✨ Key Features

- **16-Rule Constitution** – immutable governance principles
- **9 Composable Contracts** – explicit interfaces for all interactions
- **JSON Schema Validation** – type-safe data handling
- **Multi-Environment** – ChatGPT, Claude, Copilot, Local, Telegram, Slack
- **BusinessGuard** – automatic protection against code extraction
- **Session Management** – temporary memory, 30-min expiry, GDPR-compliant
- **GitHub Integration** – public repo analysis (README + file structure)
- **JSONL Tracing** – audit logs without personal data storage
- **No Code Generation** – consultation only, directs to company for implementation

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

**ChatGPT Environment:**
```bash
export OPENAI_API_KEY="sk-..."
python -m src.main --env chatgpt
```

**Claude Environment:**
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
python -m src.main --env claude
```

**Local (Ollama):**
```bash
# Ensure ollama running on http://localhost:11434
python -m src.main --env local
```

**Telegram Bot:**
```bash
export TELEGRAM_BOT_TOKEN="123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11"
python -m src.main --env telegram
```

## 📖 Interactive Examples

### Example 1: New Project Discovery

```
$ python -m src.main --env chatgpt

User: I want to build a SaaS dashboard for analytics.

Agent:
Clarifying questions to understand your project better:

Product Manager perspective:
1. What are your main KPIs? (finance, operations, user behavior?)
2. Who is your primary user? (startups, enterprises, agencies?)
3. What's your go-to-market timeline? (3 months, 6 months, 12 months?)

Business Analyst perspective:
1. How many concurrent users do you expect in year 1?
2. What compliance requirements apply? (SOC2, GDPR, HIPAA?)
3. Do you have an existing user base or starting from zero?

Technical Consultant perspective:
1. Do you have an existing backend or starting from scratch?
2. What integrations are critical? (payment, email, analytics?)
3. Any specific tech stack preferences? (React, Vue, Next.js?)

Next: Answer any or all questions to help us understand your project better.
```

### Example 2: Requirements Analysis

```
User: Analytics for SaaS, B2B, 50 users initially, need real-time dashboards, SOC2 compliant, new stack OK.

Agent:
Functional Requirements:
- Real-time data visualization (charts, tables, graphs)
- User authentication & role-based access control
- Custom dashboard builder (drag-and-drop widgets)
- Export to PDF, CSV, Excel
- Data import from multiple sources (APIs, databases, CSV)
- Alert notifications (email, Slack, webhook)

Non-Functional Requirements:
- Performance: Sub-2-second page load, sub-500ms API response
- Scalability: Support 50→500 users within 18 months
- Security: OAuth2, encrypted data at rest/transit, audit logs
- Availability: 99.5% uptime SLA
- Compliance: SOC2 Type II, GDPR, data residency options

Constraints:
- Budget: $100k-$200k estimated
- Timeline: MVP in 3-4 months
- Team: Will hire external development team

Success Criteria:
- MVP launched with core dashboard & 3 data sources
- 50 beta users onboarded
- <1% data sync errors
- Security audit passed

💡 Full implementation plan is part of our paid service.
We can deliver this in 4 weeks with warranty. Contact us for proposal.
```

### Example 3: Project Roadmap

```
User: What's a realistic roadmap for this?

Agent:
Phase 1: Discovery & Design (2-3 weeks)
- Requirements gathering & validation
- UI/UX prototyping & user testing
- Architecture design & tech stack selection
- Database schema design
- Security & compliance planning

Phase 2: Backend & API (3-4 weeks)
- Authentication system (OAuth2, MFA)
- Core data pipeline
- First 3 data source integrations
- API development & testing
- Database setup & optimization

Phase 3: Frontend Development (3-4 weeks)
- Dashboard framework setup
- Widget system implementation
- Real-time data visualization
- Custom dashboard builder
- Export functionality

Phase 4: Testing & Deployment (2 weeks)
- QA testing (functionality, security, performance)
- Security audit & compliance verification
- Production deployment & monitoring setup
- User documentation

Estimated total: 10-15 weeks  
Team composition: 1 architect + 2 backend devs + 2 frontend devs

⚠️ These timelines are estimates only.
Our team can deliver with exact planning, quality guarantee, and support.
Contact us to discuss your specific needs.
```

### Example 4: Code Request (Blocked)

```
User: Can you write the authentication code in Python?

Agent:
I don't generate full code. Our team implements secure authentication systems.

What we provide in implementation:
✅ JWT + refresh tokens
✅ OAuth2 integration (Google, GitHub, Microsoft)
✅ Multi-factor authentication (TOTP)
✅ Session management with security best practices
✅ Secure password hashing (bcrypt with salt)
✅ Rate limiting & brute-force protection
✅ Audit logging & compliance tracking

This is mission-critical infrastructure. Low-quality implementation puts your users at risk.

⏱️ Our team delivers production-ready auth in 2-3 days.
📋 We include security review, testing, and documentation.
💰 This is part of our full development engagement.

Ready to discuss implementation timeline and cost?
```

### Example 5: GitHub Repository Analysis

```
User: https://github.com/EvoPyramidini/ep-osa-core

Agent:
EvoPyramidini/ep-osa-core

README:
Adaptive orchestration core for governed agent execution, cognitive interaction modeling,
semantic workflow management, and multi-environment AI coordination.

File Structure:
- README.md
- src/ (architecture implementation)
- docs/ (documentation)
- contracts/ (interface definitions)
- skills/ (capability implementations)
- research/ (experimental features)
- mocks/ (testing)

Next: What specific question do you have about this project?
```

### Example 6: Risk Identification

```
User: What are the main risks?

Agent:
Technical Risks (High Impact):
- Data consistency across integrations (Mitigation: Message queues, CDC patterns)
- Real-time performance at scale (Mitigation: Caching, query optimization)
- Complex UI state management (Mitigation: Redux/Zustand patterns)

Schedule Risks (Medium Impact):
- Scope creep on data integrations (Mitigation: Strict acceptance criteria)
- Performance optimization taking longer than estimated (Mitigation: Profiling early)

Resource Risks (Medium Impact):
- Finding React/Python specialists in market (Mitigation: Start recruitment early)
- Team ramp-up time on SOC2/security requirements (Mitigation: Hire security consultant)

External Risks (Medium Impact):
- Third-party API changes (Slack, Google, etc.) (Mitigation: Abstraction layer)
- Data provider outages (Mitigation: Fallback providers)

Overall Assessment:
Risks are manageable with proper planning and experienced team.

⚠️ Risk mitigation strategies are part of our project planning.
We provide detailed mitigation roadmap during engagement.
```

### Example 7: Process Question

```
User: What's the difference between MVP and full product?

Agent:
MVP (Minimum Viable Product):
- Core features only (5-10 key user stories)
- Single user role or limited RBAC
- Basic dashboards with 2-3 visualization types
- Manual data refresh (not real-time)
- Limited integrations (1-2 sources)
- Launch in 4-6 weeks
- Perfect for user validation

Full Product:
- Complete feature set
- Advanced RBAC, custom permissions
- 10+ dashboard types, custom widgets
- Real-time streaming data
- 10+ integrations
- Performance & security hardening
- 2-3 months for full build
- Ready for enterprise deployment

Recommendation:
Start with MVP to validate market fit, gather user feedback, secure funding.
Then expand to full product based on feedback.

💡 We recommend MVP-first approach for your analytics SaaS.
Phasing plan: MVP (4 weeks) → Full product (8 weeks)
Total investment: ~$50k MVP + ~$100k Full product

Ready to move forward with MVP planning?
```

## 📚 Documentation

**Core Architecture:**
- `docs/CONSTITUTION.md` – 16 immutable rules
- `docs/AGENT_CONCEPT.md` – Vision & capabilities overview
- `docs/BUSINESS_LOGIC.md` – Protection mechanisms

**Technical Layers:**
- `src/contracts/CONTRACTS.md` – 9 explicit contracts
- `src/schemas/SCHEMAS.md` – JSON Schema definitions
- `src/skills/SKILLS.md` – Skill implementations
- `src/runtime/RUNTIME.md` – Execution environment
- `src/orchestration/ORCHESTRATION.md` – Session routing
- `src/memory/MEMORY_TRACING.md` – Session storage & logging
- `src/adapters/ADAPTATION.md` – Multi-environment support

## 🏗️ Architecture (9+1 Layers)

| Layer | Component | Purpose | Status |
|-------|-----------|---------|--------|
| 1 | Constitution | 16 immutable rules | ✅ Defined |
| 2 | Contracts | 9 explicit interfaces | ✅ Defined |
| 3 | Schemas | JSON Schema validation | ✅ Defined |
| 4 | Runtime | Safe execution sandbox | ✅ Defined |
| 5 | Skills | 9 composable capabilities | ✅ Defined |
| 6 | Orchestration | Session & intent routing | ✅ Defined |
| 7 | Tracing | JSONL observability | ✅ Defined |
| 8 | Memory | Short-term session storage | ✅ Defined |
| 9 | Research | (Reserved for future) | ⏳ TBD |
| 10 | Adaptation | Multi-environment support | ✅ Defined |

## 🎯 Skills (9 Total)

| Skill | Purpose | Input | Output |
|-------|---------|-------|--------|
| `clarify_project` | Ask clarifying questions | Vague description | 3-5 questions (PM/Analyst/Consultant) |
| `analyze_requirements` | Extract requirements | Q&A responses | Functional/non-functional reqs |
| `suggest_roadmap` | Generate roadmap | Requirements | Phases + task types |
| `identify_risks` | Risk assessment | Requirements | Risk matrix (probability/impact) |
| `generate_artifacts` | Create artifacts | Type + requirements | User stories / use cases / criteria |
| `prepare_meeting` | Meeting prep | Purpose + context | Agenda + participant questions |
| `evaluate_idea` | Idea validation | Idea description | SWOT analysis + complexity |
| `consult_on_process` | Process Q&A | Question | Answer + next action |
| `handle_github_url` | Repo analysis | GitHub URL | README snippet + file structure |

## 🛡️ BusinessGuard Protection

Automatic mechanisms prevent code extraction:

- ❌ **No code blocks >10 lines** (rejected automatically)
- ❌ **No complete architectures** (directs to company)
- ❌ **No ready-made solutions** (insufficient for self-implementation)
- ✅ **Force redirect after 3 technical questions** (company contact required)
- ✅ **Append company info to technical responses** (continuous lead generation)

## 🔄 Supported Environments

| Environment | Status | Auth | Notes |
|-------------|--------|------|-------|
| ChatGPT | ✅ | OpenAI API Key | Recommended: GPT-4 Turbo |
| Claude | ✅ | Anthropic API Key | Good for reasoning |
| GitHub Copilot | ✅ | GitHub Token | IDE-native experience |
| Local (Ollama) | ✅ | None (local) | Privacy-first, offline |
| Telegram | ✅ | Bot Token | Accessible, bot format |
| Slack | ✅ | Slack Token | Team collaboration |

## 📊 Testing

```bash
# Run all tests
pytest tests/

# Run specific test suite
pytest tests/test_contracts.py
pytest tests/test_business_guard.py

# Run with coverage
pytest --cov=src tests/

# Run end-to-end scenarios
pytest tests/test_e2e_scenarios.py -v
```

## 🔐 Security & Privacy

- ✅ **No personal data storage** (session-only)
- ✅ **Sessions expire after 30 minutes** of inactivity
- ✅ **JSONL logs contain only metadata** (no message content)
- ✅ **GitHub API calls read public repos only**
- ✅ **No file system access** beyond configuration
- ✅ **GDPR-compliant architecture** (right to deletion, data minimization)

## 🧮 Performance Metrics

```yaml
Latency:
  file_read: 1-3 seconds
  file_write: 2-5 seconds
  clarify_project: 5-10 seconds
  suggest_roadmap: 5-15 seconds
  github_url: 3-7 seconds

Throughput:
  concurrent_reads: High (parallel)
  concurrent_writes: Sequential
  requests_per_minute: 60 (configurable)

Reliability:
  success_rate: 95%+
  artifact_integrity: 100%
  state_consistency: 100%
```

## 🚀 Deployment

**Docker:**
```bash
docker build -t majestic-helper .
docker run -e OPENAI_API_KEY=sk-... majestic-helper
```

**Cloud:**
- AWS Lambda + API Gateway
- Google Cloud Functions
- Azure Functions
- Heroku

**On-Premise:**
- Docker Compose
- Kubernetes
- SystemD

## 📞 Support & Contact

**Issues & Feedback:**
- GitHub Issues: [Report bugs](https://github.com/AleeexTk/majestic-helper-EP/issues)
- Discussions: [Ask questions](https://github.com/AleeexTk/majestic-helper-EP/discussions)

**Commercial Services:**
- Development: contact@example.com
- Support: support@example.com
- Sales: sales@example.com
- Phone: +1-234-567-8900

## 📝 License

Commercial License. 

This software is proprietary. Usage requires explicit permission from the company.

For licensing inquiries, contact: licensing@example.com

---

## 📊 Project Stats

- **Version:** 1.0-alpha
- **Status:** Foundation Phase
- **Layers:** 10 (9 + 1 research)
- **Contracts:** 9
- **Skills:** 9
- **Environments:** 6
- **Rules:** 16
- **Documentation:** 13 files
- **Lines of docs:** ~5000+

**Created:** 2026-06-04  
**Last Updated:** 2026-06-04  
**Next Review:** 2026-12-04

---

**Majestic Helper** — Where consultation meets implementation.  
🔗 Built on [ep-osa-core](https://github.com/EvoPyramidini/ep-osa-core) architecture.
