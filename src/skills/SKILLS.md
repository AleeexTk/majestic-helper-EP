# Skills — Composable Capabilities (Layer 5)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Overview

Skills are reusable, composable capabilities. Each skill:
- Implements one contract
- Validates input against schema
- Executes logic (LLM or deterministic)
- Validates output against schema
- Applies BusinessGuard filter
- Returns result or error

---

## Base Skill Interface

**All skills inherit from `BaseSkill`:**

```python
class BaseSkill:
    def __init__(self, contract, schema_validator, llm_client, business_guard):
        self.contract = contract
        self.validator = schema_validator
        self.llm = llm_client
        self.guard = business_guard
    
    async def execute(self, params: dict, session_context: dict) -> dict:
        """
        Execute skill:
        1. Validate input against schema
        2. Execute contract logic
        3. Validate output against schema
        4. Apply BusinessGuard filter
        5. Return result or error
        """
        # Input validation
        input_schema = self.contract['input_schema']
        input_validation = self.validator.validate(params, input_schema)
        if not input_validation['valid']:
            return self.contract['error_handler']['SCHEMA_VALIDATION'](input_validation['errors'])
        
        # Execute with timeout
        try:
            result = await asyncio.wait_for(
                self._execute_logic(params, session_context),
                timeout=self.contract['timeout']
            )
        except asyncio.TimeoutError:
            return self.contract['error_handler']['TIMEOUT']()
        except Exception as e:
            return self.contract['error_handler']['EXECUTION_ERROR'](str(e))
        
        # Output validation
        output_schema = self.contract['output_schema']
        output_validation = self.validator.validate(result, output_schema)
        if not output_validation['valid']:
            return self.contract['error_handler']['SCHEMA_VALIDATION'](output_validation['errors'])
        
        # Apply BusinessGuard
        result_text = json.dumps(result)
        filtered_result = self.guard.filter_response(result_text, session_context)
        
        return json.loads(filtered_result)
    
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        """Implement in subclasses"""
        raise NotImplementedError
```

---

## Skill 1: `ClarifyProjectSkill`

**Contract:** `clarify_project`  
**Input:** Project description (vague)  
**Output:** 3-5 clarifying questions across PM/Analyst/Consultant roles  

### Implementation Logic

```python
class ClarifyProjectSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        prompt = f"""
        Given this project description:
        \"{params['description']}\"
        
        Generate 3 clarifying questions from each perspective:
        1. Product Manager (business focus)
        2. Business Analyst (requirements focus)
        3. Technical Consultant (technical feasibility)
        
        Questions should be:
        - Specific to the project
        - Non-overlapping across roles
        - Open-ended (not yes/no)
        - Actionable (answerable by user)
        
        Format as JSON object with 'pm', 'analyst', 'consultant' arrays.
        """
        
        response = await self.llm.generate(prompt, max_tokens=300)
        parsed = json.loads(response)
        
        confidence = self._calculate_confidence(
            params['description'],
            len(parsed.get('pm', [])) + len(parsed.get('analyst', [])) + len(parsed.get('consultant', []))
        )
        
        return {
            'questions': parsed,
            'next_suggested_action': "Answer any or all questions to help us understand your project better.",
            'confidence': confidence
        }
    
    def _calculate_confidence(self, description: str, num_questions: int) -> float:
        """Confidence is based on input clarity and questions generated"""
        base_confidence = min(len(description) / 500, 0.7)  # max 0.7 from input length
        question_bonus = min(num_questions / 9, 0.3)  # bonus up to 0.3 from questions
        return base_confidence + question_bonus
```

---

## Skill 2: `AnalyzeRequirementsSkill`

**Contract:** `analyze_requirements`  
**Input:** Answers to clarifying questions  
**Output:** Functional/non-functional requirements, constraints, success criteria  

### Implementation Logic

