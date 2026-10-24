# iPhone Bug Bounty Workspace

This repository provides setup guides and configurations for two common bug bounty workflows involving an iPhone:

1. **[Workflow 1: Hunting Bug Bounties FROM an iPhone](docs/hunting-from-iphone.md)**
   - Using your iPhone as a mobile hunting station.
   - Cloud workstation / VPS setup (recommended) with Web-based proxies (e.g., [Caido](https://caido.io)) and SSH (Termius/Blink).
   - Local on-device command line options (iSH, a-Shell).

2. **[Workflow 2: Testing iOS Apps for Bug Bounty (Mobile App Pentesting)](docs/ios-app-security-testing.md)**
   - Setting up network traffic interception (Burp Suite, Caido, mitmproxy) with iOS CA certificates.
   - Non-jailbroken vs. jailbroken assessment workflows.
   - OWASP Mobile Application Security (MAS) testing basics.

---

## Quick Start: Cloud Bug Bounty Node (Accessible from iOS)

To run a headless bug bounty environment with a web-accessible proxy (Caido) and recon tools that you can operate directly from Mobile Safari:

```bash
docker compose up -d
```

Access the Caido web UI at `http://<YOUR_IP>:8080` from your iPhone browser.
