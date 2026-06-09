# Env verification pitfall for Hermes research setup

## What happened

A terminal-based check that echoed `.env`-style lines was misleading after secret redaction. It looked like multiple research API keys existed even though they were not actually present in `~/.hermes/.env`.

## Durable lesson

When verifying research-tool readiness in Hermes:

1. Treat `hermes config env-path` as the path of record.
2. Expect `read_file` on `~/.hermes/.env` to be blocked because it is a credential store.
3. If inspection is necessary from terminal, parse only:
   - variable name
   - whether a value is present / empty
4. Check file presence separately from process-environment presence.
5. Never conclude that a key exists from redacted terminal output alone.

## Safe terminal pattern

Use a parser that emits only key names and `set` / `empty`:

```python
from pathlib import Path
p = Path.home() / '.hermes' / '.env'
for line in p.read_text().splitlines():
    if not line.strip() or line.lstrip().startswith('#'):
        continue
    if '=' in line:
        k, v = line.split('=', 1)
        print(f'{k}={'empty' if v == '' else 'set'}')
```

Then separately check the live process environment:

```python
import os
for k in ['PARALLEL_API_KEY', 'CAMOFOX_URL']:
    print(k, 'present-in-process-env' if os.getenv(k) else 'missing-in-process-env')
```

## Why this matters

Research setup questions often hinge on whether a provider is truly configured. A false positive creates bad guidance about which workflow is available right now.
