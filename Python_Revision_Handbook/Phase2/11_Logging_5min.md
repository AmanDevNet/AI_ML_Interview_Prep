# Chapter 11: Logging — "Stop Using print() for Debugging"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Logging is the practice of recording diagnostic messages from an application to a file, console, or external service while it runs. |
| **Why It Exists** | Using `print()` is transient, lacks severity levels (e.g., info vs crash), has no timestamps, and cannot easily write to file/network without complex custom code. |
| **Where It Is Used** | Production applications, web servers (Django/Flask), microservices, CLI tools, debugging system crashes, security audits. |
| **Real-World Analogy** | A flight data recorder (black box) on an airplane. It quietly records everything during a flight so investigators can find the root cause if something goes wrong. |

---

## Think of It This Way

Imagine a user reports a bug in your production app: *"The app crashed when I checked out."*

If you used `print()` statements for debugging:
1. They only go to standard output. In production, stdout might be discarded, or hard to access.
2. You have no timestamp or user context.
3. You cannot see what happened right before the crash because `print` statements don't have severity controls.

If you used **Logging**:
1. You can inspect the logs file `app.log`.
2. You can search for `ERROR` or `CRITICAL` entries around the time of the checkout.
3. You can see the full execution flow leading up to it (`INFO` and `DEBUG` logs).
4. You can see the exact traceback printed automatically.

---

## Step 1: Log Levels

There are 5 standard log levels, in increasing order of severity:

| Level | Value | Used For |
|---|---|---|
| **DEBUG** | 10 | Detailed diagnostic information (variable values, query results). |
| **INFO** | 20 | Normal system events (service started, user logged in). |
| **WARNING** | 30 | Something unexpected happened, but the app is still working. |
| **ERROR** | 40 | A serious issue occurred; some functionality failed. |
| **CRITICAL** | 50 | A fatal error; the entire program is stopping. |

```python
import logging

# Basic Configuration
logging.basicConfig(level=logging.WARNING)

# By default, levels below WARNING (like DEBUG and INFO) are ignored
logging.debug("This is a debug message")       # Not printed
logging.info("This is an info message")        # Not printed
logging.warning("This is a warning message")    # Printed!
logging.error("This is an error message")      # Printed!
logging.critical("This is a critical message")  # Printed!
```

---

## Step 2: Custom Loggers, Handlers & Formatters

To build a professional logging setup, we configure a Logger hierarchy:
1. **Logger:** The entry point where the application sends messages.
2. **Handler:** Sends the log records to the right destination (Console, File, Socket).
3. **Formatter:** Sets the structure of the log message (timestamp, logger name, level, message).

```python
import logging

# 1. Create a custom logger
logger = logging.getLogger("payment_service")
logger.setLevel(logging.DEBUG)  # Set the threshold for this logger

# 2. Create handlers
console_handler = logging.StreamHandler()      # Writes to Console
file_handler = logging.FileHandler("app.log")    # Writes to File

# Set levels on handlers
console_handler.setLevel(logging.INFO)
file_handler.setLevel(logging.DEBUG)

# 3. Create formatters
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# Add formatters to handlers
console_handler.setFormatter(formatter)
file_handler.setFormatter(formatter)

# 4. Add handlers to the logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# Test the setup
logger.debug("Connecting to Database...")      # Written only to 'app.log'
logger.info("Database connection established!") # Written to BOTH console and 'app.log'
logger.error("Transaction failed: Timeout")    # Written to BOTH console and 'app.log'
```

---

## Code Examples

### Easy Examples
```python
# 1. Logging to a file with basicConfig
# Overwrites file each run with filemode='w'
logging.basicConfig(
    filename='basic.log',
    filemode='w',
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logging.info("Application started successfully.")

# 2. Logging exception trackbacks
try:
    x = 1 / 0
except ZeroDivisionError:
    # logging.exception automatically appends stack traceback
    logging.exception("An exception occurred!")

# 3. Disable logging temporarily
logging.disable(logging.CRITICAL)  # Disables ALL logging calls of severity CRITICAL and below
logging.error("This will not be logged.")
logging.disable(logging.NOTSET)    # Reset logging back to normal

# 4. Log with parameters (avoid string formatting)
# The string is only formatted if the message is actually logged (improves speed)
user = "Alice"
logging.info("User %s logged in from IP %s", user, "127.0.0.1")

# 5. Get a child logger
# Useful for tracing module-specific logs (e.g. "app.database")
root_logger = logging.getLogger("app")
db_logger = logging.getLogger("app.database") # child logger
```

### Medium Examples
```python
# 6. Logging with rotating files (prevents log files from growing infinitely)
from logging.handlers import RotatingFileHandler

logger = logging.getLogger("rotating_app")
# Rotates when file hits 1MB, keeps up to 3 backup files
handler = RotatingFileHandler("rotated.log", maxBytes=1_000_000, backupCount=3)
logger.addHandler(handler)

# 7. Logging to console with colors using colorama or standard codes
class ColorFormatter(logging.Formatter):
    GREY = "\x1b[38;21m"
    YELLOW = "\x1b[33;21m"
    RED = "\x1b[31;21m"
    RESET = "\x1b[0m"
    FORMAT = "%(asctime)s - %(levelname)s - %(message)s"

    def format(self, record):
        if record.levelno == logging.WARNING:
            self._style._fmt = self.YELLOW + self.FORMAT + self.RESET
        elif record.levelno >= logging.ERROR:
            self._style._fmt = self.RED + self.FORMAT + self.RESET
        else:
            self._style._fmt = self.GREY + self.FORMAT + self.RESET
        return super().format(record)

# 8. Filter class - only allow warnings that contain 'DB'
class DBWarningFilter(logging.Filter):
    def filter(self, record):
        return "DB" in record.getMessage()

logger = logging.getLogger("db_app")
logger.addFilter(DBWarningFilter())
logger.warning("Something failed.")      # Filtered out
logger.warning("DB connection slow.")   # Allowed!

# 9. TimeRotatingFileHandler (rotates logs at midnight every day)
from logging.handlers import TimedRotatingFileHandler
time_handler = TimedRotatingFileHandler("daily.log", when="midnight", interval=1)

# 10. Capture warnings from warnings module
logging.captureWarnings(True)
import warnings
warnings.warn("This deprecation warning is now captured by logging!")
```

