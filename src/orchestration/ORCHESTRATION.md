# Orchestration — Session Management & Routing (Layer 6)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Overview

Orchestrator is the entry point. It manages:
- Session lifecycle
- Intent classification
- Routing to appropriate contracts/skills
- Context preservation
- Output filtering via BusinessGuard

---

## Session Flow

```
User Message
    ↓
Orchestrator.route(session_id, message)
    ↓
1. Load/Create Session
    ↓
2. Classify Intent
    ↓
3. Route to Contract
    ↓
4. Execute Skill
    ↓
5. Apply BusinessGuard Filter
    ↓
6. Update Session State
    ↓
7. Return Response
```

---

## Intent Classification

**Fast rule-based + LLM fallback:**

```python
class IntentClassifier:
    def __init__(self, llm_client=None):
        self.rules = {
            'github_intent': r'github\.com/[\w-]+/[\w-]+',
            'clarify_intent': r'(what|describe|tell me about)',
            'roadmap_intent': r'(roadmap|plan|phases|timeline)',
            'risk_intent': r'(risk|danger|challenge|problem)',
            'artifact_intent': r'(user story|use case|criteria)',
            'meeting_intent': r'(meeting|agenda|discuss)',
            'idea_intent': r'(idea|evaluate|swot|startup)',
            'process_intent': r'(how to|process|methodology|agile|scrum)',
        }
        self.llm = llm_client
    
    def classify(self, message: str, last_contract: str = None) -> dict:
        """
        Returns:
        {
            'intent': contract_name,
            'confidence': 0.0-1.0,
            'requires_clarification': bool
        }
        """
        
        # Rule-based detection
        for intent, pattern in self.rules.items():
            if re.search(pattern, message, re.IGNORECASE):
                return {
                    'intent': intent,
                    'confidence': 0.9,
                    'requires_clarification': False
                }
        
        # If no clear intent
        if len(message) < 50:
            return {
                'intent': 'clarify_project',
                'confidence': 0.7,
                'requires_clarification': True
            }
        
        # LLM fallback (if enabled)
        if self.llm:
            return self._classify_with_llm(message, last_contract)
        
        # Default to clarify
        return {
            'intent': 'clarify_project',
            'confidence': 0.5,
            'requires_clarification': True
        }
    
    async def _classify_with_llm(self, message: str, last_contract: str) -> dict:
        prompt = f"""
        User message: \"{message}\"
        
        Classify into one of these intents:
        - clarify_project: asking about new project
        - analyze_requirements: providing answers to questions
        - suggest_roadmap: asking for project plan
        - identify_risks: asking about risks/challenges
        - generate_artifacts: requesting user stories/use cases
        - prepare_meeting: meeting preparation
        - evaluate_idea: idea evaluation
        - consult_on_process: process/methodology question
        - github_intent: GitHub URL provided
        
        Return JSON: {{"intent": "...", "confidence": 0.0-1.0}}
        """
        
        response = await self.llm.generate(prompt, max_tokens=50)
        parsed = json.loads(response)
        return parsed
```

---

## Orchestrator Class

