# SOC IP Enrichment & Automated Alerting Pipeline
A full end‑to‑end security automation workflow built with StackStorm, Python, and VirusTotal
Automates the entire alert‑triage process by extracting IP addresses from raw alerts, enriching them with real‑time threat intelligence, and delivering structured HTML notifications directly to the SOC team.
This project demonstrates practical security automation engineering: custom pack development, API integration, workflow orchestration, and automated alert delivery..

## Overview
This project demonstrates how to build a real SOC automation workflow using StackStorm:

- Extract an IP address from an alert
- Enrich it using the VirusTotal API
- Generate a structured enrichment object
- Send an HTML email alert to the SOC team
- Handle errors, transitions, and context publishing cleanly

This is a production‑grade automation pipeline suitable for SOC Level 1/2 workflows.

## Project Structure
<img width="375" height="378" alt="ProjectStructure" src="https://github.com/user-attachments/assets/3ae74f03-9778-46bc-bd70-167a0e484d54" />

### 1. Created a Custom StackStorm Pack (my_pack)
- Initialized a new pack directory under /opt/stackstorm/packs/
- Added required metadata (pack.yaml)
- Created the folder structure for actions, workflows, and configs
  
### 2. Developed a Python Action for IP Extraction & VirusTotal Enrichment
- Wrote enrich_ip.py to:
- Parse an alert string
- Extract an IP address
- Query VirusTotal using the API key
- Return structured JSON enrichment data
- Registered the action and validated execution

 This action retuns
 
 <img width="360" height="261" alt="return" src="https://github.com/user-attachments/assets/3da8792a-57e3-46d0-888c-259aec1b0495" />


### 3. Built the Workflow (enrich_ip_flow.yaml)
- Designed a multi‑step workflow that:
- Accepts alert + API key
- Runs the enrichment action
- Publishes results into workflow context
- Sends an HTML email notification
- Implemented Jinja templating for dynamic fields

### 4. Installed & Configured the Email Pack
- Installed the email pack > /opt/stackstorm/configs/email.yaml

- Created a valid email.yaml with:

  - smtp_accounts

  - imap_accounts

  - Gmail App Password

- TLS settings

- Fixed schema validation errors

- Corrected indentation (chown st2:st2, chmod 600), permissions, and line endings

Successfully registered the config

Final working config and valided email with st2 run email.send_email ...
<img width="557" height="395" alt="Email" src="https://github.com/user-attachments/assets/f18606de-7bba-4feb-b708-3a57d9ad2b3a" />

### 5. Built the Workflow (enrich_ip_flow)
My workflow:

- Accepts an alert + API key
- Extracts and enriches the IP
- Publishes enrichment results
- Sends an HTML email

Final working workflow:

#########yaml ###############
version: 1.0

description: >
  SOC workflow that extracts an IP from an alert and enriches it using VirusTotal.

input:
  - alert
  - vt_api_key

vars:
  - enrichment: null

tasks:
  extract_and_enrich:
    action: my_pack.enrich_ip
    input:
      alert: <% ctx().alert %>
      vt_api_key: <% ctx().vt_api_key %>
    next:
      - when: <% succeeded() and result() %>
        publish:
          - enrichment: <% result() %>
        do:
          - notify_email

  notify_email:
    action: email.send_email
    input:
      email_from: "soc-alerts@dct-ltd.com"
      email_to:
        - "hmiezan@gmail.com"
      subject: "SOC Alert: <% ctx().enrichment.result.extracted_ip %>"
      message: |
        <!DOCTYPE html>
        <html>
        <head><meta charset="UTF-8"><title>SOC Alert</title></head>
        <body>
          <h2>SOC Alert Enrichment</h2>
          <p><b>IP:</b> <% ctx().enrichment.result.extracted_ip %></p>
          <p><b>Reputation:</b> <% ctx().enrichment.result.vt_enrichment.reputation %></p>
          <p><b>Country:</b> <% ctx().enrichment.result.vt_enrichment.country %></p>
          <p><b>ASN:</b> <% ctx().enrichment.result.vt_enrichment.as_owner %></p>
          <p><b>Malicious:</b> <% ctx().enrichment.result.vt_enrichment.last_analysis_stats.malicious %></p>
          <p><b>Suspicious:</b> <% ctx().enrichment.result.vt_enrichment.last_analysis_stats.suspicious %></p>
          <p><b>Harmless:</b> <% ctx().enrichment.result.vt_enrichment.last_analysis_stats.harmless %></p>
        </body>
        </html>
      account: "gmail"
      mime: "html"

### 6. End‑to‑End Testing st2 run my_pack.enrich_ip_flow alert="ET MALWARE Possible Malicious Traffic from 185.220.101.4" vt_api_key="XXXX"

<img width="648" height="501" alt="image" src="https://github.com/user-attachments/assets/d98d5521-e760-4d6c-9954-1f148efa1def" />

