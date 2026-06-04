# Memory & Tracing (Layers 8 & 7)

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04

---

## Layer 8: Session Memory

### Memory Characteristics

- **Scope:** Session-only (no persistence after 30 min expiry)
- **Storage:** In-memory dictionary/cache
- **Access:** Fast (O(1) lookup)
- **Expiry:** 30 minutes of inactivity
- **Privacy:** No personal data stored

### Session Context Structure

```python
class SessionContext:
    def __init__(self, session_id: str):
        self.session_id = session_id
        self.created_at = datetime.now()
        self.last_active = datetime.now()
        
        # Memory allocation (proportions)
        self.primary_memory = {}      # 50-60%: Active operational state
        self.buffer_layer = {}         # 30%: Transitional processing
        self.reserve_memory = {}       # 10%: Critical backup
        
        # Session data
        self.context = {}
        self.answers = {}
        self.requirements = {}
        self.history = []              # Last 3-10 exchanges
        self.last_contract = None
        self.tech_question_counter = 0
    
    def is_expired(self) -> bool:
        """Check if session expired after 30 min inactivity"""
        return (datetime.now() - self.last_active).total_seconds() > 30 * 60
    
    def touch(self):
        """Update last_active timestamp"""
        self.last_active = datetime.now()
```

### Session Storage

```python
class SessionStore:
    """In-memory session storage with cleanup"""
    
    def __init__(self, cleanup_interval_seconds=60):
        self.sessions = {}  # session_id -> SessionContext
        self.cleanup_interval = cleanup_interval_seconds
        self.cleanup_task = None
    
    async def get_or_create(self, session_id: str) -> dict:
        """Get existing session or create new"""
        if session_id in self.sessions:
            self.sessions[session_id].touch()
            return self.sessions[session_id]
        
        session = SessionContext(session_id)
        self.sessions[session_id] = session
        return session
    
    async def update(self, session_id: str, updates: dict) -> None:
        """Update session data"""
        if session_id not in self.sessions:
            return
        
        session = self.sessions[session_id]
        session.touch()
        
        # Update fields
        for key, value in updates.items():
            if hasattr(session, key):
                setattr(session, key, value)
    
    async def cleanup_expired(self) -> int:
        """Remove expired sessions, return count"""
        expired_ids = [
            sid for sid, session in self.sessions.items()
            if session.is_expired()
        ]
        
        for sid in expired_ids:
            del self.sessions[sid]
        
        return len(expired_ids)
    
    async def reset_session(self, session_id: str) -> None:
        """User-requested reset (clear session)"""
        if session_id in self.sessions:
            del self.sessions[session_id]
```

### Memory Anchors

**Anchors enable navigation between related concepts:**

```python
class MemoryAnchor:
    """Semantic landmark in session memory"""
    
    def __init__(self, anchor_type: str, reference: str, context: dict):
        self.type = anchor_type  # 'issue', 'pr', 'commit', 'decision'
        self.reference = reference  # URL or ID
        self.context = context
        self.created_at = datetime.now()

class SessionAnchors:
    def __init__(self):
        self.anchors = []
    
    def add(self, anchor_type: str, reference: str, context: dict = None):
        anchor = MemoryAnchor(anchor_type, reference, context or {})
        self.anchors.append(anchor)
    
    def find_by_type(self, anchor_type: str) -> list:
        return [a for a in self.anchors if a.type == anchor_type]
    
    def find_related(self, reference: str) -> list:
        """Find anchors related to a reference"""
        return [a for a in self.anchors if reference in str(a.context)]
```

---

## Layer 7: Tracing & Observability

### Trace Logging Strategy

**JSONL format (one JSON object per line):**

```
{"timestamp": "2026-06-04T10:00:00Z", "session_id": "...", ...}
{"timestamp": "2026-06-04T10:00:05Z", "session_id": "...", ...}
{"timestamp": "2026-06-04T10:00:10Z", "session_id": "...", ...}
```

### Trace Entry Structure

```python
class TraceEntry:
    def __init__(
        self,
        session_id: str,
        contract: str,
        input_size: int,
        output_size: int,
        duration_ms: int,
        success: bool,
        error_type: str = None,
        tech_counter: int = 0
    ):
        self.timestamp = datetime.now().isoformat() + 'Z'
        self.session_id = session_id
        self.contract = contract
        self.input_size = input_size
        self.output_size = output_size
        self.duration_ms = duration_ms
        self.success = success
        self.error_type = error_type
        self.tech_counter = tech_counter
    
    def to_json(self) -> str:
        return json.dumps({
            'timestamp': self.timestamp,
            'session_id': self.session_id,
            'contract': self.contract,
            'input_size': self.input_size,
            'output_size': self.output_size,
            'duration_ms': self.duration_ms,
            'success': self.success,
            'error_type': self.error_type,
            'tech_counter': self.tech_counter
        })
```

### Trace Logger