```python
class AnalyzeRequirementsSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        # Build context from session
        description = session_context.get('context', {}).get('description', '')
        answers_text = '\n'.join([f"Q: {k}\nA: {v}" for k, v in params['answers'].items()])
        
        prompt = f"""
        Original project: {description}
        
        User's answers to questions:
        {answers_text}
        
        Extract and structure:
        1. Functional requirements (what system does) - natural language list
        2. Non-functional requirements (how system behaves) - performance, scalability, security, availability
        3. Constraints (budget, timeline, team)
        4. Success criteria (measurable outcomes)
        
        DO NOT generate:
        - API specifications
        - Database schemas
        - Code examples
        - Technical specifications sufficient for implementation
        
        Format as JSON object.
        """
        
        response = await self.llm.generate(prompt, max_tokens=500)
        parsed = json.loads(response)
        
        # Ensure no technical specs leaked
        parsed = self._sanitize_requirements(parsed)
        
        return parsed
    
    def _sanitize_requirements(self, reqs: dict) -> dict:
        """Remove any technical specs or implementation details"""
        # Check for keywords indicating leaked specs
        forbidden_keywords = ['endpoint', 'schema', 'table', 'field', 'def ', 'class ', 'function']
        
        for section in ['functional_requirements', 'non_functional_requirements']:
            if section in reqs:
                for req in reqs[section] if isinstance(reqs[section], list) else [reqs[section]]:
                    for keyword in forbidden_keywords:
                        if keyword.lower() in req.lower():
                            raise ValueError(f"Sanitization failed: found '{keyword}' in output")
        
        return reqs
```

---

## Skill 3: `SuggestRoadmapSkill`

**Contract:** `suggest_roadmap`  
**Input:** Requirements + budget + team size  
**Output:** Phases, task types, example durations  

### Implementation Logic

```python
class SuggestRoadmapSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        reqs = params['requirements']
        budget = params.get('budget_estimate', 'unknown')
        team_size = params.get('team_size', 3)
        
        prompt = f"""
        Project requirements:
        Functional: {reqs.get('functional_requirements', [])}
        Non-functional: {reqs.get('non_functional_requirements', {})}
        
        Budget: {budget}
        Team size: {team_size}
        
        Generate a high-level roadmap with 3-5 phases.
        Each phase should have:
        - Name (generic: 'Design Phase', 'Development Phase')
        - Task types (categories, not specific tasks)
        - Estimated duration in weeks (example only)
        
        DO NOT provide:
        - Specific tasks (e.g., 'Write login controller')
        - Sprint breakdowns
        - Implementation steps
        - PR workflow
        
        Include disclaimer: 'These timelines are estimates. Contact us for exact planning.'
        
        Format as JSON.
        """
        
        response = await self.llm.generate(prompt, max_tokens=400)
        parsed = json.loads(response)
        
        # Ensure disclaimer present
        if 'disclaimer' not in parsed or 'contact' not in parsed['disclaimer'].lower():
            parsed['disclaimer'] = "These timelines are estimates only. Contact us for exact planning and cost."
        
        return parsed
```

---

## Skill 4: `IdentifyRisksSkill`

**Contract:** `identify_risks`  
**Input:** Requirements + team context  
**Output:** Risk list with probability, impact, mitigation  

### Implementation Logic

```python
class IdentifyRisksSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        reqs = params['requirements']
        team_context = params.get('team_context', 'not provided')
        
        prompt = f"""
        Project requirements: {reqs}
        Team context: {team_context}
        
        Identify 5-8 realistic risks:
        - Technical risks (complexity, new tech, integration)
        - Schedule risks (scope creep, estimation errors)
        - Resource risks (skill gaps, availability)
        - Scope risks (unclear requirements, changing specs)
        - External risks (vendor dependencies, market changes)
        
        For each risk provide:
        - Description (what could go wrong)
        - Probability (low/medium/high)
        - Impact (low/medium/high)
        - Mitigation (generic strategy)
        
        Format as JSON with 'risks' array and 'risk_summary' string.
        """
        
        response = await self.llm.generate(prompt, max_tokens=400)
        parsed = json.loads(response)
        
        return parsed
```

---

## Skill 5: `GenerateArtifactsSkill`

**Contract:** `generate_artifacts`  
**Input:** Artifact type (user_stories/use_cases/acceptance_criteria) + requirements  
**Output:** Markdown formatted artifact  

### Implementation Logic

```python
class GenerateArtifactsSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        artifact_type = params['artifact_type']
        reqs = params['requirements']
        
        if artifact_type == 'user_stories':
            return await self._generate_user_stories(reqs)
        elif artifact_type == 'use_cases':
            return await self._generate_use_cases(reqs)
        elif artifact_type == 'acceptance_criteria':
            return await self._generate_acceptance_criteria(reqs)
    
    async def _generate_user_stories(self, reqs: dict) -> dict:
        prompt = f"""
        Requirements: {reqs['functional_requirements']}
        
        Generate user stories in Connextra format:
        As a [actor], I want to [action], So that [benefit]
        
        For each story include acceptance criteria (3-5 points).
        
        Return as Markdown string (wrapped in 'artifact' field).
        """
        
        response = await self.llm.generate(prompt, max_tokens=500)
        
        return {
            'artifact': response,
            'type': 'user_stories'
        }
    
    async def _generate_use_cases(self, reqs: dict) -> dict:
        # Similar implementation
        pass
```

