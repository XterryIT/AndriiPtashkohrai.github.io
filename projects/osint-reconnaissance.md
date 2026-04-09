---
layout: default
title: "Advanced OSINT & Threat Reconnaissance"
permalink: /projects/osint-reconnaissance/
---

# Advanced OSINT & Threat Reconnaissance: Profiling and Leak Analysis

In the realm of Threat Intelligence and Red Teaming, the reconnaissance phase dictates the success of an operation. This project demonstrates advanced Open-Source Intelligence (OSINT) gathering techniques, combining automated Python scraping, framework-driven profiling, and deep-dive breach analysis to construct a comprehensive target profile.

## 🎯 Project Objectives & Arsenal

* **Objective:** Map the external attack surface of a target organization (e.g., educational institutions like Politechnika Wrocławska) without direct interaction, utilizing passive reconnaissance.
* **Key Techniques:** Web Scraping, Social Media Profiling, Geolocation (Shadow Analysis), and Compromised Credential Analysis.
* **Tool Stack:** Python (`BeautifulSoup`, `requests`), `Recon-ng`, `Maltego`.

## ⚙️ Automated Web Scraping (Python)

To aggregate unstructured data from web sources efficiently, I developed custom Python scripts. Instead of manual enumeration, `BeautifulSoup` was utilized to parse HTML DOM structures, extract metadata, and identify hidden endpoints or exposed sensitive directories.

```python
import requests
from bs4 import BeautifulSoup

def perform_web_scraping(target_url):
    """
    Automated extraction of page metadata and structural elements 
    to identify potential information disclosure.
    """
    try:
        # Utilizing custom headers to avoid basic WAF blocking during recon
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'}
        response = requests.get(target_url, headers=headers, timeout=10)
        
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            
            print(f"[*] Target Title: {soup.title.string}")
            
            # Example: Extracting all external links for relationship mapping
            links = soup.find_all('a', href=True)
            for link in links:
                if "http" in link['href']:
                    print(f"[+] Discovered Endpoint: {link['href']}")
                    
    except requests.exceptions.RequestException as e:
        print(f"[-] Connection Error: {e}")

# Example execution during the recon phase
# perform_web_scraping('[https://target-domain.com](https://target-domain.com)')
```

## 🕵️‍♂️ Framework-Driven Profiling (Recon-ng)

Manual searching is prone to human error. I leveraged **Recon-ng**, a full-featured web reconnaissance framework, to automate identity aggregation. 

Using the `recon/profiles-profiler/profiler` module, I conducted permutation analyses on target usernames (e.g., `PolitechnikaWroclawska`, `politechnika-wroclawska`). The framework automatically queried hundreds of platforms, successfully identifying 21 active profiles across critical infrastructure and developer ecosystems, including **GitHub, GitLab, Docker Hub, and SourceForge**. This mapping is crucial for identifying exposed source code or container registries.

## 🌍 Advanced Techniques & Breach Analysis

Beyond standard enumeration, the investigation employed advanced analytical techniques:

1. **Breach Data Analysis:** Investigated historical data breaches (using specialized databases and leak aggregates) to find compromised credentials associated with the target's domain. Identifying reused passwords provides a direct vector for credential stuffing attacks.
2. **Geolocation via Shadow Analysis (IMINT):** Applied Image Intelligence (IMINT) techniques to determine the exact geographical location of assets or individuals based on the angle, length of shadows, and time-of-day metadata extracted from publicly available photographs.
3. **Relationship Mapping (Maltego):** Utilized **Maltego** to visually map the relationships between discovered email addresses, DNS records, and social media profiles, transforming raw data into actionable intelligence graphs.

## 📉 Tactical Takeaways

* **Automation is Key:** Custom Python tooling drastically reduces the time required for the initial reconnaissance phase compared to manual dorking.
* **Developer Exposure:** The discovery of official accounts on GitHub and Docker Hub highlights the need for organizations to aggressively monitor their public repositories for hardcoded secrets (API keys, AWS credentials).
* **The OSINT Threat:** This project proves that a sophisticated threat actor can map an organization's entire digital footprint, identify key personnel, and locate compromised passwords without ever sending a single packet to the target's internal network.

<div class="back-link-container">
  <a href="/" class="btn-back">← Back to Home</a>
</div>