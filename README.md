# AWS Strands Decider 2B Local Setup Guide

A step-by-step guide to install, configure, and run AWS Strands Decider 2B locally on Windows using WSL2 and Ubuntu.

---

## Overview

Strands Decider is a lightweight decision model designed for AI agent routing, tool selection, classification, validation, and confidence scoring.

This guide demonstrates how to:

- Set up WSL2 Ubuntu
- Create a Python virtual environment
- Install Strands Decider
- Download and run the Strands Decider 2B model
- Expose a local inference API
- Test the model using Python
- Use it as a routing layer in a RAG or Agentic AI architecture

---

## Prerequisites

### Windows

- Windows 11
- WSL2 enabled
- Ubuntu installed

Verify WSL installation:

```powershell
wsl -l -v
```

Expected output:

```text
NAME      STATE    VERSION
Ubuntu    Running  2
```

Launch Ubuntu:

```powershell
wsl -d Ubuntu
```

---

## Create Project Directory

```bash
mkdir -p /mnt/c/GENAI/strands
cd /mnt/c/GENAI/strands
```

---

## Install System Dependencies

Update Ubuntu packages:

```bash
sudo apt update
```

Install required packages:

```bash
sudo apt install -y \
    python3 \
    python3-pip \
    python3-venv \
    python3-dev \
    build-essential \
    gcc \
    g++
```

Verify installation:

```bash
python3 --version
gcc --version
```

---

## Create Python Virtual Environment

Create a virtual environment:

```bash
python3 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

Upgrade pip and related packages:

```bash
pip install --upgrade pip setuptools wheel
```

---

## Install Strands Decider

Install the package:

```bash
pip install strands-decider
```

Verify:

```bash
strands-decider --help
```

---

## Discover Available Models

Install Hugging Face Hub:

```bash
pip install huggingface_hub
```

List available Strands models:

```bash
python -c "from huggingface_hub import list_models; [print(m.id) for m in list_models(search='strands')]"
```

Official model used in this guide:

```text
StrandsAgents/strands-decider-2B-hobson-v19
```

---

## Start the Local Server

Run the model server:

```bash
strands-decider serve StrandsAgents/strands-decider-2B-hobson-v19 --device cpu
```

Expected startup output:

```text
Application startup complete.
Uvicorn running on http://127.0.0.1:8000
```

The first startup downloads and caches model weights automatically.

---

## Verify Server Availability

Open:

```text
http://127.0.0.1:8000/docs
```

Or test using curl:

```bash
curl http://127.0.0.1:8000/openapi.json
```

---

## Supported Question Types

### Choice Question

Used to select one option from many.

```json
{
  "type": "choice",
  "instructions": "Select the best datasource.",
  "criteria": {
    "PLM": "Engineering changes and parts",
    "JIRA": "Issue tracking system",
    "CONFLUENCE": "Documentation repository",
    "UNKNOWN": "No suitable source"
  }
}
```

---

### Noul Question

Used for Yes/No decisions.

```json
{
  "type": "noul",
  "instructions": "Determine whether the statement is true."
}
```

---

### Score Question

Used to rate against an ordered scale.

```json
{
  "type": "score",
  "instructions": "Rate the sentiment.",
  "criteria": [
    "Very Negative",
    "Negative",
    "Neutral",
    "Positive",
    "Very Positive"
  ]
}
```

---

## Python Test Example

Create a file named:

```text
decider_demo.py
```

### Sample Code

```python
import requests
import json

payload = {
    "state": "User wants ECO information",
    "questions": {
        "datasource": {
            "type": "choice",
            "instructions": "Select the most appropriate datasource.",
            "criteria": {
                "PLM": "Engineering changes and parts",
                "JIRA": "Issue tracking system",
                "CONFLUENCE": "Documentation repository",
                "UNKNOWN": "No suitable source"
            }
        }
    }
}

response = requests.post(
    "http://127.0.0.1:8000/v1/systemone",
    json=payload
)

print("Status:", response.status_code)
print(json.dumps(response.json(), indent=2))
```

Run:

```bash
python decider_demo.py
```

---

## Sample Response

```json
{
  "answers": {
    "datasource": {
      "choice": "PLM",
      "confidence": 0.94
    }
  }
}
```

---

## Using Strands Decider as a RAG Router

Example:

```python
decision = response["answers"]["datasource"]["choice"]

if decision == "PLM":
    plm_rag.search(query)

elif decision == "JIRA":
    jira_rag.search(query)

elif decision == "CONFLUENCE":
    confluence_rag.search(query)

else:
    print("No reliable datasource identified.")
```

---

## Example Architecture

```text
User Query
    |
    v
Strands Decider
    |
    +---------------------+
    |                     |
    v                     v
 PLM RAG              JIRA RAG
    |                     |
    +---------+-----------+
              |
              v
        LLM Response
```

---

## Use Cases

- Multi-RAG routing
- Tool selection for AI agents
- Intent classification
- Confidence-based validation
- Retrieval quality assessment
- Agent workflow orchestration
- Hallucination reduction

---

## Benefits

✅ Low latency inference

✅ Local execution

✅ Confidence scoring

✅ Deterministic decision making

✅ Ideal for agent routing and orchestration

✅ Works well with RAG and MCP architectures

---

## Author

**Ujjwalkumar Soni**  


---

## License

This tutorial is provided for educational and research purposes. Refer to the Strands Decider and associated model licenses for production usage requirements.