### Advanced Examples
```python
# 11. Configuration via DictConfig (Production standard)
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'DEBUG',
            'formatter': 'standard',
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': 'production.log',
            'level': 'ERROR',
            'formatter': 'standard',
            'maxBytes': 5_000_000,
            'backupCount': 5,
        }
    },
    'loggers': {
        '': {  # root logger
            'handlers': ['console', 'file'],
            'level': 'INFO',
        },
        'django.db': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False, # Stop propagation up to root logger
        }
    }
}

logging.config.dictConfig(LOGGING_CONFIG)
logger = logging.getLogger("django.db")
logger.debug("SELECT * FROM users") # Logged to console, not production.log

# 12. Contextual Logging with Extra (Adding Request IDs or User IDs to logs)
logger = logging.getLogger("api")
formatter = logging.Formatter('%(asctime)s - [%(request_id)s] - %(message)s')
handler = logging.StreamHandler()
handler.setFormatter(formatter)
logger.addHandler(handler)

# Must pass extra dict with the key specified in formatter
logger.info("User accessed cart", extra={"request_id": "REQ-99432"})

# 13. Contextual Logging with LoggerAdapter
# Automates passing 'extra' contextual variables
adapter = logging.LoggerAdapter(logger, {"request_id": "REQ-COMMON"})
adapter.info("Process step 1")
adapter.info("Process step 2")

# 14. Async QueueHandler (highly performance critical apps)
# Offloads blocking file writes to a background thread
from logging.handlers import QueueHandler, QueueListener
import queue

log_queue = queue.Queue(-1)
queue_handler = QueueHandler(log_queue)

# Real handler that actually writes to disk
file_h = logging.FileHandler("async.log")
listener = QueueListener(log_queue, file_h)
listener.start()

# Use queue_handler on your active logger
root = logging.getLogger()
root.addHandler(queue_handler)
root.warning("Quick logged without slowing down main thread!")

listener.stop()

# 15. Customizing Exception Traceback formatting
# Restrict stack trace depth or strip personal/secret data from traces
class ShortExceptionFormatter(logging.Formatter):
    def formatException(self, ei):
        # ei is a tuple (type, value, traceback)
        return f"CRASH: {ei[0].__name__}: {str(ei[1])}"

# Used to log exception message without full dump in stdout
```

---

## Common Mistakes & Edge Cases

### 1. Duplicate Logs due to Propagation
If you add a handler to a child logger (e.g. `app.db`) and the parent logger (e.g. `app`) also has a handler, log messages sent to `app.db` will propagate up to `app` and print **twice**.
* **Fix:** Set `logger.propagate = False` on the child logger.

### 2. Using f-strings inside Logging Calls
* **Bad:** `logging.debug(f"User data: {fetch_huge_data()}")` (Calculates the f-string even if the log level is set to WARNING, causing CPU/Memory overhead).
* **Good:** `logging.debug("User data: %s", fetch_huge_data)` (Calculates the string representation ONLY if debug messages are allowed).

### 3. Calling `logging.basicConfig()` multiple times
`basicConfig` only configures the root logger **once**. Subsequent calls do nothing. Use `dictConfig` or clear existing handlers from the root logger before re-configuring.

---

## Interview Questions (Top 5)

**Q1: Why is `print()` bad for logging in production applications?**
> `print()` writes only to standard output (`sys.stdout`), making redirecting, filtering, or rotating logs hard. It has no built-in timestamp, severity classification (e.g. debug vs fatal error), or module metadata. It blocks the execution thread during operations, whereas logging can be configured asynchronously or written to multiple targets (files, consoles, databases) with a simple configuration change.

**Q2: What is Logger propagation and how do you disable it?**
> Logger propagation is when a log message sent to a child logger is automatically sent to all handlers of its ancestor loggers. You disable it by setting `logger.propagate = False` on the child logger.

**Q3: How do you log a full exception traceback in Python?**
> By calling `logging.exception("message")` inside an `except` block. This is shorthand for `logging.error("message", exc_info=True)`. Setting `exc_info=True` tells the logger to append the full traceback information to the log message.

**Q4: How does a `RotatingFileHandler` work and why is it useful?**
> It manages the log file size by rolling it over to a backup file once it reaches a certain size limit (`maxBytes`). It keeps a fixed number of backup files (`backupCount`), deleting the oldest ones. This prevents your server's disk space from being filled up by runaway logs.

**Q5: What is the difference between `logging.basicConfig()` and `logging.config.dictConfig()`?**
> `basicConfig` is a simple helper function to quickly configure root logging (useful for small scripts). `dictConfig` accepts a dictionary containing logger hierarchies, handlers, formatters, and filters. It is the enterprise standard because it is completely configurable and allows loading layouts from JSON or YAML files.

---

*← [Previous: Chapter 10 (Regex)](10_Regex.md) | [Next: Chapter 12 (Virtual Environments & Requirements) →](12_Virtual_Environments_Requirements.md)*
