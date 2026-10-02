# Local AI Workstation Stack

A robust, GPU-accelerated local AI workspace running **Ollama** and **Open WebUI** containerized with Docker on Ubuntu Server. Optimized for high-performance coding models and heavy-context LLM tasks.

---

## 🖥️ Hardware & Environment Specifications

- **Workstation:** HP Z6 G5 Workstation
- **OS:** Ubuntu Server 24.04 LTS
- **GPU:** NVIDIA RTX 6000 (24 GB VRAM)
- **RAM:** 256 GB System Memory
- **Core Stack:** Docker Engine, NVIDIA Container Toolkit, Ollama, Open WebUI
- **Primary Model:** `qwen2.5-coder:32b`

---

## 🚀 Quick Start

### 1. Prerequisites

Ensure you have the latest NVIDIA drivers and the **NVIDIA Container Toolkit** installed and verified:

```bash
nvidia-smi
sudo docker run --rm --gpus all nvidia/cuda:12.0.0-base-ubuntu22.04 nvidia-smi
```

### 2. Repository Setup

Clone this repository and create your directory structure:

```bash
git clone https://github.com/YOUR_USERNAME/local-ai-stack.git
cd local-ai-stack
```

### 3. Deploy Stack

Launch the services using Docker Compose:

```bash
sudo docker compose up -d
```

### 4. Download Recommended Model

Pull **Qwen 2.5 Coder 32B** directly into the running Ollama container:

```bash
sudo docker exec -it ollama ollama pull qwen2.5-coder:32b
```

Access Open WebUI at `http://<YOUR_SERVER_IP>:3000`.

---

## 🛠️ Docker Compose Configuration (`docker-compose.yml`)

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ./ollama_data:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    ports:
      - "3000:8080"
    volumes:
      - ./open-webui_data:/app/backend/data
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - ENABLE_PERSISTENT_CONFIG=False
      - ENABLE_FUNCTION_CALLING=False
      - ENABLE_OLLAMA_TOOL_CALLING=False
      - DEFAULT_MODELS=qwen2.5-coder:32b
    depends_on:
      - ollama
```

---

## ⚠️️ Important Troubleshooting: Qwen 2.5 Coder Tool Calling Issue

### Issue
When asking basic questions (e.g., `"hi"`), `qwen2.5-coder:32b` may output raw JSON structured calls like `{"name": "ask_user", ...}` instead of plain text responses.

### Cause
`qwen2.5-coder` natively supports function calling. Open WebUI injects built-in system tools (such as `ask_user`) into prompts by default. Qwen sees these tool definitions and assumes it must execute a function call instead of responding in normal conversational text.

### Solution
You **must** disable built-in tools for the model inside Open WebUI:

1. Open Open WebUI (`http://<YOUR_SERVER_IP>:3000`).
2. Go to **Admin Panel** / **Workspace** $\rightarrow$ **Models** $\rightarrow$ Click the **Edit (Pencil)** icon next to `qwen2.5-coder:32b`.
3. Scroll to the **Capabilities** section.
4. Locate **Built-in Tools** (displayed as `settings.admin.models.capabilities.builtinTools.label` if interface translation keys are active).
5. **Uncheck / Deactivate** `builtinTools` (Built-in Tools).
6. Click **Save** at the bottom of the page.
7. Open a **New Chat** session.

---

## 📁 Repository Structure & `.gitignore`

Ensure model data and user database contents are excluded from Git commits:

```text
.gitignore
├── ollama_data/        # Excluded: Model weights (20 GB+)
├── open-webui_data/    # Excluded: User chats and database
├── docker-compose.yml
└── README.md
```
