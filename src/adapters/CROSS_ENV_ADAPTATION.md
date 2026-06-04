# Cross-Environment Adaptation & Consistency (Layer 10)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Overview

Different LLM environments have:
- Different token limits
- Different input/output formats
- Different API capabilities
- Different latency profiles

**Challenge:** Ensure identical agent behavior across all environments.

**Solution:** Unified core + capability-aware runtime + adapters.

---

## Environment Capabilities

### ChatGPT (OpenAI)

```yaml
name: chatgpt
model: gpt-4-turbo
max_input_tokens: 8000
max_output_tokens: 4000
supports_system_message: true
supports_functions: true
supports_vision: false
rate_limit_rpm: 3500
latency_avg_ms: 2000
timeout_seconds: 60
cost_per_1k_input: $0.01
cost_per_1k_output: $0.03
```

### Claude (Anthropic)

```yaml
name: claude
model: claude-3-opus
max_input_tokens: 200000  # Much larger context window
max_output_tokens: 4000
supports_system_message: true
supports_functions: false
supports_vision: true
rate_limit_rpm: 1000
latency_avg_ms: 3000
timeout_seconds: 120
cost_per_1k_input: $0.015
cost_per_1k_output: $0.075
```

### Local (Ollama)

```yaml
name: local
model: llama2  # or other
max_input_tokens: 4096
max_output_tokens: 2048
supports_system_message: true
supports_functions: false
supports_vision: false
rate_limit_rpm: unlimited
latency_avg_ms: 5000-15000  # Varies by hardware
timeout_seconds: 180
cost_per_1k_input: $0
cost_per_1k_output: $0
```

---

## Capability Detection & Adaptation

### Runtime Adapts Based on Manifest

```python
class CapabilityAwareRuntime:
    def __init__(self, environment_manifest: dict):
        self.env = environment_manifest
        self.max_input = self.env['max_input_tokens']
        self.max_output = self.env['max_output_tokens']
        self.timeout = self.env['timeout_seconds']
    
    async def execute_skill(self, skill, params, session):
        """
        Adapts execution based on environment capabilities.
        """
        
        # 1. Validate input size
        input_text = json.dumps(params)
        if len(input_text) > self.max_input:
            # Truncate or summarize
            params = self._truncate_input(params, self.max_input)
        
        # 2. Use environment-specific timeout
        try:
            result = await asyncio.wait_for(
                skill.execute(params, session),
                timeout=self.timeout
            )
        except asyncio.TimeoutError:
            if 'local' in self.env['name']:
                return self._error("Model processing slow. Try simpler input.")
            else:
                return self._error("Request timeout. Try again.")
        
        # 3. Validate output fits max_output_tokens
        output_text = json.dumps(result)
        if len(output_text) > self.max_output:
            # Split into multiple responses or summarize
            result = self._chunk_output(result, self.max_output)
        
        return result
    
    def _truncate_input(self, params, max_chars):
        """Safely reduce input while preserving meaning"""
        # Truncate descriptions, keep structure
        if 'description' in params:
            params['description'] = params['description'][:max_chars - 100]
        return params
    
    def _chunk_output(self, result, max_chars):
        """Split large responses across multiple outputs"""
        # Mark as continuation: "(1/3)" in response
        return result
```

---

## Constitution Enforcement Across Environments

**All 16 rules enforced identically:**

```python
class ConstitutionValidator:
    """Validates all outputs against Constitution"""
    
    RULES = {
        1: "task_focus",           # No greetings
        2: "no_guessing",         # Ask clarifying questions
        3: "advise_not_decide",   # Recommendations not commands
        4: "structured_response", # Lists, brief, to the point
        5: "acknowledge_limits",  # Say "I don't know"
        6: "end_with_action",     # Question or next step
        7: "no_full_code",        # Max 10-line snippets
        8: "explain_why",         # Reasoning for recommendations
        9: "public_repos_only",   # GitHub public access only
        10: "no_personal_data",   # Session-only storage
        11: "no_complete_backend",# Fragments with warning
        12: "no_ready_solutions", # Insufficient for self-implementation
        13: "company_redirect",   # End with company contact
        14: "track_bypass",       # Monitor for code requests
        15: "no_code_gen",        # Cognitive architecture limit
        16: "text_only",          # Markdown, JSON, YAML only
    }
    
    async def validate_output(self, output: dict, session: dict) -> dict:
        """Check output against all 16 rules before returning"""
        
        # Rule 7: No full code
        if self._contains_code_block(output):
            if session['tech_question_counter'] >= 3:
                return self._redirect_to_company()
            else:
                output = self._sanitize_code(output)
        
        # Rule 13: Company redirect
        if self._is_technical_response(output):
            output = self._append_company_signature(output)
        
        # Rule 16: Text only
        if not self._is_text_format(output):
            raise ValueError("Output must be text format (Markdown/JSON/YAML)")
        
        return output
```

---

## Testing Cross-Environment Consistency

