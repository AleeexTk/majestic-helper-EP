# Contracts — Explicit Interfaces (Layer 2)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Overview

All contracts follow YAML spec with:
- **Input Schema:** Expected data structure
- **Output Schema:** Guaranteed response format
- **Pre/Post Conditions:** Pre-execution + post-execution guarantees
- **Timeout:** Maximum execution time
- **Error Handling:** Expected failure modes

---

## Contract 1: `clarify_project`

**Purpose:** Ask clarifying questions for vague project descriptions

### Input
```yaml
type: object
required: [description]
properties:
  description:
    type: string
    minLength: 1
    maxLength: 5000
    description: "Initial project description"
  budget_estimate:
    type: string
    enum: ["<1M", "1M-3M", "3M-10M", ">10M"]
    description: "Optional budget range"
  deadline_hint:
    type: string
    format: date
    description: "Optional target launch date"
```

### Output
```yaml
type: object
required: [questions, next_suggested_action]
properties:
  questions:
    type: object
    properties:
      pm:
        type: array
        items: {type: string}
        minItems: 2
        maxItems: 3
        description: "Product Manager perspective questions"
      analyst:
        type: array
        items: {type: string}
        minItems: 2
        maxItems: 3
        description: "Business Analyst perspective questions"
      consultant:
        type: array
        items: {type: string}
        minItems: 2
        maxItems: 3
        description: "Technical Consultant perspective questions"
    additionalProperties: false
  next_suggested_action:
    type: string
    description: "What user should do next"
  confidence:
    type: number
    minimum: 0
    maximum: 1
    description: "Confidence score for understanding project"
```

### Pre-Conditions
- Input description > 0 characters
- Budget estimate (if provided) is valid enum value
- Deadline (if provided) is valid ISO date

### Post-Conditions
- Questions returned across all 3 roles (PM, Analyst, Consultant)
- Each role has 2-3 unique, non-overlapping questions
- Next action is actionable instruction
- Confidence score reflects question coverage

### Timeout
**15 seconds**

### Error Handling
```yaml
errors:
  EMPTY_INPUT:
    trigger: "description length < 1"
    response: "Please describe your project. What are you building?"
  INVALID_BUDGET:
    trigger: "budget not in enum"
    response: "Invalid budget format. Use: <1M, 1M-3M, 3M-10M, >10M"
  INVALID_DATE:
    trigger: "deadline not ISO date"
    response: "Please provide deadline in YYYY-MM-DD format"
  TIMEOUT:
    trigger: "execution > 15s"
    response: "Service temporarily slow. Try again in a moment."
```

---

## Contract 2: `analyze_requirements`

**Purpose:** Extract functional/non-functional requirements from clarifying question answers

### Input
```yaml
type: object
required: [answers]
properties:
  answers:
    type: object
    additionalProperties: {type: string}
    description: "User's answers to clarifying questions"
  context:
    type: object
    properties:
      github_repo:
        type: string
        description: "Optional existing project URL"
      team_experience:
        type: string
        enum: ["startup", "scale-up", "enterprise"]
        description: "Optional team context"
    description: "Optional context information"
```

### Output
```yaml
type: object
required: [functional_requirements, non_functional_requirements, success_criteria]
properties:
  project_name:
    type: string
  goal:
    type: string
  target_audience:
    type: string
  functional_requirements:
    type: array
    items: {type: string}
    minItems: 5
    description: "What the system does (natural language)"
  non_functional_requirements:
    type: object
    properties:
      performance:
        type: string
        description: "Speed, latency, throughput expectations"
      scalability:
        type: string
        description: "Growth, concurrent users, data volume"
      security:
        type: string
        description: "Authentication, authorization, compliance"
      availability:
        type: string
        description: "Uptime, disaster recovery"
    required: [performance, security]
  constraints:
    type: object
    properties:
      budget:
        type: string
      timeline:
        type: string
      team_constraints:
        type: string
  success_criteria:
    type: array
    items: {type: string}
    minItems: 3
    description: "Measurable outcomes"
```

