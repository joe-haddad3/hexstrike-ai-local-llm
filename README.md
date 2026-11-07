# HexStrike AI + Local LLM

An ethical hacking lab project for the Master in Cybersecurity at Saint Joseph
University. The goal was to install the HexStrike AI orchestrator, run a local
language model next to it so that nothing leaves the machine, connect the two
together, and then use the setup to test a lab network.

HexStrike AI is an MCP server that lets an AI agent drive more than 150 security
tools (nmap, nikto, sqlmap, gobuster, hydra, metasploit and others) instead of
running each tool by hand. In this project the "brain" is a local model served
by Ollama, so there is no cloud involved.

> **For authorized lab use only.** Everything here was run against a public
> practice target (megacorpone, made public by OffSec for exactly this) and a
> Metasploitable 3 machine that we own, on an isolated host only network. Never
> point these tools at any system you are not explicitly authorized to test.

## Setup

- **Host:** Windows 11 (Intel i7-13700HX, 24 GB RAM, RTX 5050 laptop GPU, 8 GB VRAM)
- **Local LLM runtime:** Ollama, serving `qwen2.5:14b` and a custom model `pentest-assistant`
- **Attacker VM:** Kali Linux in VirtualBox, running HexStrike AI
- **Target VM:** Metasploitable 3 (Ubuntu) on a host only network at `192.168.56.102`
- **Recon target:** megacorpone.com (public practice target)

```
  Windows host                         Kali VM
  +----------------------+             +-----------------------------+
  |  Ollama (local LLM)  |  network    |  ollmcp / bridge            |
  |  qwen2.5:14b         |<----------->|  hexstrike_mcp (MCP)        |
  |  pentest-assistant   |             |  hexstrike_server (:8888)   |
  +----------------------+             |  150+ security tools        |
                                       +--------------+--------------+
                                                      |
                                                      v
                                        Targets: megacorpone / Metasploitable
```

## What is in this repo

| Path | What it is |
|------|------------|
| `Modelfile` | The custom `pentest-assistant` model config (temperature, context, system role) |
| `bridge/ai_pentest_bridge.py` | Method 1: a small REST bridge that lets the local model drive HexStrike's API |
| `patches/mcp_v2_fastmcp_shim.md` | The patch that made HexStrike work with the mcp v2 library so ollmcp could connect |
| `docs/PROMPTS.md` | The plain English objectives given to the model |
| `docs/HexStrike_Report.pdf` / `.docx` | The full project report |

## How to reproduce

### 1. Install HexStrike (in Kali)
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install hexstrike-ai
hexstrike_server          # starts the API server on 127.0.0.1:8888
```

### 2. Local model (on Windows, with Ollama)
```bash
ollama pull qwen2.5:14b
ollama create pentest-assistant -f Modelfile
```
Make Ollama reachable from the VM by setting `OLLAMA_HOST=0.0.0.0:11434` and
restarting it.

### 3a. Method 1, the REST bridge
```bash
python3 bridge/ai_pentest_bridge.py
```
Give it an objective, approve each command, and it drives HexStrike's REST API.

### 3b. Method 2, the MCP path (ollmcp)
Apply the patch in `patches/mcp_v2_fastmcp_shim.md`, then:
```bash
pip install mcp-client-for-ollama
ollmcp -s /usr/share/hexstrike-ai/hexstrike_mcp.py \
       --model qwen2.5:14b \
       --host http://192.168.56.1:11434
```
Inside ollmcp, use `/tools` to enable only the tools you need (we used nmap,
enum4linux, gobuster, nikto, metasploit and hydra). Keeping the tool list short
is what made the local model reliable at firing tool calls.

## Results (Metasploitable 3)

- **Scanning (nmap):** 9 open services including FTP, SSH, HTTP (Apache), SMB, MySQL, IRC (UnrealIRCd), WEBrick and Jetty
- **Enumeration (enum4linux):** empty SMB session allowed, user `chewbacca`, shares `print$`, `public`, `IPC$`
- **Web discovery (gobuster):** a Drupal site at `/drupal` and phpMyAdmin at `/phpmyadmin`
- **Vulnerability testing (nikto):** outdated Apache and PHP, directory indexing, missing security headers, dangerous HTTP methods enabled
- **Exploitation:** the Drupal `drupageddon2` Metasploit module run against `/drupal`, and **hydra cracked the SSH login (`vagrant` / `vagrant`)**
- **Complete test:** the model chained nmap, enum4linux, gobuster, nikto and hydra on its own, approving each step through the human in the loop gate

## Key takeaways

- A local LLM orchestrator is doable and keeps everything confidential.
- The REST bridge is simple and reliable because the model only prints a command.
- The MCP path is the intended design but a local 14B model struggles with 150 tools. Cutting the tool list down to the five we needed fixed that and the model then ran a full, successful penetration test.

## Authors

- Joe Haddad
- Georges EL Hajj

