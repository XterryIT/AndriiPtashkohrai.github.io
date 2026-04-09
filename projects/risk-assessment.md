---
layout: default
title: "Business-Centric Cloud Risk Assessment"
permalink: /projects/risk-assessment/
---

# Business-Centric Cloud Risk Assessment & Disaster Recovery

In enterprise environments, cybersecurity is fundamentally about risk management. This project demonstrates the ability to translate technical vulnerabilities into quantifiable business impacts. I conducted a comprehensive threat modeling and risk assessment for an e-commerce application deployed on **Google Cloud Platform (GCP)**, culminating in the design of robust Disaster Recovery (DRP) and Business Continuity (BCP) plans.

## 🏗️ Cloud Architecture & Threat Landscape

The assessed infrastructure was a modern, containerized e-commerce platform:
* **Compute:** Kubernetes (GKE) running Django/Gunicorn containers.
* **Database:** PostgreSQL accessed securely via Cloud SQL Proxy.
* **Network & Security:** VPC peering, Ingress NGINX with Let's Encrypt TLS, and **Google Cloud Armor** for WAF/DDoS protection.

The primary threat landscape included OWASP Top 10 vulnerabilities (XSS, SQLi), volumetric DDoS attacks, cloud misconfigurations, and catastrophic regional outages.

## 📊 Risk Analysis & Mitigation Strategy

Instead of merely listing vulnerabilities, risks were evaluated based on their probability and potential financial impact on the e-commerce operations. 

### 1. Infrastructure Security & Network Isolation
* **Identified Risk:** Direct database exposure and lateral movement within the cloud environment.
* **Mitigation Implemented:** Enforced strict VPC firewall rules allowing only HTTP/HTTPS traffic to the load balancer. The Cloud SQL database has **no public IP** and is exclusively accessible via the Cloud SQL Proxy from authenticated backend instances.

### 2. Application Security (AppSec)
* **Identified Risk:** Client-side attacks (XSS, CSRF) leading to customer session hijacking and data breaches.
* **Mitigation Implemented:** Enforcement of strict Content Security Policies (CSP), regular container image scanning in GKE, and mandatory Multi-Factor Authentication (MFA) for administrative API access.

### 3. Availability & DDoS Protection
* **Identified Risk:** Application downtime resulting in immediate revenue loss during volumetric or application-layer DDoS attacks.
* **Mitigation Implemented:** Integration of Google Cloud Armor at the edge to filter malicious traffic, apply geoblocking, and rate-limit suspicious IP ranges.

## 🔄 Disaster Recovery (DRP) & Business Continuity (BCP)

A security architecture is incomplete without a resilient recovery strategy. I defined critical business metrics and actionable recovery procedures:

* **RTO (Recovery Time Objective):** <= 24 hours. The maximum acceptable downtime before the business suffers irreversible reputational and financial damage.
* **RPO (Recovery Point Objective):** <= 3 hours. Ensured by automated, incremental database backups running every 6 hours and transaction log archiving.
* **Failover Strategy:** In the event of a total GCP region failure, the DRP mandates automated DNS failover to a secondary standby region.
* **Testing:** Established a schedule for monthly "Tabletop Drills" to test incident communication and backup restoration procedures.

## 📉 Conclusions & Business Value

This assessment bridges the gap between DevOps and executive stakeholders. By implementing Cloud Armor, VPC isolation, and strict DRP metrics, the organization not only secures its infrastructure against advanced threats but also ensures regulatory compliance and guarantees business continuity during critical incidents.

<div class="back-link-container">
  <a href="/" class="btn-back">← Back to Home</a>
</div>