### Pre-Conditions
- Answers object is non-empty
- All answer values are strings

### Post-Conditions
- Functional requirements: natural language only (no code, no DB schema)
- Non-functional requirements: generic (e.g., "responsive design" not "CSS media queries")
- Success criteria: measurable (e.g., "10k users in month 1")
- No technical specifications sufficient for self-implementation

### Timeout
**30 seconds**

### Error Handling
```yaml
errors:
  EMPTY_ANSWERS:
    trigger: "answers object empty"
    response: "Please answer at least one clarifying question."
  INSUFFICIENT_DATA:
    trigger: "confidence < 0.5"
    response: "Need more information. Please elaborate on [topic]."
  TIMEOUT:
    trigger: "execution > 30s"
    response: "Analysis taking longer than expected. Please try again."
```

---

## Contract 3: `suggest_roadmap`

**Purpose:** Generate project roadmap with phase names, task types, and example durations

### Input
```yaml
type: object
required: [requirements]
properties:
  requirements:
    type: object
    description: "Output from analyze_requirements"
  budget_estimate:
    type: string
    enum: ["<1M", "1M-3M", "3M-10M", ">10M"]
  team_size:
    type: integer
    minimum: 1
    maximum: 100
    description: "Expected team size"
  preferred_approach:
    type: string
    enum: ["mvp", "phased", "full-scope"]
    default: "phased"
```

### Output
```yaml
type: object
required: [phases, disclaimer]
properties:
  phases:
    type: array
    items:
      type: object
      required: [name, task_types, duration_weeks_example]
      properties:
        name:
          type: string
          description: "Phase name (e.g., 'Design Phase')"
        task_types:
          type: array
          items: {type: string}
          description: "Generic task types, not specific tasks"
        duration_weeks_example:
          type: integer
          description: "Example duration; not binding"
        team_roles:
          type: array
          items: {type: string}
          description: "Suggested roles for this phase"
    minItems: 3
    maxItems: 6
  estimated_total_weeks:
    type: integer
  team_composition_example:
    type: string
    description: "E.g., '1 architect + 2 backend devs + 2 frontend devs'"
  disclaimer:
    type: string
    description: "Always: 'Contact us for exact planning and cost'"
```

### Pre-Conditions
- Requirements object contains functional and non-functional requirements
- Budget and team size (if provided) are valid

### Post-Conditions
- Phases use generic names ("Design Phase" not "Design UI in Figma")
- Task types are categories ("API development" not "Write FastAPI endpoints")
- Disclaimer present in output
- Total duration is estimate only
- No implementation roadmap (no PR steps, no sprint breakdowns)

### Timeout
**20 seconds**

### Error Handling
```yaml
errors:
  MISSING_REQUIREMENTS:
    trigger: "requirements object incomplete"
    response: "Please complete requirements analysis first."
  INVALID_BUDGET:
    trigger: "budget not in enum or missing"
    response: "Roadmap generated without budget constraints."
  TIMEOUT:
    trigger: "execution > 20s"
    response: "Roadmap generation in progress. Please wait."
```

---

## Contract 4: `identify_risks`

**Purpose:** Identify project risks with probability, impact, and mitigation strategies

### Input
```yaml
type: object
required: [requirements]
properties:
  requirements:
    type: object
    description: "Output from analyze_requirements"
  team_context:
    type: string
    description: "Team experience, constraints, etc."
  historical_data:
    type: object
    description: "Optional past project data"
```

### Output
```yaml
type: object
required: [risks]
properties:
  risks:
    type: array
    items:
      type: object
      required: [description, probability, impact, mitigation]
      properties:
        description:
          type: string
          description: "Risk description"
        category:
          type: string
          enum: ["technical", "schedule", "resource", "scope", "external"]
        probability:
          type: string
          enum: ["low", "medium", "high"]
        impact:
          type: string
          enum: ["low", "medium", "high"]
        mitigation:
          type: string
          description: "Generic mitigation strategy (not specific tasks)"
    minItems: 3
    maxItems: 10
  risk_summary:
    type: string
    description: "Overall risk assessment"
```

