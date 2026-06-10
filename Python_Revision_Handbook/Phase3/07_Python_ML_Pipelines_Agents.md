# Chapter 7: Python in ML Pipelines & AI Agents — "Production AI Code"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **ML Pipelines** stream and process data dynamically for model training/inference. **AI Agents** are event-driven execution loops that call tools and maintain state to solve problems. |
| **Why It Exists** | Loading large datasets (like 100 GB of images) into memory causes system crashes. AI models need clean tool schema definitions and state persistence between reasoning runs. |
| **Where It Is Used** | Model training scripts (PyTorch), LLM pipelines, autonomous coding agents, chatbot systems, and data extraction pipelines. |
| **Real-World Analogy** | **Data Loaders:** A factory assembly line. Instead of dumping a giant pile of raw materials onto the factory floor at once, components are streamed in precisely at the speed the machine consumes them. **Agent Tool Call:** A chef reading a recipe. When the recipe calls for chopping (an agent triggering a tool), the chef pulls out the cutting board (parameters) and runs the task. |

---

## Think of It This Way

### 1. PyTorch DataLoaders: Multiprocessing Generators
When training a deep learning model, the GPU is extremely fast at processing batches. If the CPU is slow at loading and preprocessing data, the GPU sits idle (the "data bottleneck").
A PyTorch `DataLoader` uses Python's `multiprocessing` to pre-load and queue up batches in the background while the GPU trains. 
Under the hood, it calls a custom class that implements `__len__` and `__getitem__` (which acts as an index mapper to retrieve single records on-demand).

### 2. AI Agents: Function Introspection
To allow an LLM to call a Python function, you must present the function to the LLM as a JSON schema. 
Instead of writing this JSON schema by hand for every function, modern agent frameworks use Python's **`inspect`** module. It reads the function's docstring and type hints to dynamically compile the JSON tool schema automatically.

---

## Step 1: Writing a Custom PyTorch-like Dataset Loader

We implement `__len__` and `__getitem__` to stream data from disk dynamically:

```python
# Conceptual PyTorch Dataset interface (without external torch dependency)
class TextDataset:
    def __init__(self, file_paths):
        self.file_paths = file_paths # Only store metadata in memory!

    def __len__(self):
        return len(self.file_paths)

    def __getitem__(self, idx):
        # Only load the file into memory when requested
        path = self.file_paths[idx]
        with open(path, "r") as f:
            data = f.read()
        return self._preprocess(data)

    def _preprocess(self, text):
        # Standard tokenization or normalization
        return text.lower().split()

dataset = TextDataset(["doc1.txt", "doc2.txt"]) # fast startup
# batch = [dataset[i] for i in range(2)]        # streamed dynamically
```

---

## Step 2: Dynamic Tool Schema Compilation for AI Agents

We use the standard `inspect` module to parse a Python function's metadata and generate an LLM-compatible tool definition:

```python
import inspect
from typing import get_type_hints

def get_weather(location: str, unit: str = "celsius") -> str:
    """
    Get the weather forecast for a specific location.
    """
    return f"Weather in {location} is 22 {unit}"

def compile_tool_schema(func):
    sig = inspect.signature(func)
    hints = get_type_hints(func)
    doc = inspect.getdoc(func) or ""

    parameters = {}
    for name, param in sig.parameters.items():
        parameters[name] = {
            "type": str(hints.get(name, "any").__name__),
            "default": None if param.default == inspect.Parameter.empty else param.default
        }

    return {
        "name": func.__name__,
        "description": doc.strip(),
        "parameters": parameters
    }

print(compile_tool_schema(get_weather))
# Output:
# {
#   'name': 'get_weather',
#   'description': 'Get the weather forecast for a specific location.',
#   'parameters': {
#       'location': {'type': 'str', 'default': None},
#       'unit': {'type': 'str', 'default': 'celsius'}
#   }
# }
```

---

## Code Examples

### Easy Examples
```python
# 1. Cleaning GPU Memory context in PyTorch loops
# Useful during validation epochs to prevent CUDA Out Of Memory (OOM)
def validate_model(model, data_loader):
    import torch
    model.eval()
    with torch.no_grad(): # Disables gradient tracking (saves massive memory)
        for inputs in data_loader:
            model(inputs)
    # Clear PyTorch internal cache allocation
    torch.cuda.empty_cache()
```

