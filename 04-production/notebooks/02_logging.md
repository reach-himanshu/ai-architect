# Production 02: Logging Best Practices

Print statements are not professional. The `logging` module provides powerful, configurable logging.

## 1. Basic Usage

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("Application started")
logger.warning("Low memory")
logger.error("Connection failed")
```

## 2. Log Levels
| Level | When to Use |
|:---|:---|
| DEBUG | Detailed diagnostic info |
| INFO | Confirmation that things work |
| WARNING | Something unexpected happened |
| ERROR | A function failed |
| CRITICAL | Program may not continue |

## 3. Formatting

```python
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
```

## 4. File Logging

```python
logging.basicConfig(
    filename='app.log',
    level=logging.INFO
)
```

## 5. Best Practices
- Use `__name__` for logger names.
- Don't log sensitive data (passwords, PII).
- Use structured logging (JSON) for production systems.
- Log at function entry/exit points.