### Pre-Conditions
- Requirements object is complete

### Post-Conditions
- Risks identified across multiple categories
- Probability and impact are balanced (not all "high")
- Mitigations are generic ("Add security review" not "Hire OWASP expert")
- Risk summary is actionable but non-alarming

### Timeout
**20 seconds**

---

## Contract 5: `generate_artifacts`

**Purpose:** Generate user stories, use cases, or acceptance criteria in Markdown format

### Input
```yaml
type: object
required: [artifact_type, requirements]
properties:
  artifact_type:
    type: string
    enum: ["user_stories", "use_cases", "acceptance_criteria"]
  requirements:
    type: object
    description: "Functional requirements"
  format:
    type: string
    enum: ["connextra", "bdd", "bullet_list"]
    default: "connextra"
```

### Output
```yaml
type: string
description: "Markdown formatted artifact"
examples:
  user_stories: |
    # User Stories
    
    ## US-1: User Authentication
    As a user, I want to log in with email and password,
    So that I can access my personalized dashboard.
    
    Acceptance Criteria:
    - User enters email and password
    - System validates credentials
    - User redirected to dashboard on success
    - Error message on failure
    
  use_cases: |
    # Use Case: Search Products
    
    **Actors:** User, Search Engine
    **Preconditions:** User is logged in
    **Main Flow:**
    1. User enters search term
    2. System queries database
    3. Results displayed
    
    **Exceptions:** No results found
```

### Pre-Conditions
- Artifact type is valid enum
- Requirements object contains functional requirements

### Post-Conditions
- Output is valid Markdown
- No Gherkin scenarios that map directly to step definitions
- No code examples or pseudo-code
- Artifacts are planning-level, not implementation-level

### Timeout
**25 seconds**

---

## Contract 6: `prepare_meeting`

**Purpose:** Generate meeting agenda and participant questions

### Input
```yaml
type: object
required: [purpose]
properties:
  purpose:
    type: string
    enum: ["discovery", "planning", "review", "kickoff", "retrospective"]
  context:
    type: string
    description: "Meeting context"
  participants:
    type: array
    items: {type: string}
    description: "Participant roles"
```

### Output
```yaml
type: object
required: [agenda, questions, expected_outcomes]
properties:
  meeting_title:
    type: string
  duration_minutes:
    type: integer
  agenda:
    type: array
    items:
      type: object
      properties:
        topic: {type: string}
        duration_minutes: {type: integer}
        owner: {type: string}
  questions_for_participants:
    type: array
    items: {type: string}
    description: "Questions to pose, not answers"
  expected_outcomes:
    type: array
    items: {type: string}
    description: "Meeting goals"
```

### Pre-Conditions
- Purpose is valid enum

### Post-Conditions
- Agenda is time-boxed
- Questions are open-ended (not yes/no)
- Outcomes are clear and achievable in meeting duration

### Timeout
**15 seconds**

---

## Contract 7: `evaluate_idea`

**Purpose:** SWOT analysis and complexity estimate for ideas

### Input
```yaml
type: object
required: [idea_description]
properties:
  idea_description:
    type: string
  industry:
    type: string
    description: "Optional industry context"
  market_data:
    type: object
    description: "Optional competitive/market info"
```

### Output
```yaml
type: object
required: [swot, complexity_estimate]
properties:
  swot:
    type: object
    properties:
      strengths:
        type: array
        items: {type: string}
      weaknesses:
        type: array
        items: {type: string}
      opportunities:
        type: array
        items: {type: string}
      threats:
        type: array
        items: {type: string}
  complexity_estimate:
    type: string
    enum: ["low", "medium", "high"]
  reasoning:
    type: string
  similar_examples:
    type: array
    items: {type: string}
    description: "Optional market references"
```

### Pre-Conditions
- Idea description is non-empty

### Post-Conditions
- SWOT analysis balanced (not all positive/negative)
- Complexity estimate justified
- References are factual or acknowledged as speculative

### Timeout
**20 seconds**

---

## Contract 8: `consult_on_process`

**Purpose:** Answer process and methodology questions

