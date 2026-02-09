# BOTSv3 Incident Analysis Report

**Student Name:** [Your Name]
**Student ID:** [Your Student ID]
**Module:** COMP3010HK
**Date:** February 2026

---

## 📹 Video Presentation
[**Click here to watch my video presentation on YouTube**](PASTE_YOUR_YOUTUBE_LINK_HERE)

---

## 1. Introduction
This report presents an incident investigation using the BOTSv3 dataset, focusing on AWS and S3 bucket events in the Frothly environment. Using Splunk as the primary analysis platform, the investigation reconstructs the cyber kill chain, explains how the incident unfolded, and identifies key vulnerabilities and misconfigurations. The primary goal of this investigation is to locate potential data leakage points in Frothly’s AWS environment and understand how they could be exploited, and to derive actionable recommendations that reduce the likelihood of similar incidents in the future. Splunk Enterprise was deployed on an Ubuntu virtual machine as an independent SOC environment for ingesting and analysing BOTSv3 logs. The following sections discuss SOC roles and incident handling, installation and data preparation, guided question analysis, and finally the overall conclusions and presentation strategy.

## 2. SOC Roles & Incident Handling Reflection
In this incident, several preventive controls should have been in place, including mandatory MFA, least‑privilege IAM policies, S3 public access blocks, and alerting on high‑risk configuration changes. Without these measures, the misconfiguration of a public S3 bucket and weak authentication could easily remain unnoticed, leaving Tier 1 with no meaningful alerts to act on. With proper prevention and monitoring, Tier 1 and Tier 2 could use CloudTrail, S3 access logs, and winhostmon data to quickly detect that a bucket had been made public and that an endpoint was behaving abnormally.

Once the incident is confirmed, Tier 2 is responsible for response actions such as isolating affected resources, revoking or rolling back risky access controls, notifying the relevant owners, and documenting findings and actions. Tier 3 then uses this report as input for the recovery and improvement phase, ensuring that new access controls are fully implemented and that lessons learned are translated into updated security policies, detection rules, and playbooks. Currently, BOTSv3 provides raw CloudTrail and S3 access logs without predefined alerts. In a real SOC, these patterns would be implemented as Splunk alerts.
This case illustrates that without CloudTrail and Splunk monitoring, and without controls around S3 public access, a data‑exposure risk could easily be overlooked; a layered SOC model helps ensure that each tier contributes to monitoring, deep analysis, and continuous improvement of the security posture.


## 3. Installation & Data Preparation
For the SOC infrastructure, an Ubuntu 22.04.5 virtual machine was deployed on a Windows host, and Splunk Enterprise 10.0.2 was installed on this VM for the investigation. This setup emulates a typical SOC architecture where a dedicated Linux server hosts the SIEM platform.
Using a VM-based Splunk deployment provides a controllable and isolated environment, separating security log analysis from the local operating system. This aligns with SOC best practices, where critical monitoring components are hosted on dedicated infrastructure rather than analyst workstations. In this lab, Splunk acts as the central SIEM, responsible for receiving, indexing, and querying security events, effectively covering both the data receiver and data enquiry roles.

The following sections describe the VM installation steps and data preparation process in detail.
*   **OS:** Ubuntu Linux
*   **SIEM:** Splunk Enterprise
*   **Data:** BOTSv3 Dataset

### Installation Evidence

![Figure 3.1](Installation%20&%20Data%20Preparation/01.jpg)

Figure 3.1: Configure the VM name and select the Ubuntu ISO image

![Figure 3.2](Installation%20&%20Data%20Preparation/02.jpg)

Figure 3.2: Configure the VM username and password

![Figure 3.3](Installation%20&%20Data%20Preparation/03.jpg)

Figure 3.3: Allocate sufficient processors, RAM, and disk space to the VM

![Figure 3.4](Installation%20&%20Data%20Preparation/04.jpg)

Figure 3.4: Log in to the Ubuntu VM

![Figure 3.5](Installation%20&%20Data%20Preparation/05.jpg)

Figure 3.5: Configure VirtualBox shared folders for file transfer

![Figure 3.6](Installation%20&%20Data%20Preparation/06.jpg)

Figure 3.6: Mount the shared folder in the VM and verify its contents

![Figure 3.7](Installation%20&%20Data%20Preparation/07.jpg)

Figure 3.7: Verify that the Splunk package is suitable for installation using dpkg

![Figure 3.8](Installation%20&%20Data%20Preparation/08.jpg)

Figure 3.8: Install Splunk Enterprise and create the administrator profile

![Figure 3.9](Installation%20&%20Data%20Preparation/09.jpg)

Figure 3.9: Start the Splunk service

![Figure 3.10](Installation%20&%20Data%20Preparation/10.jpg)

Figure 3.10: Import the BOTSv3 dataset into Splunk

![Figure 3.11](Installation%20&%20Data%20Preparation/11.jpg)

Figure 3.11: Verify that the BOTSv3 dataset was successfully ingested

![Figure 3.12](Installation%20&%20Data%20Preparation/12.jpg)

Figure 3.12: Log in to Splunk and run an initial search on the BOTSv3 index

In this environment, the query index=botsv3 earliest=0 | stats count by sourcetype shows that the botsv3 index contains 107 sourcetypes, including critical ones such as aws:cloudtrail and winhostmon. This confirms that both key cloud audit logs and host telemetry are available for the subsequent SOC investigation.


## 4. Guided Questions Analysis
*Here is the evidence for the Quiz questions.*