```python
class CrossEnvConsistencyTests:
    """Verify behavior identical across environments"""
    
    async def test_clarify_project_consistency(self):
        """Same input → same output structure across envs"""
        
        test_input = {'description': 'I want to build a SaaS'}
        
        for env_name in ['chatgpt', 'claude', 'local']:
            orchestrator = self._get_orchestrator(env_name)
            result = await orchestrator.route('test-session', test_input['description'])
            
            # Verify structure
            assert 'questions' in result
            assert 'pm' in result['questions']
            assert 'analyst' in result['questions']
            assert 'consultant' in result['questions']
            assert len(result['questions']['pm']) >= 2
    
    async def test_code_rejection_consistency(self):
        """Code request rejected identically across envs"""
        
        test_messages = [
            'Write login code in Python',
            'Show me authentication implementation',
            'Generate the API endpoints'
        ]
        
        for env_name in ['chatgpt', 'claude', 'local']:
            orchestrator = self._get_orchestrator(env_name)
            
            for msg in test_messages:
                result = await orchestrator.route('test-session', msg)
                # All should redirect to company
                assert 'contact' in result.lower() or 'company' in result.lower()
    
    async def test_github_url_consistency(self):
        """GitHub parsing identical across envs"""
        
        url = 'https://github.com/EvoPyramidini/ep-osa-core'
        
        for env_name in ['chatgpt', 'claude', 'local']:
            orchestrator = self._get_orchestrator(env_name)
            result = await orchestrator.route('test-session', url)
            
            # All should return same structure
            assert 'EvoPyramidini' in result
            assert 'ep-osa-core' in result
            assert 'README' in result or 'readme' in result.lower()
```

---

## Environment-Specific Optimizations

### ChatGPT (Function Calling)

```python
# Can use function_call for strict output format
functions = [
    {
        "name": "clarify_project",
        "parameters": {
            "type": "object",
            "properties": {
                "questions": {"type": "object"}
            }
        }
    }
]

# Then parse function_call result
if response.choices[0].message.function_call:
    result = json.loads(response.choices[0].message.function_call.arguments)
```

### Claude (Larger Context)

```python
# Claude supports 200k context
# Can load more history/examples without truncation
context_window = 200000
system_prompt_size = 10000
reserved_for_output = 4000
available_for_input = context_window - system_prompt_size - reserved_for_output
# Much more room for longer project descriptions
```

### Local (Privacy)

```python
# No API calls to external services
# All processing local, no data leaves machine
# Trade-off: Slower, requires local hardware
# Perfect for: Sensitive/confidential projects
```

---

## Error Handling Across Environments

```python
class CrossEnvErrorHandler:
    
    ERROR_MAPPING = {
        'timeout': {
            'chatgpt': 'Request took too long. Try again in a moment.',
            'claude': 'Processing delayed. Please try again.',
            'local': 'Local model slow. Try simpler input.'
        },
        'rate_limit': {
            'chatgpt': 'Too many requests. Please wait a few minutes.',
            'claude': 'Rate limit reached. Try again later.',
            'local': 'Local rate limit reached. Adjust concurrency.'
        },
        'api_error': {
            'chatgpt': 'OpenAI API error. Try again in 5 minutes.',
            'claude': 'Anthropic API error. Try again in 5 minutes.',
            'local': 'Local model error. Check logs.'
        }
    }
    
    def format_error(self, error_type: str, env_name: str) -> str:
        return self.ERROR_MAPPING[error_type].get(
            env_name,
            'An error occurred. Please try again.'
        )
```

---

## Performance Comparison

```
┌─────────────┬────────┬────────┬────────┬─────────────────────┐
│ Environment │ Latency│ Cost   │ Quality│ Best For            │
├─────────────┼────────┼────────┼────────┼─────────────────────┤
│ ChatGPT     │ 2-3s   │ $$     │ High   │ Production, APIs    │
│ Claude      │ 3-5s   │ $$$    │ High++ │ Complex reasoning   │
│ Local       │ 5-15s  │ None   │ Medium │ Privacy, offline    │
│ Telegram    │ 2-3s   │ $$     │ High   │ Team collaboration  │
│ Slack       │ 2-3s   │ $$     │ High   │ Async workflows     │
│ Copilot     │ 1-2s   │ (IDE)  │ High   │ Developer workflow  │
└─────────────┴────────┴────────┴────────┴─────────────────────┘
```

---

## Migration Between Environments

**User switches from ChatGPT to Claude:**

```
1. Load session from ChatGPT
2. Validate session data compatible
3. Switch orchestrator to Claude adapter
4. Continue conversation seamlessly
5. No re-initialization needed
```

**Session format is environment-agnostic:**
```python
session = {
    'session_id': 'uuid',
    'context': {...},
    'answers': {...},
    'requirements': {...},
    'history': [...]
}
# Works on any environment
```

---

## Recommendations

**Choose environment based on:**

| Use Case | Recommended | Reason |
|----------|-------------|--------|
| Production API | ChatGPT | Fast, reliable, scalable |
| Complex reasoning | Claude | Better at nuanced analysis |
| Privacy critical | Local | No data leaves your system |
| Team async | Slack | Integrated workflow |
| Standalone chat | Telegram | Accessible, bot format |
| IDE-native | Copilot | Developer experience |

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
