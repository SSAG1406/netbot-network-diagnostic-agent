# NetBot — LLM-Powered Network Diagnostic Agent

NetBot is an agentic AI assistant that translates plain-English network complaints into autonomous, tool-driven diagnostics. You type *"I can't reach GitHub"* — NetBot decides which tools to run, executes them, interprets the raw output, and tells you exactly what's wrong and how to fix it.

Built with LLaMA 3 (via Ollama) and a structured reasoning loop that prevents hallucination by enforcing strict tool-before-summary workflows.

---

## Demo

<img width="1777" height="1271" alt="image" src="https://github.com/user-attachments/assets/14141760-18a1-42f5-8e62-78887bc52a77" />


---

## How it works

Most LLM "agents" fake tool results or guess. NetBot doesn't. The system prompt enforces a strict contract:

1. LLaMA 3 reads the user's complaint and decides which tool to call first
2. It outputs **only** the tool call — no guesses, no premature summaries
3. NetBot executes the real OS command and feeds the actual output back to the model
4. The model either calls another tool or produces a final diagnosis based on real data
5. This loop runs up to 6 tool calls before forcing a conclusion

The agent also enforces a **connectivity-first rule**: if any tool returns a network failure signal (`NO_INTERNET`, `DNS_FAIL`, `ROUTE_UNREACHABLE`), it immediately verifies global connectivity by pinging `8.8.8.8` before diagnosing anything else. This prevents misdiagnosis — e.g. blaming a website when the user's router is down.

---

## Diagnostic tools

| Tool | What it does |
|------|-------------|
| `PING <host>` | Checks reachability, latency, and packet loss. Tags `NO_INTERNET_CONFIRMED` if `8.8.8.8` fails. |
| `NSLOOKUP <host>` | DNS resolution. Returns `NXDOMAIN` if domain doesn't exist, `DNS_TIMEOUT` if server unreachable. |
| `TRACEROUTE <host>` | Hop-by-hop path trace (20 hops max). Detects blocked ICMP, routing failures, and partial paths. |
| `PORTSCAN <host> <port>` | TCP port connectivity check. Distinguishes `OPEN`, `REFUSED`, and `CLOSED/FILTERED`. |
| `HTTPCHECK <url>` | HTTP/HTTPS status check. Detects `SERVER_TIMEOUT`, `DNS_FAIL`, `SSL_ERROR`, `CONN_REFUSED`. |
| `IFCONFIG` | Local interface inspection. Detects `NO_IP` and `APIPA` (self-assigned 169.254.x.x = DHCP fail). |

---

## Interfaces

### Web UI (`app.py`)
Flask server with **Server-Sent Events (SSE)** streaming — the agent's reasoning steps and tool invocations appear in real time as they happen, not after. Runs at `http://localhost:5000`.

### CLI (`netbot_final.py`)
Terminal interface with a `NetBot` class. Clean REPL loop — type your issue, get a diagnosis. Supports custom Ollama model selection at startup.

---

## Quick start

### Prerequisites

- Python 3.8+
- [Ollama](https://ollama.com/) installed and running

```bash
ollama pull llama3
```

### Install

```bash
git clone https://github.com/SSAG1406/NetBot---LLM-Powered-Network-Diagnostic-Agent.git
cd NetBot---LLM-Powered-Network-Diagnostic-Agent
pip install flask ollama
```

### Run the web UI

```bash
python app.py
```
Open `http://localhost:5000` in your browser.

### Run the CLI

```bash
python netbot_final.py
```

---

## Example session

```
You: I can't open youtube.com but my wifi is connected

[TOOL] HTTPCHECK https://youtube.com
→ DNS_FAIL: DNS resolution failed.

[TOOL] PING 8.8.8.8
→ OK: All packets received. Host is reachable.

[TOOL] NSLOOKUP youtube.com
→ NXDOMAIN: 'youtube.com' does not exist in DNS.

NetBot: Your internet connection is working fine (Google's DNS responds).
The problem is your DNS resolver can't find youtube.com — this usually means
your DNS cache is corrupted or your ISP's DNS is filtering the domain.

Fix: Run `ipconfig /flushdns` (Windows) or `sudo dscacheutil -flushcache` (Mac),
then try switching your DNS to 8.8.8.8 or 1.1.1.1 in your network settings.
```

---

## Project structure

```
├── app.py              # Flask web server + SSE streaming agent loop
├── netbot_final.py     # CLI agent (NetBot class + REPL runner)
├── templates/
│   └── index.html      # Web UI frontend
└── README.md
```

---

## Design decisions worth noting

**No hallucination by design.** The system prompt forbids the model from writing anything after a tool call until real output is returned. This is enforced at the prompt level, not post-hoc.

**Context window management.** `app.py` trims history to the last 6 messages to prevent the LLM from losing its diagnostic thread on long sessions.

**Cross-platform.** Tool commands adapt automatically — `ping -n` on Windows, `ping -c` on Linux/Mac; `tracert` vs `traceroute`; `ipconfig` vs `ip addr`.

**APIPA detection.** If `IFCONFIG` returns a `169.254.x.x` address, NetBot immediately flags it as a DHCP failure rather than treating it as a valid connection — a common false positive other tools miss.

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| LLM | LLaMA 3 via [Ollama](https://ollama.com/) |
| Backend | Python, Flask |
| Streaming | Server-Sent Events (SSE) |
| Networking | Native OS subprocesses, Python `socket`, `urllib` |
| Frontend | HTML / JavaScript |

---

## Authors

**Siva Sai Anirudh Gundu**  
ML Research Intern, IIIT Hyderabad (LTRC) · ECE + AI/ML, VNRVJIET  
[linkedin.com/in/ssag](https://linkedin.com/in/ssag) · [github.com/SSAG1406](https://github.com/SSAG1406)

**Viswaja Kandimalla**  
ECE + AI/ML, VNRVJIET  
[linkedin.com/in/ssag](https://www.linkedin.com/in/viswaja-kandimalla/) · [github.com/SSAG1406](https://github.com/viswaja14)
