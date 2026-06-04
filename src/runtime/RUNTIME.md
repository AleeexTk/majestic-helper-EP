# Runtime — Safe Execution Environment (Layer 4)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Overview

Runtime provides:
- Resource limits (timeout, memory, output size)
- Execution isolation (sandboxing)
- Error handling (graceful degradation)
- BusinessGuard integration (output filtering)

---

## Resource Limits

### Per-Contract Limits

```python
class ResourceLimits:
    LIMITS = {
        'clarify_project': {
            'timeout_seconds': 15,
            'memory_mb': 256,
            'output_tokens': 500
        },
        'analyze_requirements': {
            'timeout_seconds': 30,
            'memory_mb': 512,
            'output_tokens': 1000
        },
        'suggest_roadmap': {
            'timeout_seconds': 20,
            'memory_mb': 384,
            'output_tokens': 800
        },
        'identify_risks': {
            'timeout_seconds': 20,
            'memory_mb': 384,
            'output_tokens': 800
        },
        'generate_artifacts': {
            'timeout_seconds': 25,
            'memory_mb': 512,
            'output_tokens': 1200
        },
        'prepare_meeting': {
            'timeout_seconds': 15,
            'memory_mb': 256,
            'output_tokens': 600
        },
        'evaluate_idea': {
            'timeout_seconds': 20,
            'memory_mb': 384,
            'output_tokens': 800
        },
        'consult_on_process': {
            'timeout_seconds': 20,
            'memory_mb': 384,
            'output_tokens': 1000
        },
        'handle_github_url': {
            'timeout_seconds': 10,
            'memory_mb': 256,
            'output_tokens': 300
        }
    }
```

### Global Limits

```python
class GlobalLimits:
    MAX_SESSION_DURATION = 30 * 60  # 30 minutes
    MAX_MESSAGES_PER_SESSION = 50
    MAX_MEMORY_PER_SESSION = 10 * 1024 * 1024  # 10 MB
    MAX_EXTERNAL_API_CALLS = 100  # per session
    RATE_LIMIT_REQUESTS_PER_MINUTE = 60
    SESSION_EXPIRY_AFTER_INACTIVITY = 30 * 60  # 30 minutes
```

---

## Execution Isolation

### No File System Access

```python
class ExecutionSandbox:
    def __init__(self):
        self.allowed_reads = []  # No file read access
        self.allowed_writes = []  # No file write access
        self.blocked_imports = ['os', 'sys', 'subprocess', 'socket']
    
    def validate_environment(self):
        # Verify no unauthorized imports
        for module in self.blocked_imports:
            if module in sys.modules:
                raise SecurityError(f"Unauthorized module: {module}")
```

### No Network Access (except allowed APIs)

```python
class NetworkPolicy:
    ALLOWED_ENDPOINTS = [
        'api.openai.com',
        'api.anthropic.com',
        'api.github.com',  # Read-only, public repos
    ]
    
    def validate_request(self, url: str):
        parsed = urllib.parse.urlparse(url)
        if parsed.hostname not in self.ALLOWED_ENDPOINTS:
            raise SecurityError(f"Unauthorized endpoint: {parsed.hostname}")
```

### No Subprocess Execution

```python
def validate_no_execution(code: str):
    forbidden = ['exec(', 'eval(', 'subprocess.', 'os.system(', 'open(']
    for keyword in forbidden:
        if keyword in code:
            raise SecurityError(f"Execution forbidden: {keyword}")
```

---

## Error Handling

### Standard Error Response Format

```python
class ErrorHandler:
    @staticmethod
    def format_error(error_type: str, contract: str, details: str = None) -> dict:
        return {
            'status': 'error',
            'type': error_type,
            'contract': contract,
            'message': ERROR_MESSAGES.get(error_type, 'Unknown error'),
            'details': details,
            'next_suggested_action': ERROR_ACTIONS.get(error_type, 'Try again')
        }

ERROR_MESSAGES = {
    'EMPTY_INPUT': 'Please provide more information.',
    'SCHEMA_VALIDATION_FAILED': 'Input format incorrect.',
    'TIMEOUT': 'Request took too long. Please try again.',
    'RATE_LIMIT': 'Service busy. Please wait a moment.',
    'API_ERROR': 'External service error. Try again in a few minutes.',
    'INTERNAL_ERROR': 'Internal error. Support team notified.'
}

ERROR_ACTIONS = {
    'EMPTY_INPUT': 'Provide more details about your project.',
    'SCHEMA_VALIDATION_FAILED': 'Check the format of your input.',
    'TIMEOUT': 'Try a simpler question or check your connection.',
    'RATE_LIMIT': 'Wait a minute and try again.',
    'API_ERROR': 'Try again in 5 minutes.',
    'INTERNAL_ERROR': 'Contact support@example.com'
}
```

