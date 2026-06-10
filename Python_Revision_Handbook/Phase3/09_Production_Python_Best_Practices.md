# Chapter 9: Production Python Best Practices — "Writing Enterprise Code"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | Production best practices define the tooling, testing structures, and configuration layouts required to build robust, maintainable Python applications. |
| **Why It Exists** | Code that works on a local laptop often breaks in production due to different dependency versions, missing packages, or untyped inputs. |
| **Where It Is Used** | CI/CD build scripts, Docker deployment builds, package publishing, and testing automation suites. |
| **Real-World Analogy** | **Pyproject.toml:** A blueprint of an airplane listing all required materials and dimensions before building. **Pytest Mocking:** Crash-test dummies. Instead of crashing a real car on the highway (running tests against production databases), you use a controlled environment and dummies (mocks) to simulate the crash. |

---

## Think of It This Way

### 1. Modern Packaging: `pyproject.toml`
Historically, Python projects used a mess of configuration files: `setup.py`, `setup.cfg`, `requirements.txt`, and `tox.ini`. 
Under PEP 518, Python standardized all build and tool configurations into a single file: **`pyproject.toml`**.
This file defines which build backend to use (Poetry, Hatch, Flit) and lists metadata, dependencies, and lint configurations in a unified format.

### 2. Dependency Locking with `uv`
`uv` is an extremely fast Python package installer and resolver written in Rust. It serves as a drop-in replacement for standard `pip` and `virtualenv`. 
Instead of waiting minutes for dependency resolution, `uv` resolves and locks package trees in milliseconds, generating a cryptographic lockfile (`uv.lock`) for reproducible deployments.

---

## Step 1: Anatomy of a Modern `pyproject.toml`

Here is the standard packaging structure:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "ai_agent_core"
version = "0.1.0"
description = "Production AI Agent tools"
dependencies = [
    "pydantic>=2.0",
    "orjson>=3.9"
]

[tool.pytest.ini_options]
minversion = "7.0"
addopts = "-ra -q"
testpaths = ["tests"]
```

---

## Step 2: Advanced Testing Patterns with `pytest`

### 1. Fixture Scopes
Pytest fixtures set up state for tests. You can control how often a fixture runs using scopes:
* **`function` (default):** Runs once per test function.
* **`class`:** Runs once per test class.
* **`module`:** Runs once per Python test file.
* **`session`:** Runs once for the entire test execution suite.

```python
import pytest

@pytest.fixture(scope="session")
def db_connection():
    # Setup runs ONCE for the entire test run
    conn = "Connected"
    yield conn
    # Teardown runs at the very end
    conn = "Closed"
```

### 2. Monkeypatching & Mocking API Calls
Never hit real external APIs (like OpenAI or database endpoints) during unit tests. It is slow, costs money, and fails if you are offline. Use `monkeypatch` or `unittest.mock` to intercept and mock calls:

```python
import httpx

def get_agent_response(prompt):
    # Hits real external API
    response = httpx.get(f"https://api.openai.com/v1/agent?q={prompt}")
    return response.json()

# --- Test File ---
def test_agent_response(monkeypatch):
    # 1. Define a dummy mock return function
    def mock_get(*args, **kwargs):
        class MockResponse:
            def json(self):
                return {"reply": "Mocked AI output"}
        return MockResponse()

    # 2. Intercept httpx.get and swap it with our mock_get
    monkeypatch.setattr(httpx, "get", mock_get)

    result = get_agent_response("hello")
    assert result["reply"] == "Mocked AI output" # ✅ Pass without internet!
```

---

## Code Examples

### Easy Examples
```python
# 1. Parameterized Pytest Tests
# Runs the same test code multiple times with different inputs
import pytest

def add(a, b): return a + b

@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3),
    (5, 5, 10),
    (-1, 1, 0)
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

### Medium Examples
```python
# 2. Mocking Context Managers
from unittest.mock import patch

class ServerTunnel:
    def __enter__(self):
        return self
    def send(self, data):
        return "Real Send"
    def __exit__(self, *args):
        pass

def send_payload():
    with ServerTunnel() as tunnel:
        return tunnel.send("payload")

# Test file
def test_send_payload():
    with patch("__main__.ServerTunnel.send", return_value="Mocked Send"):
        assert send_payload() == "Mocked Send"
```

### Advanced Examples
```python
# 3. Dynamic Mocking of Async Methods
# Testing async API calls requires Mocking Coroutines
import asyncio
import pytest

class AsyncClient:
    async def get_data(self):
        await asyncio.sleep(1)
        return "Real Data"

# Test file using pytest-asyncio
@pytest.mark.asyncio
async def test_async_client(monkeypatch):
    async def mock_get_data(self):
        return "Mocked Data"

    monkeypatch.setattr(AsyncClient, "get_data", mock_get_data)
    client = AsyncClient()
    result = await client.get_data()
    assert result == "Mocked Data"
```

---

## Common Mistakes & Edge Cases

### 1. Mutable default values inside pytest fixtures
If a fixture returns a mutable list (e.g. `return [1, 2]`) and is configured with `scope="session"`, any test that modifies this list (like `list.append(3)`) will corrupt the list for all subsequent tests, leading to flaky test suites.
* **Fix:** If a fixture yields a mutable object, use `scope="function"` so each test gets a clean, independent instance.

---

## Interview Questions (Top 5)

**Q1: What is the purpose of `pyproject.toml` and how does PEP 518 change Python packaging?**
> Prior to PEP 518, Python build configurations were fragmented (requiring `setup.py`, `setup.cfg`, etc.), and Python had to execute `setup.py` scripts to discover dependencies, creating build-security loops. `pyproject.toml` standardizes tool and build metadata into a declarative, static file. It lists the exact build backend (like Hatch or Poetry) and project dependencies in a standard format, ensuring secure and reproducible builds.

**Q2: What is the difference between a pytest fixture scoped as `function` vs one scoped as `session`?**
> * **`function` scope:** The fixture is executed fresh before every single test function that requests it. This guarantees complete test isolation.
> * **`session` scope:** The fixture is executed only once at the very beginning of the entire test run. The same resulting object is shared among all tests. This is ideal for expensive initialization steps, like spinning up a Docker test database or loading model weights.

**Q3: How do you mock an external HTTP API client in pytest to prevent unit tests from making network calls?**
> You can mock it using the `monkeypatch` fixture or `unittest.mock.patch`. By calling `monkeypatch.setattr(client, "get", mock_method)`, you swap the real HTTP request method with an in-memory mock function that returns a dummy response object containing pre-defined JSON data.

**Q4: Explain how you mock an async function (coroutine) in Python tests.**
> In modern Python (3.8+), `unittest.mock.AsyncMock` is used. Standard mocks return immediately, but async functions must return a coroutine that must be awaited. If you mock an async function with `AsyncMock`, calling it returns an awaitable object that resolves to the specified return value when awaited:
> `mock_client.get_data = AsyncMock(return_value="mocked_data")`.

**Q5: Why is `uv` preferred over standard `pip` for locking dependencies in production deployments?**
> Standard `pip` has a slow dependency resolver written in Python that fetches and analyzes packages sequentially, which can take minutes. `uv` is written in Rust, resolves dependencies concurrently, and uses an in-memory global cache. It generates a lockfile (`uv.lock`) with cryptographic package hashes, ensuring absolute environment parity during CI/CD builds and production deployments in milliseconds.

---

*← [Previous: Chapter 8 (LangChain, RAG & MCP)](08_LangChain_RAG_MCP.md) | [Next: Chapter 10 (Phase 3 Interview Handbook) →](10_Phase3_Interview_Handbook.md)*