---

## Skill 6: `PrepareMeetingSkill`

**Contract:** `prepare_meeting`  
**Input:** Purpose + context + participants  
**Output:** Agenda + questions + expected outcomes  

```python
class PrepareMeetingSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        purpose = params['purpose']
        context = params.get('context', '')
        participants = params.get('participants', [])
        
        prompt = f"""
        Meeting purpose: {purpose}
        Context: {context}
        Participants: {', '.join(participants)}
        
        Create a meeting structure:
        1. Meeting title
        2. Recommended duration (minutes)
        3. Agenda with time allocations
        4. Questions to pose (not answers)
        5. Expected outcomes
        
        Format as JSON.
        """
        
        response = await self.llm.generate(prompt, max_tokens=300)
        return json.loads(response)
```

---

## Skill 7: `EvaluateIdeaSkill`

**Contract:** `evaluate_idea`  
**Input:** Idea description + industry  
**Output:** SWOT analysis + complexity estimate  

```python
class EvaluateIdeaSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        idea = params['idea_description']
        industry = params.get('industry', 'general')
        
        prompt = f"""
        Idea: {idea}
        Industry: {industry}
        
        Provide:
        1. SWOT Analysis (Strengths, Weaknesses, Opportunities, Threats)
        2. Complexity estimate (low/medium/high) with reasoning
        3. Similar market examples (if known)
        
        Format as JSON.
        """
        
        response = await self.llm.generate(prompt, max_tokens=350)
        return json.loads(response)
```

---

## Skill 8: `ConsultOnProcessSkill`

**Contract:** `consult_on_process`  
**Input:** Question + role hint  
**Output:** Answer + next action  

```python
class ConsultOnProcessSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        question = params['question']
        role_hint = params.get('role_hint', 'general')
        
        # Check if question is about code/implementation
        if self._is_technical_question(question):
            return {
                'answer': "This is implementation work. Our team can help you. Would you like to discuss development options?",
                'next_action': "Schedule a technical discovery call",
                'redirected_to_company': True
            }
        
        prompt = f"""
        Question from {role_hint}: {question}
        
        Provide:
        1. Detailed answer with reasoning
        2. Relevant best practices
        3. Next step suggestion
        
        Format as JSON with 'answer', 'sources', 'next_action' fields.
        """
        
        response = await self.llm.generate(prompt, max_tokens=350)
        return json.loads(response)
    
    def _is_technical_question(self, question: str) -> bool:
        tech_keywords = ['code', 'implement', 'function', 'class', 'def', 'api', 'database', 'sql']
        return any(kw in question.lower() for kw in tech_keywords)
```

---

## Skill 9: `HandleGithubUrlSkill`

**Contract:** `handle_github_url`  
**Input:** GitHub URL  
**Output:** README snippet + file structure  

```python
class HandleGithubUrlSkill(BaseSkill):
    async def _execute_logic(self, params: dict, session_context: dict) -> dict:
        url = params['url']
        
        # Parse URL
        match = re.match(r'https://github\.com/([^/]+)/([^/]+)', url)
        if not match:
            raise ValueError("Invalid GitHub URL")
        
        owner, repo = match.groups()
        
        try:
            # Fetch README
            readme_response = await self._fetch_github_file(owner, repo, 'README.md')
            readme = base64.b64decode(readme_response['content']).decode('utf-8')
            readme_snippet = readme[:500]
            
            # Fetch file structure
            tree_response = await self._fetch_github_tree(owner, repo)
            file_structure = self._parse_tree(tree_response['tree'])
            
            return {
                'owner': owner,
                'repo': repo,
                'readme_snippet': readme_snippet,
                'file_structure': file_structure,
                'is_public': True,
                'next_action': "What specific question about this project?"
            }
        except Exception as e:
            if '404' in str(e):
                raise ValueError("Repository not found")
            elif '403' in str(e):
                raise ValueError("Repository is private")
            else:
                raise
    
    async def _fetch_github_file(self, owner: str, repo: str, path: str) -> dict:
        """Fetch file from GitHub API"""
        url = f"https://api.github.com/repos/{owner}/{repo}/contents/{path}"
        # Implementation with GitHub API call
        pass
```

---

## BusinessGuard Integration

Every skill's output passes through `BusinessGuard.filter_response()` which:

1. Scans for code blocks (>10 lines) → replaces with rejection
2. Detects architectural details → replaces with company offer
3. Counts technical questions → forces redirect after 3
4. Appends company disclaimer if technical content

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
