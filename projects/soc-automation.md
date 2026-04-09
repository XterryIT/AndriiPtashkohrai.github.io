---
layout: default
title: "AI-Powered SOC Automation"
permalink: /projects/soc-automation/
---

# AI-Powered SOC Automation: Integrating Wazuh SIEM with Local LLMs

In modern Security Operations Centers (SOC), alert fatigue is a critical issue. To address this, I designed and implemented a custom SOAR-like architecture that integrates **Wazuh SIEM** with a **local Large Language Model (Ollama)** to automatically analyze web attacks in real-time without sending sensitive data to external APIs.

## 🏗️ Architecture & Infrastructure

The lab environment was built using a multi-node setup connected via a ZeroTier VPN:
* **Honeypot Target:** A vulnerable Django web application running in Docker (Ubuntu VM).
* **SIEM Manager:** Wazuh Server (Ubuntu Server VM).
* **AI Engine:** Mac mini M4 running Ollama (testing `qwen2.5:14b` and `llama3.1:8b` models).
* **Attacker:** Kali Linux instance.

## ⚙️ The Integration Engine

The core of the project is a Python script that acts as an integration hook within Wazuh. It intercepts high-priority alerts, strips unnecessary ANSI escape codes, and queries the local AI model for an incident summary, threat level, and remediation steps.

```python
#!/usr/bin/env python3
import sys
import json
import requests
import datetime
import re

# --- CONFIGURATION ---
AI_URL = "[http://192.168.100.39:11434/api/generate](http://192.168.100.39:11434/api/generate)"
AI_MODEL = "qwen2.5:14b"
OUTPUT_FILE = "/var/ossec/logs/ai_responses.json"

def clean_log(text):
    # Remove ANSI escape sequences that confuse the LLM
    ansi_escape = re.compile(r'\x1B(?:[@-Z\\-_]|\[[0-?]*[ -/]*[@-~])')
    return ansi_escape.sub('', text)

try:
    # 1. Read the alert file provided by Wazuh
    alert_file = sys.argv[1]
    with open(alert_file) as f:
        alert = json.load(f)

    rule_desc = alert.get('rule', {}).get('description', 'N/A')
    full_log = alert.get('full_log', 'N/A')
    clean_full_log = clean_log(full_log)

    # 2. Prepare the prompt for the local LLM
    prompt = f"""
    You are a cybersecurity expert. Analyze the event.
    Event: {rule_desc}
    Log: {clean_full_log}
    Provide:
    1. Attack description
    2. Threat Level (0-10)
    3. Recommendations
    """
    
    payload = {
        "model": AI_MODEL,
        "prompt": prompt,
        "stream": False
    }

    # 3. Send request (Timeout extended to 120s for local processing)
    response = requests.post(AI_URL, json=payload, timeout=120)
    answer = response.json().get("response", "No response from AI")

    # 4. Save the result back for the Wazuh Dashboard
    wazuh_event = {
        "timestamp": datetime.datetime.now().isoformat(),
        "ai_analysis": True,
        "analysis_result": answer
    }

    with open(OUTPUT_FILE, "a", encoding='utf-8') as f:
        json.dump(wazuh_event, f, ensure_ascii=False)
        f.write("\n")

except Exception as e:
    pass
```

## 🛡️ Custom Threat Detection Rules

Standard Wazuh rules lack specific signatures for custom Django applications. I authored custom XML rules (`local_rules.xml`) using PCRE2 regular expressions and HEX encoding to bypass XML syntax limitations (e.g., using `\x3C` for `<`).

```xml
<rule id="100008" level="12">
  <if_sid>100005</if_sid>
  <regex type="pcre2">(?i)(%3Cscript%3E|\x3Cscript|script\x3E|src=javascript:|javascript:|on(load|error|click|mouseover|focus|submit)=|\x3Ciframe|\x3Cobject|\x3Cembed|\x3Csvg|\x3Cimg.*?onerror)</regex>
  <description>Django Attack: Advanced XSS Attempt</description>
</rule>
```

## 📉 Challenges & Lessons Learned

While the system successfully achieved a 100% detection rate on simulated SQLi, XSS, and RCE attacks, the integration phase revealed several critical engineering challenges:

1. **The "AI Loop" Bug:** Initially, the system created an infinite loop. Wazuh triggers scripts for alerts level 10+, and the AI's response was also being ingested as a level 10 alert, causing the AI to analyze its own responses. *Solution: Implemented strict filtering rules to exclude AI-generated logs from triggering the integration script.*
2. **Processing Latency under Load:** During an automated DirBuster scan generating over 14,000 requests, the AI processing queue became a massive bottleneck. While XSS alerts were processed in 30 minutes, SQLi alerts faced a latency of up to 2 hours. *Takeaway: Local LLMs require robust event deduplication and rate-limiting before the prompt generation stage to remain viable in high-traffic environments.*

<div class="back-link-container">
  <a href="/" class="btn-back">← Back to Home</a>
</div>