```python
class Orchestrator:
    def __init__(self, runtime, skills, business_guard, session_store):
        self.runtime = runtime
        self.skills = skills
        self.guard = business_guard
        self.sessions = session_store
        self.classifier = IntentClassifier()
    
    async def route(self, session_id: str, message: str) -> str:
        """
        Main entry point for routing user messages.
        Returns response string.
        """
        
        # Load or create session
        session = await self.sessions.get_or_create(session_id)
        
        # Validate message
        if not message or len(message.strip()) == 0:
            return "Please provide a message or question."
        
        # Check GitHub URL
        if self._is_github_url(message):
            result = await self._execute_skill('handle_github_url', {'url': message}, session)
            return self._format_response(result)
        
        # Classify intent
        classification = self.classifier.classify(message, session.get('last_contract'))
        
        # If confidence too low, clarify
        if classification['confidence'] < 0.6:
            intent = 'clarify_project'
        else:
            intent = classification['intent']
        
        # Prepare params based on intent
        params = self._prepare_params(intent, message, session)
        
        # Execute skill
        result = await self._execute_skill(intent, params, session)
        
        # Apply BusinessGuard
        result = self.guard.filter_response(result, session)
        
        # Update session
        await self.sessions.update(session_id, {
            'last_contract': intent,
            'last_active': datetime.now(),
            'history': session.get('history', []) + [{
                'user_message': message,
                'intent': intent,
                'timestamp': datetime.now()
            }]
        })
        
        return self._format_response(result)
    
    def _is_github_url(self, message: str) -> bool:
        return bool(re.search(r'github\.com/[\w-]+/[\w-]+', message))
    
    def _prepare_params(self, intent: str, message: str, session: dict) -> dict:
        """
        Prepare contract parameters based on intent.
        """
        if intent == 'clarify_project':
            return {'description': message}
        
        elif intent == 'analyze_requirements':
            # Parse answers from message
            return {'answers': self._parse_answers(message)}
        
        elif intent == 'suggest_roadmap':
            # Use stored requirements from session
            return {
                'requirements': session.get('requirements', {}),
                'budget_estimate': session.get('budget'),
                'team_size': session.get('team_size')
            }
        
        elif intent == 'consult_on_process':
            return {'question': message}
        
        elif intent == 'handle_github_url':
            return {'url': message}
        
        else:
            # Generic intent - store requirements and route
            return {'requirements': session.get('requirements', {})}
    
    async def _execute_skill(self, intent: str, params: dict, session: dict) -> dict:
        """
        Execute appropriate skill based on intent.
        """
        skill_mapping = {
            'clarify_project': self.skills['clarify_project'],
            'analyze_requirements': self.skills['analyze_requirements'],
            'suggest_roadmap': self.skills['suggest_roadmap'],
            'identify_risks': self.skills['identify_risks'],
            'generate_artifacts': self.skills['generate_artifacts'],
            'prepare_meeting': self.skills['prepare_meeting'],
            'evaluate_idea': self.skills['evaluate_idea'],
            'consult_on_process': self.skills['consult_on_process'],
            'handle_github_url': self.skills['handle_github_url']
        }
        
        skill = skill_mapping.get(intent)
        if not skill:
            return {'error': 'Unknown intent'}
        
        try:
            result = await self.runtime.execute_with_timeout(
                skill.execute(params, session),
                timeout=skill.contract['timeout']
            )
            
            # Store results in session for future reference
            if intent == 'clarify_project':
                pass  # Questions don't need storage
            elif intent == 'analyze_requirements':
                await self.sessions.update(session['session_id'], {
                    'requirements': result
                })
            
            return result
        
        except Exception as e:
            return {'error': str(e)}
    
    def _format_response(self, result: dict) -> str:
        """
        Convert dict result to formatted string for user.
        """
        if 'error' in result:
            return f"Error: {result['error']}"
        
        # Convert dict to readable format (Markdown)
        if isinstance(result, dict):
            return self._dict_to_markdown(result)
        
        return str(result)
    
    def _dict_to_markdown(self, data: dict) -> str:
        """
        Convert dictionary to readable Markdown format.
        """
        lines = []
        
        for key, value in data.items():
            if isinstance(value, list):
                lines.append(f"## {key.replace('_', ' ').title()}")
                for item in value:
                    if isinstance(item, dict):
                        lines.append(f"- {item}")
                    else:
                        lines.append(f"- {item}")
            elif isinstance(value, dict):
                lines.append(f"## {key.replace('_', ' ').title()}")
                for k, v in value.items():
                    lines.append(f"- **{k}:** {v}")
            else:
                lines.append(f"**{key.replace('_', ' ').title()}:** {value}")
        
        return "\n".join(lines)
```

---

## No Greetings Policy

```python
class NoGreetingPolicy:
    """
    Orchestrator does NOT send welcome messages.
    First user message is processed immediately.
    """
    
    async def route(self, session_id: str, message: str) -> str:
        # If empty or just "hi"
        if message.lower().strip() in ['', 'hi', 'hello', 'hey']:
            # Classify as clarify_project
            return "Please describe your project. What are you building?"
        
        # Otherwise process normally
        return await self._normal_route(session_id, message)
```

---

## Error Handling in Routing

```python
class OrchestratorErrorHandler:
    @staticmethod
    async def route_with_error_handling(orchestrator, session_id: str, message: str) -> str:
        try:
            return await orchestrator.route(session_id, message)
        
        except TimeoutError:
            return "Request took too long. Please try again."
        
        except SchemaValidationError as e:
            return f"Input format error: {e.message}"
        
        except RateLimitError:
            return "Service is busy. Please wait a moment and try again."
        
        except Exception as e:
            # Log error
            logger.error(f"Orchestration error: {e}", exc_info=e)
            return "An error occurred. Please try again or contact support."
```

---

## Orchestrator State Machine

```
INITIAL
  ↓
  User provides project description
  ↓
CLARIFYING
  ↓
  User answers clarifying questions
  ↓
ANALYZING
  ↓
  User asks for roadmap/risks/artifacts
  ↓
PLANNING
  ↓
  User wants to proceed to implementation
  ↓
REDIRECT_TO_COMPANY
```

Transitions:
- INITIAL → CLARIFYING: User asks vague question
- CLARIFYING → ANALYZING: User provides answers
- ANALYZING → PLANNING: User requests roadmap
- PLANNING → REDIRECT_TO_COMPANY: User asks for code/architecture

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