### Input
```yaml
type: object
required: [question]
properties:
  question:
    type: string
  role_hint:
    type: string
    enum: ["pm", "analyst", "consultant", "developer"]
    description: "Optional asker's role for context"
```

### Output
```yaml
type: object
required: [answer, next_action]
properties:
  answer:
    type: string
    description: "Detailed answer with reasoning"
  sources:
    type: array
    items: {type: string}
    description: "Optional references"
  next_action:
    type: string
    description: "Follow-up question or suggestion"
```

### Pre-Conditions
- Question is non-empty
- Question is about process/methodology (not code/implementation)

### Post-Conditions
- Answer includes reasoning
- If question is code/tech implementation → redirect to company offer
- Next action is actionable

### Timeout
**20 seconds**

### Error Handling
```yaml
errors:
  TECHNICAL_QUESTION:
    trigger: "question contains code/implementation keywords"
    response: "This is implementation work. Our team can help. Contact us."
  OUT_OF_SCOPE:
    trigger: "question unrelated to project/product development"
    response: "This is outside my scope. Could you rephrase in project context?"
```

---

## Contract 9: `handle_github_url`

**Purpose:** Extract README and file structure from public GitHub repos

### Input
```yaml
type: object
required: [url]
properties:
  url:
    type: string
    format: uri
    description: "GitHub repository URL (any format)"
```

### Output
```yaml
type: object
required: [owner, repo, readme_snippet, file_structure]
properties:
  owner:
    type: string
  repo:
    type: string
  readme_snippet:
    type: string
    description: "First 500 chars of README (decoded from base64)"
  file_structure:
    type: array
    items:
      type: object
      properties:
        name: {type: string}
        type: {type: string, enum: ["file", "dir"]}
    description: "Top-level files and directories"
  is_public:
    type: boolean
  next_action:
    type: string
```

### Pre-Conditions
- URL is valid GitHub URL
- Repository exists
- Repository is public (no auth required)

### Post-Conditions
- README snippet is sanitized (no HTML, no scripts)
- File structure limited to top-level only
- No code suggestion or analysis

### Timeout
**10 seconds**

### Error Handling
```yaml
errors:
  INVALID_URL:
    trigger: "URL is not valid GitHub format"
    response: "Please provide a valid GitHub URL."
  REPO_NOT_FOUND:
    trigger: "404 from GitHub API"
    response: "Repository not found. Is it public and does URL exist?"
  PRIVATE_REPO:
    trigger: "GitHub returns 403 (private)"
    response: "Repository is private. I can only analyze public repos."
  TIMEOUT:
    trigger: "execution > 10s"
    response: "GitHub API slow. Try again in a moment."
  RATE_LIMIT:
    trigger: "429 from GitHub API"
    response: "Rate limit reached. Try again after a few minutes."
```

---

## Contract Registry

**Summary Table:**

| Contract | Input | Output | Timeout | Use Case |
|----------|-------|--------|---------|----------|
| clarify_project | Description | Questions | 15s | Vague project intro |
| analyze_requirements | Answers | Functional/Non-functional reqs | 30s | Build from clarification |
| suggest_roadmap | Requirements | Phases + task types | 20s | High-level planning |
| identify_risks | Requirements | Risk matrix | 20s | Risk mitigation |
| generate_artifacts | Requirement type | User stories / use cases | 25s | Artifact generation |
| prepare_meeting | Purpose | Agenda + questions | 15s | Meeting prep |
| evaluate_idea | Idea | SWOT + complexity | 20s | Idea validation |
| consult_on_process | Question | Answer + next action | 20s | Methodology advice |
| handle_github_url | URL | README + structure | 10s | Repo analysis |

---

## Contract Validation Flow

Every contract execution:

```
1. Input Validation
   ↓
2. Pre-Condition Check
   ↓
3. Execute (LLM call or logic)
   ↓
4. Output Schema Validation
   ↓
5. Post-Condition Check
   ↓
6. BusinessGuard Filter
   ↓
7. Return to User
```

If any step fails → error handler returns appropriate error message from contract spec.

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