```python
class TraceLogger:
    """JSONL logger for all contract executions"""
    
    def __init__(self, log_file: str = 'logs/trace.jsonl'):
        self.log_file = log_file
        self.buffer = []
        self.buffer_size = 10
        self.lock = asyncio.Lock()
    
    async def log_execution(self, entry: TraceEntry) -> None:
        """Log a contract execution"""
        async with self.lock:
            self.buffer.append(entry.to_json())
            
            # Flush if buffer full
            if len(self.buffer) >= self.buffer_size:
                await self._flush()
    
    async def _flush(self) -> None:
        """Write buffered entries to file"""
        if not self.buffer:
            return
        
        try:
            with open(self.log_file, 'a') as f:
                for entry in self.buffer:
                    f.write(entry + '\n')
            self.buffer = []
        except IOError as e:
            logger.error(f"Failed to write trace log: {e}")
    
    async def rotate_logs(self) -> None:
        """Daily log rotation, keep 7 days"""
        # Flush current buffer
        async with self.lock:
            await self._flush()
        
        # Archive old logs
        # Keep logs for 7 days max
```

### Trace Analysis

```python
class TraceAnalyzer:
    """Analyze trace logs for insights"""
    
    def __init__(self, log_file: str = 'logs/trace.jsonl'):
        self.log_file = log_file
        self.entries = []
        self.load_entries()
    
    def load_entries(self) -> None:
        """Load all trace entries from file"""
        try:
            with open(self.log_file, 'r') as f:
                for line in f:
                    self.entries.append(json.loads(line))
        except FileNotFoundError:
            pass
    
    def get_metrics(self) -> dict:
        """Get aggregate metrics"""
        if not self.entries:
            return {}
        
        by_contract = {}
        for entry in self.entries:
            contract = entry['contract']
            if contract not in by_contract:
                by_contract[contract] = {
                    'count': 0,
                    'success': 0,
                    'errors': 0,
                    'total_duration_ms': 0,
                    'avg_duration_ms': 0
                }
            
            by_contract[contract]['count'] += 1
            by_contract[contract]['total_duration_ms'] += entry['duration_ms']
            
            if entry['success']:
                by_contract[contract]['success'] += 1
            else:
                by_contract[contract]['errors'] += 1
        
        # Calculate averages
        for contract in by_contract:
            count = by_contract[contract]['count']
            by_contract[contract]['avg_duration_ms'] = (
                by_contract[contract]['total_duration_ms'] / count
            )
        
        return {
            'total_executions': len(self.entries),
            'by_contract': by_contract,
            'success_rate': sum(1 for e in self.entries if e['success']) / len(self.entries)
        }
    
    def detect_abuse_patterns(self) -> list:
        """Detect potential abuse (high tech_counter per session)"""
        by_session = {}
        
        for entry in self.entries:
            session_id = entry['session_id']
            if session_id not in by_session:
                by_session[session_id] = 0
            by_session[session_id] = max(by_session[session_id], entry['tech_counter'])
        
        # Sessions with 3+ technical questions
        suspicious = [
            (sid, count) for sid, count in by_session.items()
            if count >= 3
        ]
        
        return suspicious
```

### What's NOT Logged

✅ **Safe to log:**
- Timestamps (ISO 8601)
- Session IDs (opaque UUIDs)
- Contract names
- Input/output sizes
- Duration
- Success/failure status
- Error types (no error messages)
- Tech question counter

❌ **Never logged:**
- User message content
- Generated response text
- Personal data (names, emails, companies)
- GitHub URLs
- Sensitive conversation details

---

## Memory Cleanup

```python
class MemoryCleanup:
    """Automatic cleanup of expired sessions"""
    
    def __init__(self, session_store: SessionStore):
        self.store = session_store
        self.cleanup_interval = 60  # seconds
        self.running = False
    
    async def start(self) -> None:
        """Start background cleanup task"""
        self.running = True
        asyncio.create_task(self._cleanup_loop())
    
    async def stop(self) -> None:
        """Stop cleanup task"""
        self.running = False
    
    async def _cleanup_loop(self) -> None:
        """Periodic cleanup of expired sessions"""
        while self.running:
            try:
                count = await self.store.cleanup_expired()
                if count > 0:
                    logger.info(f"Cleaned up {count} expired sessions")
            except Exception as e:
                logger.error(f"Cleanup error: {e}")
            
            await asyncio.sleep(self.cleanup_interval)
```

---

## Privacy Compliance

**GDPR-Friendly Architecture:**

1. **No long-term storage:** Sessions expire after 30 min
2. **No personal data:** No names, emails, addresses stored
3. **Right to deletion:** `session.reset()` immediately clears data
4. **Audit trail:** Logs contain only metadata (no content)
5. **Data minimization:** Only essential session state stored

---

**Version:** 1.0-alpha  
**Status:** Foundation Phase  
**Last Updated:** 2026-06-04