### Medium Examples
```python
# 2. Simple Agent Loop (Reentrancy with Tool Execution)
class SimpleAgent:
    def __init__(self, tools):
        self.tools = {t.__name__: t for t in tools}

    def execute(self, tool_name, arguments):
        tool = self.tools.get(tool_name)
        if not tool:
            return f"Error: Tool {tool_name} not found"
        try:
            # Dynamically execute function using argument dictionary expansion
            return tool(**arguments)
        except Exception as e:
            return f"Error running tool: {str(e)}"

def calc_sum(a: int, b: int):
    return a + b

agent = SimpleAgent([calc_sum])
print(agent.execute("calc_sum", {"a": 10, "b": 20})) # 30
```

### Advanced Examples
```python
# 3. Thread-Safe Agent State Store
import threading

class AgentSessionStore:
    def __init__(self):
        self._lock = threading.Lock()
        self._sessions = {}

    def get_state(self, session_id):
        with self._lock:
            if session_id not in self._sessions:
                self._sessions[session_id] = {"history": [], "memory": {}}
            return self._sessions[session_id]

    def update_state(self, session_id, history_entry):
        with self._lock:
            state = self._sessions[session_id]
            state["history"].append(history_entry)
```

---

## Common Mistakes & Edge Cases

### 1. PyTorch DataLoader worker leakage
When using multiple workers (`num_workers > 0`) in PyTorch DataLoaders, CPython forks new processes. If your dataset contains unpicklable objects (like active database sessions or open sockets), child worker processes will crash with a `PicklingError` or create duplicate connections.
* **Fix:** Initialize database connections or sockets lazily inside `__getitem__` rather than inside the Dataset `__init__`.

---

## Interview Questions (Top 5)

**Q1: How do PyTorch datasets use Python's dunder methods to enable index-based dynamic streaming?**
> A custom dataset inherits from the `Dataset` class and implements two primary magic methods: `__len__(self)` (which returns the total item count) and `__getitem__(self, index)` (which fetches and returns a single record). 
> The DataLoader uses these methods to retrieve records dynamically. Instead of storing the whole dataset in memory, only the indices are managed. The item is loaded from disk and preprocessed only when its index is requested.

**Q2: What is the purpose of `torch.no_grad()` inside model inference pipelines?**
> By default, PyTorch tracks operations on tensors to construct an autograd computation graph for backward gradient propagation. During inference, we do not update weights. Wrapping code inside the `with torch.no_grad()` context manager disables this tracking, which eliminates the graph allocation overhead and saves massive amounts of GPU memory.

**Q3: How do AI Agent frameworks dynamically register and expose python functions to LLMs?**
> They use runtime reflection via the standard `inspect` module. The framework inspects the function's signature (`inspect.signature`) to read parameters and default values, resolves data types via `typing.get_type_hints()`, and parses the docstring (`inspect.getdoc`) to understand functionality. This metadata is then formatted into a JSON schema block that is passed to the LLM as a tool definition.

**Q4: Why can't you safely pass open file streams or active network sockets as properties of a PyTorch Dataset class when using multiple loaders?**
> When PyTorch's `DataLoader` is configured with `num_workers > 0`, it uses Python's multiprocessing to fork child processes. Python cannot serialize (pickle) open file descriptors or network sockets to send them across process boundaries, which raises a `TypeError` or `PicklingError`. Even if it manages to fork them on Unix, writing/reading to the shared descriptors concurrently will cause data corruption.

**Q5: How do you handle state persistence inside an AI Agent that calls multiple tools in a loop?**
> The agent runtime runs an execution loop. State is managed by wrapping the session history inside a structured, thread-safe memory manager (like a dictionary protected by locks or using an active database transaction store). After each tool executes, the output is formatted as a "Tool Role" message, appended to the session's chat history list, and fed back into the next LLM call iteration.

---

*← [Previous: Chapter 6 (NumPy & Pandas Internals)](06_Numpy_Pandas_Internals.md) | [Next: Chapter 8 (LangChain, RAG & MCP Concepts) →](08_LangChain_RAG_MCP.md)*
