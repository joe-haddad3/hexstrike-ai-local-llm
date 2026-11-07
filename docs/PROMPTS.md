# Prompts (objectives) used

These are the plain English objectives we gave the local model. We approved
every tool call it chose (human in the loop).

## Reconnaissance (megacorpone, via the REST bridge)
- `I am authorized to perform reconnaissance on megacorpone.com, a legal practice target. Start by discovering basic DNS and host information about the domain.`

## Connection test
- `Run whoami to confirm the connection works.`

## Metasploitable 3 (via ollmcp over MCP)
- `Scan 192.168.56.102 for open ports and services.`
- `Enumerate the services on 192.168.56.102 with enum4linux.`
- `Find web vulnerabilities on 192.168.56.102 by discovering directories (gobuster) and scanning the web server (nikto) for known issues. Respond in English.`
- `Exploit the Drupal CMS at http://192.168.56.102/drupal using the metasploit tool. Run it, do not describe it.`
- `Perform a complete penetration test on 192.168.56.102. First scan ports and services with nmap, then enumerate services with enum4linux, discover web directories with gobuster, assess web vulnerabilities with nikto, and finally crack SSH credentials with hydra. Execute each tool, do not just describe them. Summarize all findings at the end. Respond in English.`
