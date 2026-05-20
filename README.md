# NetBot 🌐🤖

NetBot is an intelligent, AI-driven network diagnostic assistant. Powered by local LLMs (via Ollama), it translates natural language queries into automated network troubleshooting workflows. The agent autonomously selects the right networking tools, executes them, and translates the raw technical output into plain-English diagnoses with actionable fixes.

## 🚀 Key Features

* **LLM-Driven Diagnostics:** Uses Llama3 (via Ollama) to parse user issues, trigger tools, and analyze results.
* **Autonomous Tool Dispatcher:** Built-in Python functions to run and parse real system commands:
  * `PING` (Reachability, latency, packet loss)
  * `NSLOOKUP` (DNS resolution)
  * `TRACEROUTE` (Hop-by-hop path tracing)
  * `PORTSCAN` (TCP port connectivity)
  * `HTTPCHECK` (HTTP/HTTPS status codes and SSL errors)
  * `IFCONFIG` (Local network interfaces and IP allocation)
* **Dual Interfaces:**
  * **Web UI (`app.py`):** A Flask-based server utilizing Server-Sent Events (SSE) for real-time streaming of the agent's thought process and tool execution.
  * **CLI Mode (`netbot_final.py` / `netcla.py`):** A lightweight terminal interface for quick, command-line diagnostics.
* **Context-Aware Reasoning:** The agent follows strict diagnostic workflows (e.g., verifying global internet connectivity via `8.8.8.8` before diagnosing specific domain failures) to prevent hallucinated conclusions.

## 🛠️ Tech Stack

* **Backend:** Python, Flask
* **AI/LLM:** Ollama (default: `llama3`)
* **Networking:** Native OS sub-processes (`ping`, `nslookup`, `tracert`/`traceroute`), Python `socket`, `urllib`
* **Frontend:** HTML/JS with Server-Sent Events (SSE) for the web interface

## ⚙️ Getting Started

### Prerequisites
1. Install [Python 3.8+](https://www.python.org/downloads/)
2. Install [Ollama](https://ollama.com/) and pull the Llama3 model:
   ```bash
   ollama run llama3
