# Adaptation — Multi-Environment Support (Layer 10)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Overview

Agent core is **environment-agnostic**. Only input/output adapters change based on deployment environment.

**Core logic:** Identical across all environments  
**Adapters:** Different for each environment (ChatGPT, Claude, Copilot, etc.)

---

## Adapter Architecture

### Base Adapter

```python
class BaseAdapter:
    """All environment adapters inherit from this"""
    
    def __init__(self, orchestrator, config: dict):
        self.orchestrator = orchestrator
        self.config = config
        self.max_retries = config.get('max_retries', 3)
    
    async def handle(self, raw_input: any, session_id: str) -> any:
        """
        Main entry point for each environment.
        
        Steps:
        1. Extract message from environment-specific format
        2. Route through orchestrator
        3. Format response back to environment format
        """
        try:
            # Normalize input
            message = self._extract_message(raw_input)
            
            # Route through orchestrator
            response_text = await self.orchestrator.route(session_id, message)
            
            # Format output for environment
            return self._format_output(response_text)
        
        except Exception as e:
            logger.error(f"Adapter error: {e}", exc_info=e)
            return self._format_error(str(e))
    
    def _extract_message(self, raw_input: any) -> str:
        """Override in subclass to extract message"""
        raise NotImplementedError
    
    def _format_output(self, text: str) -> any:
        """Override in subclass to format output"""
        raise NotImplementedError
    
    def _format_error(self, error_msg: str) -> any:
        """Override in subclass to format error"""
        raise NotImplementedError
```

---

## Environment Adapters

### 1. ChatGPT Adapter

```python
class ChatGPTAdapter(BaseAdapter):
    """Adapter for OpenAI ChatGPT API"""
    
    def __init__(self, orchestrator, config: dict):
        super().__init__(orchestrator, config)
        self.api_key = config['openai_api_key']
        self.model = config.get('model', 'gpt-4-turbo')
    
    def _extract_message(self, raw_input: dict) -> str:
        """Extract from OpenAI API format"""
        if isinstance(raw_input, dict) and 'message' in raw_input:
            return raw_input['message']
        return str(raw_input)
    
    def _format_output(self, text: str) -> dict:
        """Format as OpenAI response"""
        return {
            'response': text,
            'role': 'assistant',
            'content_type': 'text'
        }
```

### 2. Claude Adapter

```python
class ClaudeAdapter(BaseAdapter):
    """Adapter for Anthropic Claude API"""
    
    def __init__(self, orchestrator, config: dict):
        super().__init__(orchestrator, config)
        self.api_key = config['anthropic_api_key']
        self.model = config.get('model', 'claude-3-opus')
    
    def _extract_message(self, raw_input: dict) -> str:
        """Extract from Claude format"""
        if 'text' in raw_input:
            return raw_input['text']
        return str(raw_input)
    
    def _format_output(self, text: str) -> dict:
        """Format as Claude response"""
        return {
            'content': [{'type': 'text', 'text': text}],
            'stop_reason': 'end_turn'
        }
```

### 3. GitHub Copilot Adapter

```python
class CopilotAdapter(BaseAdapter):
    """Adapter for GitHub Copilot (IDE chat)"""
    
    def _extract_message(self, raw_input: str) -> str:
        """Extract from IDE chat message"""
        # Raw input is plain text from IDE
        return raw_input.strip()
    
    def _format_output(self, text: str) -> str:
        """Format as Markdown comment"""
        return f"```markdown\n{text}\n```"
```

### 4. Local Ollama Adapter

```python
class LocalAdapter(BaseAdapter):
    """Adapter for local Ollama instance"""
    
    def __init__(self, orchestrator, config: dict):
        super().__init__(orchestrator, config)
        self.base_url = config.get('base_url', 'http://localhost:11434')
        self.model = config.get('model', 'llama2')
    
    def _extract_message(self, raw_input: str) -> str:
        """Extract from plain text"""
        return raw_input.strip()
    
    def _format_output(self, text: str) -> str:
        """Format as plain text"""
        return text
```

### 5. Telegram Bot Adapter

```python
class TelegramAdapter(BaseAdapter):
    """Adapter for Telegram Bot API"""
    
    def __init__(self, orchestrator, config: dict):
        super().__init__(orchestrator, config)
        self.bot_token = config['telegram_bot_token']
    
    def _extract_message(self, update: dict) -> str:
        """Extract from Telegram update"""
        return update['message']['text']
    
    def _format_output(self, text: str) -> dict:
        """Format as Telegram reply"""
        return {
            'text': text,
            'parse_mode': 'Markdown',
            'disable_web_page_preview': True
        }
```

### 6. Slack Adapter

```python
class SlackAdapter(BaseAdapter):
    """Adapter for Slack App API"""
    
    def __init__(self, orchestrator, config: dict):
        super().__init__(orchestrator, config)
        self.signing_secret = config['slack_signing_secret']
    
    def _extract_message(self, event: dict) -> str:
        """Extract from Slack event"""
        return event['text']
    
    def _format_output(self, text: str) -> dict:
        """Format as Slack message"""
        return {
            'text': text,
            'mrkdwn': True
        }
```

---

## Environment Manifest

**Each environment declares its capabilities:**

```yaml
# environments/chatgpt/manifest.yaml
name: chatgpt
max_input_tokens: 8000
max_output_tokens: 4000
supports_system_message: true
supports_functions: true
rate_limit_rpm: 3500
timeout_seconds: 60

# environments/local/manifest.yaml
name: local
max_input_tokens: 4000
max_output_tokens: 2000
supports_system_message: true
supports_functions: false
rate_limit_rpm: unlimited
timeout_seconds: 120  # Slower local model
```

### Runtime Adapts Based on Manifest

```python
class CapabilityAwareRuntime:
    def __init__(self, manifest: dict):
        self.manifest = manifest
    
    async def execute(self, skill, params, session):
        # Truncate input if exceeds max_input_tokens
        if 'max_input_tokens' in self.manifest:
            max_tokens = self.manifest['max_input_tokens']
            # Implement truncation logic
        
        # Use environment-specific timeout
        timeout = self.manifest.get('timeout_seconds', 30)
        
        try:
            result = await asyncio.wait_for(
                skill.execute(params, session),
                timeout=timeout
            )
            return result
        except asyncio.TimeoutError:
            # Different message for different environments
            if 'local' in self.manifest['name']:
                return {'error': 'Local model processing slow. Try simpler input.'}
            else:
                return {'error': 'Request timeout. Try again.'}
```

---

## Configuration Management

```python
class ConfigManager:
    """Load environment-specific configs"""
    
    def __init__(self, env_name: str):
        config_file = f'environments/{env_name}/config.yaml'
        with open(config_file, 'r') as f:
            self.config = yaml.safe_load(f)
    
    def get(self, key: str, default=None):
        return self.config.get(key, default)
    
    def validate(self):
        """Verify required keys present"""
        required_keys = ['name', 'max_input_tokens', 'max_output_tokens']
        for key in required_keys:
            if key not in self.config:
                raise ValueError(f"Missing required config: {key}")
```

---

## No Environment Leakage

**Core guarantees:**

1. **Agent doesn't know which environment it runs on**
   - Orchestrator receives normalized (session_id, message)
   - Orchestrator returns normalized response text

2. **Logic identical across environments**
   - Same contracts executed
   - Same skills invoked
   - Same BusinessGuard filters applied

3. **Adapters are stateless**
   - Only translate format
   - No business logic in adapters

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
