# Agent Command Center

**Gen Digital NeoClaw Hackathon 2026 | Team: Autono-Mates**

## Overview

Agent Command Center (ACC) is a security-first platform for managing AI agents across Gen Digital's product ecosystem. It provides real-time agent orchestration, prompt injection detection, browsing context analysis, and intelligent product recommendations — all powered by live Claude Sonnet sessions through OpenClaw.

## Problem

As enterprises deploy AI agents at scale, three critical challenges emerge:

1. **Security**: AI agents are vulnerable to prompt injection attacks that can override instructions, exfiltrate data, or manipulate behavior.
2. **Visibility**: Organizations lack centralized monitoring of what their AI agents are doing in real time.
3. **Monetization**: There is no automated way to connect user browsing behavior to relevant product recommendations across Gen Digital's portfolio (Norton, LifeLock, MoneyLion, GoBankingRates).

## Solution

ACC addresses all three through a unified platform:

- **Live AI Agents**: Real Claude Sonnet sessions (via OpenClaw) that perform security analysis and financial research on demand.
- **Prompt Injection Scanner**: Every task is scanned before reaching an agent. Malicious inputs (instruction overrides, role manipulation, imperative injection) are blocked automatically.
- **Browsing Context Analyzer**: Detects user intent from URLs and page content (banking, shopping, tax filing, investing) and maps it to a life stage model. Recommends relevant Gen Digital products with relevance scoring.
- **Browser Extension**: A lightweight Chrome extension that runs on every page, providing real-time security scores and contextual product recommendations without leaving the current site.
- **Dashboard**: A centralized command interface for sending tasks to agents, scanning content, analyzing browsing context, and monitoring activity.

## Architecture

```
Browser Extension (Chrome)
        |
        v
   Nginx (HTTPS / AWS ALB)
        |
        v
   Flask API Server (port 8080)
   ├── /api/agents          - Agent management (status, pause, kill, resume)
   ├── /api/agents/<id>/task - Send tasks to live agents (with injection scan)
   ├── /api/scanner/scan    - Prompt injection detection
   ├── /api/products/analyze - Browsing context analysis + product recommendations
   ├── /api/feed            - Activity log
   └── /api/stats           - Aggregate metrics
        |
        ├── OpenClaw Bridge → Claude Sonnet Sessions (real AI agents)
        ├── Prompt Injection Scanner (pattern + heuristic detection)
        └── Browsing Context Analyzer (18 Gen Digital products, life stage model)
```

## Components

### 1. API Server (`api_server.py`)
Flask application that orchestrates all components. Manages agent lifecycle, routes tasks through the injection scanner before forwarding to real OpenClaw sessions, and serves the dashboard.

### 2. OpenClaw Bridge (`scripts/openclaw_bridge.py`)
Connects the Flask API to OpenClaw's gateway service. Manages real Claude Sonnet sessions, sends tasks via CLI, and parses structured responses including token usage and session metadata.

### 3. Prompt Injection Scanner (`scripts/prompt_injection_scanner.py`)
Multi-layer detection engine that identifies:
- Instruction override attempts ("ignore previous instructions")
- Role manipulation ("you are now DAN")
- System prompt extraction requests
- Imperative command injection
- Encoded/obfuscated payloads

Returns a safety score (0-100), threat level, and detailed breakdown of detected threats.

### 4. Browsing Context Analyzer (`scripts/browsing_context_analyzer.py`)
Analyzes URLs and page content to detect browsing signals (banking, shopping, tax filing, investing, social media, etc.) and maps them to a life stage model (student, young professional, family, homeowner, business owner). Recommends relevant Gen Digital products with relevance scoring based on 18 products across Norton, LifeLock, MoneyLion, and GoBankingRates.

Product catalog includes real pricing scraped from Gen Digital websites:
- Norton AntiVirus Plus (from $29.99/yr)
- Norton 360 with LifeLock (from $2.50/mo)
- Norton Secure VPN (from $29.99/yr)
- MoneyLion Instacash (up to $500)
- MoneyLion Credit Builder Plus
- MoneyLion Personal Loans (up to $100K)
- And 12 more products

### 5. Dashboard (`dashboard.html`)
Single-page application served via Nginx. Features:
- Agent cards with task input and real-time response display
- Prompt injection scanner with threat visualization
- Browsing context analyzer with product recommendations
- Activity log
- Manual refresh control

### 6. Browser Extension (`extension/`)
Chrome Manifest V3 extension that injects on every page. On click, it:
- Sends the current URL and page content to the context analyzer
- Displays agent status
- Shows relevant Gen Digital product recommendations for the current page
- Links to the full dashboard

## Live Agents

Two real Claude Sonnet sessions are active through OpenClaw:

| Agent | Session | Capability |
|-------|---------|------------|
| Norton Security Agent | `761f8dce-...` | Visits URLs via web_fetch, analyzes for phishing/malware, returns security reports |
| MoneyLion Finance Agent | `a72f1972-...` | Searches the web for financial data, compares products, returns structured research |

Both agents use web_fetch and web_search tools to perform real analysis. Response times are 10-30 seconds per task.

## Tech Stack

- **Backend**: Python 3.12, Flask
- **AI**: Claude Sonnet 4.5 via OpenClaw (Amazon Bedrock)
- **Frontend**: Vanilla HTML/CSS/JS (single-file dashboard)
- **Extension**: Chrome Manifest V3
- **Infrastructure**: AWS EC2, Nginx, HTTPS via AWS ALB
- **Domain**: autono-mates-neoclaw.securebrowser.com

## Setup

```bash
# Start the API server
cd ~/.openclaw/workspace/agent-command-center
python3 api_server.py &

# Dashboard is served by Nginx at:
# https://autono-mates-neoclaw.securebrowser.com/dashboard.html

# Install browser extension:
# chrome://extensions -> Developer mode -> Load unpacked -> select extension/
```

```

## File Structure

```
agent-command-center/
  api_server.py                  # Main Flask API
  scripts/
    openclaw_bridge.py           # OpenClaw session management
    prompt_injection_scanner.py  # Injection detection engine
    browsing_context_analyzer.py # Context analysis + product recommendations

/var/www/dashboard/
  dashboard.html                 # Single-page dashboard

extension/
  manifest.json                  # Chrome extension manifest
  content.js                     # Extension logic
  widget.css                     # Extension styles
  icon.png                       # Extension icon
```

## Team

- **Abirami Ravishankar** 
- **Hoshika Gandavaram** 
