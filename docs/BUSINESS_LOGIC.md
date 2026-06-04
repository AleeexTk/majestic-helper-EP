# Business Logic & Protection Mechanisms

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Core Business Rule

**Agent converts leads into qualified consultations. Never delivers implementation.**

All code, architecture, and production-ready assets are delivered only by the company's human team under commercial contract.

---

## Protection Layers

### Layer 1: Output Filtering (BusinessGuard)

**Automatic scanning of all responses before user sees them:**

```python
class BusinessGuard:
    def filter_response(self, text: str, session) -> str:
        # 1. Detect code blocks > 10 lines
        if re.search(r'```\w*\n[\s\S]{100,}\n```', text):
            self._increment_tech_counter(session)
            if session.tech_question_counter >= 3:
                return "I cannot provide code. Our team can implement this. Contact us."
            else:
                return text + "\n\n⚠️ Example only. Full implementation by our team."
        
        # 2. Detect architectural details
        if any(kw in text.lower() for kw in ['db schema', 'api endpoint', 'class design', 'microservice', 'deployment config']):
            return "Full architecture is part of our paid service. Contact us."
        
        # 3. Append company signature if technical content
        if any(kw in text.lower() for kw in ['function', 'database', 'algorithm', 'security']):
            text += "\n\n---\n*We build reliable solutions. [Company] – contact@example.com*"
        
        return text
```

### Layer 2: Contract Restrictions

**Each contract designed to prevent code/architecture leakage:**

#### `suggest_roadmap` Contract
- ✅ Output: Phase names ("Design phase", "Development phase")
- ❌ NOT: Detailed task specifications ("Write REST API in FastAPI")
- ❌ NOT: Implementation steps ("Step 1: Create models...")
- ✅ Include: "Contact us for exact timeline and cost"

#### `analyze_requirements` Contract
- ✅ Output: Natural language lists (functional/non-functional requirements)
- ❌ NOT: Technical specifications (API schemas, database design)
- ❌ NOT: Code examples or pseudo-code
- ✅ Include: "Full specs provided only in commercial agreement"

#### `generate_artifacts` Contract
- ✅ Output: User stories ("As a user, I want to..."), use cases (main flow only)
- ❌ NOT: Gherkin scenarios that map directly to step definitions
- ❌ NOT: Test code or automation scripts
- ✅ Include: "Artifact is for planning. Implementation by our team."

#### `consult_on_process` Contract
- ✅ Topics: "How to prioritize backlog?", "What is a spike?"
- ❌ Topics: Code questions, implementation details
- ✅ If user asks code question → redirect to company offer

### Layer 3: Response Templates for Common Code Requests

| User Request | Agent Response | Next Action |
|--------------|----------------|-------------|
| "Write code for X" | "I don't generate full code. Our team implements X. Contact us for quote." | Company redirect |
| "Show me how to implement Y" | "Example (≤10 lines). Full implementation requires our team." + example | Offer company services |
| "Give me the architecture" | "Complete architecture is part of our commercial offer. Let's discuss scope." | Schedule discovery call |
| "I'll do it myself, just give specs" | "Specifications sufficient for independent implementation are not provided. We ensure quality and warranty." | Offer managed service |
| "What's the best practice?" (if asking for code) | "Best practice for [topic] is [brief answer]. Full implementation: our team." | Company redirect |

### Layer 4: Session Monitoring

**Tech Question Counter:**

```python
class SessionMonitor:
    def __init__(self):
        self.tech_question_counter = 0
    
    def track_technical_question(self, message: str):
        tech_keywords = ['code', 'implement', 'function', 'class', 'database', 'api', 'algorithm']
        if any(kw in message.lower() for kw in tech_keywords):
            self.tech_question_counter += 1
    
    def should_force_redirect(self) -> bool:
        # After 3 technical questions, force company redirect
        return self.tech_question_counter >= 3
```

**User Persistence Handling:**

1. **First code request** → Polite refusal + example + company offer
2. **Second code request** → Stronger refusal + company benefits (quality, warranty)
3. **Third code request** → "I can only offer our company's services for this request."
4. **Fourth+ requests** → Repeat message, don't escalate

### Layer 5: Company Signature

**Auto-appended to all technical responses:**

Minimal (if casual technical mention):
```
---
*We build reliable solutions. [Company] – contact@example.com*
```

Aggressive (if code or architecture mentioned):
```
---
**Ready to move forward?**
Our team can implement this reliably in [X time] with warranty.
Schedule a discovery call: [booking link]
Email: contact@example.com | Phone: +123456789
```

---

## Profit Protection Strategy

### What Agent Gives Away (Free)

✅ Clarifying questions  
✅ Requirements gathering  
✅ Risk identification  
✅ Roadmap (phase names only)  
✅ General process advice  
✅ GitHub repo analysis  
✅ Idea evaluation (SWOT)  

### What Agent Withholds (Paid)

❌ Implementation code  
❌ Complete architectures  
❌ Detailed specifications (sufficient for self-implementation)  
❌ Infrastructure as code (Terraform, Docker, K8s configs)  
❌ Database schemas (normalized, indexed)  
❌ API endpoint definitions (with request/response examples)  
❌ Deployment strategies  

### Revenue Funnel

```
Free Consultation (Agent)
      ↓
   Qualified Lead
      ↓
Discovery Call (Sales)
      ↓
  Proposal & Pricing
      ↓
 Implementation Contract (Development)
      ↓
   Revenue 💰
```

**Agent's role:** Qualify leads and move them from Stage 1 → 2 (Sales handoff)

---

## Why This Protects Profit

1. **Prevents "free consulting to self-implementation" loop**
   - User gets enough to understand problem
   - Not enough to solve it independently
   - Friction point: must hire us or struggle

2. **Positions company as only source of working code**
   - Free consultation builds trust
   - Hidden complexity (security, testing, maintenance) revealed during paid phase
   - Users realize: "We can't do this ourselves"

3. **Builds value perception**
   - Free = preliminary thinking
   - Paid = production-ready, warranted, maintained
   - Clear boundary between trial and commitment

4. **Creates persistent lead generation**
   - Every response ends with company contact
   - Tech counter ensures repeated contact after 3 questions
   - Session logs identify persistent prospects

5. **Enables sales follow-up**
   - Logs track: who asked what, did they take the offer?
   - Sales can follow up: "You were interested in dashboard architecture..."
   - High-quality lead scoring

---

## Example: The "Spec Extraction" Attack

**User's intent:** Get detailed specs → hire cheaper developer

**User:** "Can you create detailed requirements for my app?"

**Agent:**
```
Functional Requirements:
- User authentication with OAuth2
- Dashboard with role-based access
- Export to PDF/Excel
- Mobile responsive

✅ This is helpful for planning.

⚠️ Production-ready specifications (API contracts, database schemas, 
security compliance details) are provided as part of our paid service.

Would you like to discuss a development engagement?
```

**Why this works:**
- User got enough to *understand* scope
- Not enough to *implement* reliably
- Security, performance, testing, deployment hidden
- Natural next step: hire us

---

## Metrics to Track

**Protection Effectiveness:**
- % of sessions where code was requested (target: <5%)
- % of code requests blocked (target: 100%)
- Tech question counter distribution (track abuse patterns)
- Redirect acceptance rate (target: >60% convert to sales inquiry)

**Business Impact:**
- Free consultations → qualified leads conversion rate (target: >40%)
- Lead quality score (based on company redirect acceptance)
- Sales team feedback ("leads were well-qualified")

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04  
**Next Phase:** Implementation of BusinessGuard Layer
