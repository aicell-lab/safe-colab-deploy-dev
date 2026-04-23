---
name: safe-colab
description: Interact securely with published datasets in Safe Colab to read documentation and run Python analysis code.
license: MIT
metadata:
  version: 1.0.0
---

# Safe Colab Dataset Interaction Skill

This document provides instructions for an AI agent on how to interact with Safe Colab published datasets to perform secure data analysis. 

Safe Colab uses the Hypha platform to expose services over HTTP. You will be provided with a **workspace** and a **Service ID** representing a mounted dataset. You may also be provided with an **access key** for callers that are not already authenticated as a shared user by email or Hypha ID.

## General Interaction

The dataset is exposed as a Hypha service. 
To call a function on the dataset service via HTTP, you can use `curl`. The endpoint URL follows the format:
`https://hypha.aicell.io/<workspace>/services/<service_id>/<function_name>`

**DO:**
- Provide arguments via JSON body when using POST requests.
- Use `curl -L` to follow redirects. 

**DON'T:**
- Do not add a trailing slash `/` to the function endpoint. It will cause Method Not Allowed errors.
- Do not attempt to mount datasets yourself unless explicitly asked. The user will provide the service ID.
- Do not output raw credentials or tokens.

## Available Functions

A mounted Safe Colab dataset exposes the following functions:

### 1. `get_docs(access_key?)`
Retrieves the documentation/metadata associated with the dataset (usually describing the table schemas, columns, missing values, etc.).

**Usage:**
```bash
curl -L -X POST "https://hypha.aicell.io/<workspace>/services/<service_id>/get_docs" \
     -H "Content-Type: application/json" \
     -d '{"access_key": "<access_key>"}'
```

### 2. `run_python(code, access_key?)`
Executes arbitrary Python code securely inside the dataset's sandbox environment.
The dataset is typically mounted at `/data` (e.g., `/data/data.csv`), but you should always check `get_docs()` to confirm the exact path.

You will receive the result directly after the code executes.

**Usage:**
```bash
curl -s --max-time 120 -L -X POST "https://hypha.aicell.io/<workspace>/services/<service_id>/run_python" \
     -H "Content-Type: application/json" \
     -d '{"code": "import pandas as pd\ndf = pd.read_csv(\"/data/data.csv\")\nprint(df.head())", "access_key": "<access_key>"}'
```

*(Note the `\n` and `\"` escaping inside the JSON body)*

## Example Workflow

1. **Read Dataset Documentation:**
    Start by reading the documentation to understand which files are available and their schema:
   ```bash
   curl -L -X POST "https://hypha.aicell.io/<workspace>/services/<service_id>/get_docs" \
        -H "Content-Type: application/json" \
        -d '{"access_key": "<access_key>"}'
   ```

2. **Run Analysis Code:**
   Write code to analyze the data. For example, to read the first few rows of the dataset:
   ```bash
   curl -s --max-time 120 -L -X POST "https://hypha.aicell.io/<workspace>/services/<service_id>/run_python" \
        -H "Content-Type: application/json" \
        -d '{"code": "import pandas as pd\ndf = pd.read_csv(\"/data/data.csv\")\nprint(df.head())", "access_key": "<access_key>"}'
   ```
   *Note: Ensure proper escaping of quotes (`\"`) and newlines (`\n`) in your JSON payload.*

## Authorization
If you were given an access key, include it in every `get_docs` and `run_python` JSON body as `"access_key": "<access_key>"`. If you are authenticated as a shared user by email or Hypha ID, the key may be optional. If a bearer token is required or provided by the user, add the `-H "Authorization: Bearer <token>"` header to your API requests as well.

**Important**: 
When running python, always print the output to standard out using `print()`, otherwise you might receive an empty output. Or simply rely on the last expression output returned.
