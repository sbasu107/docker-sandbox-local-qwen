# Qwen Sandbox Environment - Setup & Dependency Notes

We ran into several bleeding-edge compatibility issues due to the Sandbox using a very recent version of Python (3.14). Here is exactly how we resolved them:

### 1. Starting the API
Instead of the chat interface, we ran `llama.cpp` as an OpenAI-compatible API server in the background:
```bash
llama-server -m ./qwen2.5-3b.gguf --port 8080 &
```

### 2. TikToken Compilation (Rust Missing)
`tiktoken` lacked pre-built Python 3.14 wheels, triggering a source build that failed without a Rust compiler.
**Fix:**
```bash
sudo apt update && sudo apt install -y rustc cargo
```

### 3. PyO3 Python 3.14 Restriction
The `pyo3` library inside `tiktoken` officially blocked compilation on Python versions > 3.12.
**Fix:** We bypassed the version check by forcing the stable ABI:
```bash
PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1 pip install open-interpreter
```

### 4. Missing `pkg_resources`
Python 3.14 / setuptools v84+ officially removed the deprecated `pkg_resources` module, which crashed `open-interpreter` on startup.
**Fix:** We downgraded to an older `setuptools` build:
```bash
pip install "setuptools<70.0.0"
```

### Launch Command
```bash
interpreter --api_base "http://localhost:8080/v1" --api_key "local" --model "openai/qwen"
```