### Error Isolation

```python
class ErrorIsolation:
    """Errors in one contract don't affect others"""
    
    async def execute_with_isolation(self, skill, params, session):
        try:
            return await skill.execute(params, session)
        except Exception as e:
            # Log error
            self.logger.error(f"Skill execution failed: {skill.__class__.__name__}", exc_info=e)
            # Return user-friendly error
            return ErrorHandler.format_error(
                'INTERNAL_ERROR',
                skill.contract['name'],
                details=None  # Never leak internal error to user
            )
```

---

## Timeout Handling

```python
import asyncio

class TimeoutManager:
    async def execute_with_timeout(self, coro, timeout_seconds: int):
        try:
            result = await asyncio.wait_for(
                coro,
                timeout=timeout_seconds
            )
            return result
        except asyncio.TimeoutError:
            return {
                'status': 'error',
                'type': 'TIMEOUT',
                'message': f'Request exceeded {timeout_seconds}s limit.',
                'next_suggested_action': 'Try again or contact support.'
            }
```

---

## BusinessGuard Integration

```python
class RuntimeWithGuard:
    def __init__(self, business_guard):
        self.guard = business_guard
    
    async def execute_contract(self, skill, params, session):
        # Execute skill
        result = await skill.execute(params, session)
        
        # If success, apply guard
        if result.get('status') != 'error':
            result = self.guard.filter_response(result, session)
        
        return result
```

---

## Monitoring & Metrics

```python
class RuntimeMetrics:
    def __init__(self):
        self.execution_times = {}
        self.error_counts = {}
        self.success_counts = {}
    
    async def track_execution(self, contract_name: str, coro):
        start = time.time()
        try:
            result = await coro
            duration_ms = (time.time() - start) * 1000
            self.execution_times[contract_name] = duration_ms
            self.success_counts[contract_name] = self.success_counts.get(contract_name, 0) + 1
            return result
        except Exception as e:
            self.error_counts[contract_name] = self.error_counts.get(contract_name, 0) + 1
            raise
    
    def report_metrics(self) -> dict:
        return {
            'execution_times': self.execution_times,
            'error_counts': self.error_counts,
            'success_counts': self.success_counts,
            'total_executions': sum(self.success_counts.values()) + sum(self.error_counts.values())
        }
```

---

## No Code Execution Policy

```python
class NoCodeExecutionPolicy:
    """Runtime executes ZERO user code. All output is text."""
    
    def validate_output(self, output: any):
        """
        Verify output is safe text only.
        No evaluation, no execution, no serialization of callables.
        """
        
        forbidden_types = [types.FunctionType, types.CodeType, bytes]
        
        if isinstance(output, dict):
            for value in output.values():
                if type(value) in forbidden_types:
                    raise TypeError(f"Output contains forbidden type: {type(value)}")
        
        # Output is always stringifiable to Markdown/JSON/YAML
        try:
            if isinstance(output, dict):
                json.dumps(output)  # Verify JSON serializable
            elif isinstance(output, str):
                pass  # Strings are safe
            else:
                str(output)  # Convert to string
        except Exception as e:
            raise ValueError(f"Output not safely serializable: {e}")
```

---

## Async-First Architecture

```python
class AsyncRuntime:
    """All I/O operations are non-blocking async"""
    
    async def execute_parallel_skills(self, skills_list: list, params_list: list):
        """Execute multiple skills concurrently"""
        tasks = [
            skill.execute(params, {})
            for skill, params in zip(skills_list, params_list)
        ]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        return results
```

---

## Resource Cleanup

```python
class ResourceCleanup:
    """Guaranteed cleanup on success or failure"""
    
    async def execute_with_cleanup(self, skill, params, session):
        resources = []
        try:
            result = await skill.execute(params, session)
            return result
        finally:
            # Always cleanup
            for resource in resources:
                resource.close()
            
            # Log cleanup
            logger.info(f"Cleaned up {len(resources)} resources")
```

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
