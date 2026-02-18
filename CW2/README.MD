#  Courcework 2: BOTSv3 Incident Analysis Report

**Student ID:** [10957621]\
**Module:** COMP3010HK Security Operations and Incident Management\
**Date:** February 2026

---
## Contents
1. [GitHub](#github)
2. [Video Presentation](#video-presentation)
3. [1. Introduction](#1-introduction)
4. [2. SOC Roles & Incident Handling Reflection](#2-soc-roles--incident-handling-reflection)
    - [Preventive Controls Analysis](#preventive-controls-analysis)
    - [Detection & Response](#detection--response)
    - [Conclusion on SOC Visibility](#conclusion-on-soc-visibility)
5. [3. Installation & Data Preparation](#3-installation--data-preparation)
    - [Installation Evidence](#installation-evidence)
    - [Data Validation](#data-validation)
6. [4. Guided Questions Analysis](#4-guided-questions-analysis)
    - [Question 1: Identifying IAM Users](#question-1-identifying-iam-users)
    - [Question 2: Multi-Factor Authentication Check](#question-2-multi-factor-authentication-check)
    - [Question 3: Hardware Inventory](#question-3-hardware-inventory)
    - [Question 4-6: S3 bucket](#question-4-6-s3-bucket)
    - [Question 7: Impact Assessment](#question-7-impact-assessment)
    - [Question 8: FQDN](#question-8-fqdn)
7. [5. Conclusion, References and Presentation](#5-conclusion-references-and-presentation)
8. [Reference](#reference)
9. [Appendix A - Cryptojacking](#appendix-a---cryptojacking)
10. [Appendix B - Student Declaration of AI Tool Use](#appendix-b---student-declaration-of-ai-tool-use)
11. [Appendix C - GitHub Repository Availability & Incident Report](#appendix-c---github-repository-availability--incident-report)
    - [1. Incident Summary](#1-incident-summary)
    - [2. Impact on Submission and Incident Response & Verification](#2-impact-on-submission-and-incident-response--verification)
    - [3. Evidence of the Github work](#3-evidence-of-the-github-work)
---

## GitHub
If the link returns a 404 error, please refer to Appendix C – GitHub Repository Availability & Incident Report for details.
**https://github.com/10957621/COMP3010HK_ASSESSMENT_CW1-CW2.git**
![GitHub0](Github0.png)
![GitHub](GitHub.png)
## Video Presentation
**https://www.youtube.com/watch?v=SM90B3rc2lk**
![youtubepreview](youtubepreview.png)

---

## 1. Introduction
This report presents an incident investigation using the BOTSv3 dataset[1], focusing on AWS and S3 bucket events in the Frothly environment. Using Splunk as the primary analysis platform, the investigation reconstructs the cyber kill chain following the NIST Incident Response Framework [2], explains how the incident unfolded, and identifies key vulnerabilities and misconfigurations. 

The primary goal of this investigation is to locate potential data leakage points in Frothly’s AWS environment and understand how they could be exploited, and to derive actionable recommendations that reduce the likelihood of similar incidents in the future. 

Splunk Enterprise[8] was deployed on an Ubuntu virtual machine as an independent SOC environment for ingesting and analysing BOTSv3 logs. The following sections discuss SOC roles and incident handling, installation and data preparation, guided question analysis, and finally the overall conclusions and presentation strategy.

## 2. SOC Roles & Incident Handling Reflection
In this report, the analysis is carried out from the perspective of a Tier 2 SOC analyst[6]handling an escalated incident. 

### Preventive Controls Analysis
In this incident, several preventive controls should have been in place, including mandatory MFA, least‑privilege IAM policies, S3 public access blocks, and alerting on high‑risk configuration changes. Without these measures, the misconfiguration of a public S3 bucket and weak authentication could easily remain unnoticed, leaving Tier 1 with no meaningful alerts to act upon, which significantly hampers the effectiveness of the initial triage phase [9]. 

### Detection & Response
With proper prevention and monitoring, Tier 1 and Tier 2 could use CloudTrail, S3 access logs, and winhostmon data to quickly detect that a bucket had been made public and that an endpoint was behaving abnormally.

*   **Tier 2 Responsibility:** Once the incident is confirmed, Tier 2 is responsible for response actions such as isolating affected resources, revoking or rolling back risky access controls, notifying the relevant owners, and documenting findings and actions. 

*   **Tier 3 Responsibility:** Tier 3 then uses this report as input for the recovery and improvement phase, ensuring that new access controls are fully implemented and that lessons learned are translated into updated security policies, detection rules, and playbooks. 

### Conclusion on SOC Visibility
Currently, BOTSv3 provides raw CloudTrail and S3 access logs, it seems without predefined alerts. In a real SOC, these patterns would be implemented as alerts.

## 3. Installation & Data Preparation
For the SOC infrastructure, an Ubuntu 22.04.5 virtual machine was deployed on a Windows host, and Splunk Enterprise 10.0.2 was installed on this VM for the investigation. This setup emulates a typical SOC architecture where a dedicated Linux server hosts the SIEM platform.
Using a VM-based Splunk deployment provides a controllable and isolated environment, separating security log analysis from the local operating system. This aligns with SOC best practices, where critical monitoring components are hosted on dedicated infrastructure rather than analyst workstations. In this lab, Splunk acts as the central SIEM, responsible for receiving, indexing, and querying security events, effectively covering both the data receiver and data enquiry roles.

The following sections describe the VM installation steps and data preparation process in detail.
*   **OS:** Ubuntu 22.04.5
*   **SIEM:** Splunk Enterprise 10.0.2
*   **Data:** BOTSv3 Dataset

### Installation Evidence

![Figure 3.1](Installation%20&%20Data%20Preparation/01.jpg)\
**Figure 3.1:** Configure the VM name and select the Ubuntu ISO image  
  
![Figure 3.2](Installation%20&%20Data%20Preparation/02.jpg)\
**Figure 3.2:** Configure the VM username and password
  
![Figure 3.3](Installation%20&%20Data%20Preparation/03.jpg)\
**Figure 3.3:** Allocate sufficient processors, RAM, and disk space to the VM

![Figure 3.4](Installation%20&%20Data%20Preparation/04.jpg)\  
**Figure 3.4:** Login to the Ubuntu VM  
 
![Figure 3.5](Installation%20&%20Data%20Preparation/05.jpg)\
**Figure 3.5:** Configure VirtualBox shared folders for file transfer 
  
![Figure 3.6](Installation%20&%20Data%20Preparation/06.jpg)\  
**Figure 3.6:** Mount the shared folder in the VM and verify its contents
  
![Figure 3.7](Installation%20&%20Data%20Preparation/07.jpg)\  
**Figure 3.7:** Verify that the Splunk package is suitable for installation using dpkg
 
![Figure 3.8](Installation%20&%20Data%20Preparation/08.jpg)\
**Figure 3.8:** Install Splunk Enterprise and create the administrator profile 
  
![Figure 3.9](Installation%20&%20Data%20Preparation/09.jpg)\
**Figure 3.9:** Start the Splunk service
  
![Figure 3.10](Installation%20&%20Data%20Preparation/10.jpg)\  
**Figure 3.10:** Import the BOTSv3 dataset into Splunk
 
![Figure 3.11](Installation%20&%20Data%20Preparation/11.jpg)\
**Figure 3.11:** Verify that the BOTSv3 dataset was successfully ingested 
  
![Figure 3.12](Installation%20&%20Data%20Preparation/12.jpg)\
**Figure 3.12:** Log in to Splunk and run an initial search on the BOTSv3 index

### Data Validation
In this environment, the validation query index=botsv3 earliest=0 | stats count by sourcetype shows that the botsv3 index contains 107 sourcetypes, including critical ones such as aws:cloudtrail and winhostmon. This confirms that both key cloud audit logs and host telemetry are available for the subsequent SOC investigation. From a SOC perspective, validating sourcetype coverage, event volumes, and field extraction is a critical onboarding step before building detections and dashboards on top of a new data source.

While this all‑in‑one Splunk deployment on a single Ubuntu VM is sufficient for this incident analysis, it has limitations compared to a production environment. It does not separate indexer and search head roles. These are acceptable in a lab context because they preserve the core SIEM concepts—centralised logging, correlation, and tiered analysis—while keeping the environment simple enough to deploy and replicate for this report.


## 4. Guided Questions Analysis
In a production environment, incident analysis typically begins with an automated alert. This report adopts the persona of a Tier 2 SOC Analyst at **Frothly** who has received a ticket regarding suspicious configuration changes within the organization's Amazon AWS cloud infrastructure.

The objective of this section is to conduct a forensic investigation using Splunk to determine the full scope of the incident. This includes identifying the trigger point, tracking the suspect via Identity & Access Management (IAM), assessing the scope of impact which is public data exposure, and capturing infrastructure evidence to support remediation.

### Question 1: Identifying IAM Users

The investigation began by identify who exists in the environment  queried CloudTrail logs to list all unique user identities to understand who exists in the environment.

**Splunk Query:**
index=botsv3 sourcetype="aws:cloudtrail" userIdentity.type="IAMUser" 
| stats count by userIdentity.userName

![Figure 4.1](Quiz/Q1.jpg)\
**Figure 4.1** 

The IAM users identified are:\ 
bstoll, btun, splunk_access, web_admin

In SOC environment, any account found here that does not map to a current list would be flagged for immediate disablement.

### Question 2: Multi-Factor Authentication Check

After identify who exists in the environment drilled down into the JSON of the CloudTrail logs to check the status of Multi-Factor Authentication (MFA) Status.

The specific JSON path used to identify non-MFA sessions is: **userIdentity.sessionContext.attributes.mfaAuthenticated**.

![Figure 4.2.1](Quiz/Q2.1.jpg)\
**Figure 4.2.1** 

**Splunk Query:**\
index=botsv3 sourcetype="aws:cloudtrail" 
| spath output=mfa_status path=userIdentity.sessionContext.attributes.mfaAuthenticated
| search mfa_status=false
| stats count by userIdentity.userName
| sort - count

The Figure 4.2.1 is showing  the  MFA status as false, and though the query Figure 4.2.2 show the all the user who the MFA status as false and count by descending.

![Figure 4.2.2](Quiz/Q2.2.jpg)\
**Figure 4.2.2** 

In a production SOC, lack of MFA on administrative accounts is a critical vulnerability, and CISA [7] explicitly recommends phishing-resistant MFA for all privileged cloud accounts."

### Question 3: Hardware Inventory

After identifying who can access this environment, the second step is to identify the web servers' hardware specifications for asset inventory. This pivots to WinHostMon and hardware data sources, enabling correlation between incidents and physical assets.

**Splunk Query:**\
index=botsv3 sourcetype="WinHostMon"
| stats count by Type


![Figure 4.3.1](Quiz/Q3.1.jpg)\
**Figure 4.3.1**

Figure 4.3.1 shows the WinHostMon event counts by Type (processor, disk, logical), representing Windows host monitoring information from various terminals.

**Splunk Query:**\
index=botsv3 sourcetype=hardware
| stats count by host

![Figure 4.3.2](Quiz/Q3.2.jpg)\
**Figure 4.3.2**

Figure 4.3.2 lists all web servers in the dataset, all using Intel(R) Xeon(R) CPU E5-2676 v3 @ 2.40GHz.

Hardware inventory is critical in a SOC and in digital forensics because it provides a complete catalog of system assets. It helps organizations identify the scope of affected devices and assess their risk exposure. In digital forensics, it can confirm the system configuration at the time of an incident and support device authenticity verification.

### Question 4-6: S3 bucket 

It is known that this incident was triggered by an S3 bucket being configured as public. To investigate this, I utilized the aws:cloudtrail sourcetype, which is the standard log source for recording AWS API calls [5]. By searching these logs, I located the specific Event ID to determine the trigger time, identify the user who performed the misconfiguration, and confirm the S3 bucket name.

**Splunk Query:**\
index=botsv3 sourcetype="aws:cloudtrail" eventName="PutBucketAcl" "AllUsers"
| table eventTime eventSource eventName requestParameters eventide

![Figure 4.4.1](Quiz/Q4.1.jpg)\
**Figure 4.4.1**

![Figure 4.5.1](Quiz/Q5.1.jpg)\
**Figure 4.5.1**

![Figure 4.6.1](Quiz/Q6.1.jpg)\
**Figure 4.6.1**

In Figure 4.4.1, the query lists all eventName="PutBucketAcl" events where the grantee is AllUsers within sourcetype="aws:cloudtrail". In this incident, the relevant eventID is **“ab45689d-69cd-41e7-8705-5350402cf7ac**

When drilling down into the JSON event under requestParameters, there are two grantees. For each grantee, the uri is set to AllUsers, and the permissions are set to READ and WRITE (with xsi:type set to Group). This indicates that the event configured the S3 bucket ACL to be publicly accessible, violating AWS Security Best Practices [3].

In the same JSON event shown in Figures 4.5.1 and 4.6.1, the configuration applies to the bucketName: **"frothlywebcode"** and the userName: **"bstoll"**. This indicates that on **"2018-08-20 at 13:01:46"**, user bstoll configured the frothlywebcode bucket, resulting in it being accessible by everyone.

### Question 7: Impact Assessment

After identifying that the "frothlywebcode" bucket has been misconfigured, as a SOC Tier 2 analyst, I am required to determine if the exposure was exploited. Use the following query to look for PUT and GET requests from external IP addresses targeting the "frothlywebcode" bucket.

**Splunk Query:**\
index=botsv3 sourcetype="aws:s3:accesslogs" "frothlywebcode" "PUT"
| rex field=_raw "(?<client_ip>\d+\.\d+\.\d+\.\d+)"
| rex field=_raw "^\S+\s+(?<bucket>\S+)\s+\["
| rex field=_raw "\"PUT\s+/(?<object_key>[^ ]+)\s+HTTP/1\.1\""
| table _time client_ip bucket object_key

![Figure SPL7.1](Quiz/SPL7.1.png)\

![Figure 4.7.1](Quiz/Q7.1.jpg)\
**Figure 4.7.1**

**Splunk Query:**\
index=botsv3 sourcetype="aws:s3:accesslogs" "frothlywebcode" "REST.GET.OBJECT" 
| rex field=_raw "(?<client_ip>\d+.\d+.\d+\.\d+)" 
| rex field=_raw "^\S+\s+(?<bucket>\S+)\s+\[" 
| rex field=_raw "\"GET\s+/(?<object_key>[^ ]+)\s+HTTP/1\.1\""
| table _time client_ip bucket object_key

![Figure SPL7.2](Quiz/SPL7.2.png)\

![Figure 4.7.2](Quiz/Q7.2.jpg)\
**Figure 4.7.2**

In Figure 4.7.1 is show someone put a. text file **“OPEN_BUCKET_PLEASE_FIX.txt”** into the bucket from IP address 52.66.146.128, and IP address 35.182.46.222 put the **"frothly_html_memcached.tar.gz”** into the bucket.  From  Figure 4.7.3 in the event detail of IP address 107.77.212.175 can prove the user as bstoll configure the acl and using the pre-signed URL to upload the **”frothly_html_memcached.tar.gz”** From  Figure 4.7.4.

![Figure 4.7.3](Quiz/Q7.3.jpg)\
**Figure 4.7.3**

![Figure 4.7.4](Quiz/Q7.4.jpg)\
**Figure 4.7.4**

Figure 4.7.1 shows that an external actor at IP 52.66.146.128 uploaded a text file named OPEN_BUCKET_PLEASE_FIX.txt. This is the specific file requested in the investigation.Additionally, IP 35.182.46.222 uploaded frothly_html_memcached.tar.gz.Figure 4.7.2 confirms that frothly_html_memcached.tar.gz was subsequently downloaded by multiple external IPs (35.182.246.222, 52.53.233.88), confirming that the bucket exposure was actively exploited for data exfiltration.

### Question 8: FQDN

With bstoll identified as the suspect, the next step was to locate his physical endpoint device. The following query is lists out all host and then find the computer name of user bstoll.

**Splunk Query:**\
index=botsv3 sourcetype="WinHostMon" 
| dedup host 
| table host
| sort host

![Figure 4.8.1](Quiz/Q8.1.jpg)\
**Figure 4.8.1**

**Splunk Query:**\
index=botsv3 "BSTOLL-L"  host="BSTOLL-L"

![Figure 4.8.2](Quiz/Q8.2.jpg)\
**Figure 4.8.2**

Figure 4.8.1 shows all hosts that have been identified, and Figure 4.8.2 shows that the FQDN of host **"BSTOLL-L"** is **"BSTOLL-L.froth.ly"**

**Splunk Query:**\
index=botsv3 sourcetype="WinHostMon" Type=OperatingSystem 
| stats count by host, OS

![Figure 4.8.3](Quiz/Q8.3.jpg)\
**Figure 4.8.3**

Figure 4.8.3 shows which operating system each host uses. BSTOLL-L is using Microsoft window 10 Enterprise.

## 5. Conclusion, References and Presentation 
This investigation confirmed a data exposure incident within Frothly's AWS environment. The root cause was identified as a misconfiguration by user bstoll, who modified the frothlywebcode S3 bucket's Access Control List (ACL) to public via the PutBucketAcl API (Event ID: ab45689d-69cd-41e7-8705-5350402cf7ac).

This exposure allowed external actors to upload a warning file, 'OPEN_BUCKET_PLEASE_FIX.txt', and a potentially malicious archive, 'frothly_html_memcached.tar.gz', which could be downloaded, modified, and re-uploaded. Analysis of the associated endpoint (BSTOLL-L) revealed indicators of cryptojacking, specifically sustained 100% CPU usage by browser processes, suggesting the user's compromise extended from the cloud environment to the physical endpoint.

The incident highlights that while logs were being collected, a lack of active alerting prevented Tier 1 analysts from responding. Furthermore, the absence of Multi-Factor Authentication (MFA) for multiple users represents a fundamental failure in identity security.

**Recommendations:**

1.Enable 'Block Public Access' at the AWS account level to prevent accidental bucket exposure.

2.Enforce MFA for all IAM users and rotate bstoll's credentials immediately.

3.Create high-severity Splunk alerts for specific CloudTrail events (like PutBucketAcl) and endpoint alerts for sustained high CPU usage associated with known mining pools to detect cryptojacking."

## Reference

[1] Splunk, "Boss of the SOC (BOTS) Version 3 Dataset," *GitHub Repository*, 2020. [Online]. Available: https://github.com/splunk/botsv3. [Accessed: Feb. 14, 2026].

[2] P. Cichonski, T. Millar, T. Grance, and K. Scarfone, "Computer Security Incident Handling Guide," *National Institute of Standards and Technology (NIST)*, Gaithersburg, MD, Special Publication 800-61 Rev. 2, Aug. 2012. doi: 10.6028/NIST.SP.800-61r2.

[3] Amazon Web Services, "Security Best Practices for Amazon S3," *AWS Documentation*, 2024. [Online]. Available: https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html.

[4] MITRE, "Resource Hijacking (T1496)," *MITRE ATT&CK Framework*, 2024. [Online]. Available: https://attack.mitre.org/techniques/T1496/.

[5] Amazon Web Services, "Logging Amazon S3 API calls using AWS CloudTrail," *AWS Documentation*, 2024. [Online]. Available: https://docs.aws.amazon.com/AmazonS3/latest/userguide/cloudtrail-logging.html.

[6] C. Crowley and J. Pescatore, "Common and Best Practices for Security Operations Centers: Results of the 2019 SOC Survey," *SANS Institute*, 2019. [Online]. Available: https://www.sans.org/white-papers/39350/.

[7] Cybersecurity and Infrastructure Security Agency (CISA), "Implementing Phishing-Resistant MFA," *CISA Fact Sheet*, 2023. [Online]. Available: https://www.cisa.gov/resources-tools/resources/implementing-phishing-resistant-mfa.

[8] Splunk, "Splunk Enterprise Documentation: About Splunk Enterprise," *Splunk Docs*, 2024. [Online]. Available: https://docs.splunk.com/Documentation/Splunk/latest/Overview/AboutSplunkEnterprise.

[9] E. D. Kilgallon, "Improving the Effectiveness of the Tier 1 Security Analyst," *SANS Reading Room*, 2020.

---
## Appendix A - Cryptojacking
*(This section provides supplementary forensic analysis beyond the guided questions)*

The previous section showed that Frothly's S3 bucket was misconfigured. Since the dataset is from 2018, a period when cryptojacking was prevalent, I hypothesized that the compromised endpoint (BSTOLL-L) might be infected.

**Splunk Query:**
index=botsv3 sourcetype="PerfmonMk:Process"
| rex field=_raw "^(?<extracted_process_name>[^\s]+)\s+\S+\s+(?<extracted_cpu>\S+)"
| search extracted_process_name="*"
| search extracted_cpu > 0
| stats avg(extracted_cpu) as Avg_CPU max(extracted_cpu) as Max_CPU by extracted_process_name, host
| sort -Max_CPU

![Figure 4.9.0](Quiz/Q9.0.jpg)\
**Figure 4.9.0**

Figure 4.9.0 shows that the browser process is unusually consuming 100% of the CPU. Figures 4.9.1 to 4.9.3 confirm that the browser's high CPU usage time matches the DNS record, indicating a browser-based cryptojacking incident,, which aligns with MITRE ATT&CK technique T1496 (Resource Hijacking) [4].

**Splunk Query:**\
index=botsv3 sourcetype="PerfmonMk:Process" host="BSTOLL-L"
| rex field=_raw "^(?<extracted_process_name>[^\s]+)\s+\S+\s+(?<extracted_cpu>\S+)"
| search extracted_process_name IN ("_Total", "chrome#4", "chrome#5", "TiWorker", "MsMpEng", "chrome#6")
| timechart span=1m max(extracted_cpu) by extracted_process_name limit=10

![Figure 4.9.1](Quiz/Q9.1.jpg)\
**Figure 4.9.1**

**Splunk Query:**\
index=botsv3 sourcetype="stream:dns" query!=" " query!="-"
earliest="08/20/2018:21:39:00"
| table _time, query
| sort _time

![Figure 4.9.2](Quiz/Q9.2.jpg)\
**Figure 4.9.2**\
![Figure 4.9.3](Quiz/Q9.3.jpg)\
**Figure 4.9.3**

## Appendix B - Student Declaration of AI Tool Use

## Appendix B - Student Declaration of AI Tool Use

The complete Student Declaration PDF follows on the next page.

[View Student Declaration PDF](./Student%20Declaration%20of%20AI%20Tool%20use%20in%20this%20Assessment.pdf)



## Appendix C  - GitHub Repository Availability & Incident Report
Date of Report: 16 Feb 2026
GitHub Username: 10957621
Repository URL: https://github.com/10957621/COMP3010HK_ASSESSMENT_CW1-CW2

### 1. Incident Summary
On 2026-02-16 12:38 UTC, I received an email from GitHub with the subject **"Password reset notification due to suspicious activity"**. Therefore, I believe my GitHub account was flagged by GitHub's automated security systems due to a detected unauthorized login attempt (Account Takeover Protection). As a result, my public profile and repositories are currently hidden from public view and return a 404 error. I reset the password and enable the MFA immediately.
![Figure c.1.1](Appendix_C/c.1.1.png)\
**Figure C.1.1**

![Figure c.1.2](Appendix_C/c.1.2.png)\
**Figure C.1.2**

![Figure c.1.3](Appendix_C/c.1.3.png)\
**Figure C.1.3**

![Figure c.1.4](Appendix_C/c.1.4.png)\
**Figure C.1.4**

![Figure c.1.5](Appendix_C/c.1.5.png)\
**Figure C.1.5**

### 2. Impact on Submission and Incident Response & Verification

Then I checked the SSH and GPG keys and also reviewed the authorized applications.

![Figure c.2.1](Appendix_C/c.2.1.png)\
**Figure C.2.1**

![Figure c.2.2](Appendix_C/c.2.2.png)\
**Figure C.2.2**

![Figure c.2.3](Appendix_C/c.2.3.png)\
**Figure C.2.3**

I can still upload or edit the repository, but all images I upload seem to be restricted by GitHub policy and show as damaged. Even though the repository is set as public, only I can see the content, which means it is not ready for assessment review. To fix this, I submitted a request to the GitHub Support team and explained the situation to request further assistance.

![Figure c.2.4](Appendix_C/c.2.4.png)\
**Figure C.2.4**

![Figure c.2.5](Appendix_C/c.2.5.png)\
**Figure C.2.5**

![Figure c.2.6](Appendix_C/c.2.6.png)\
**Figure C.2.6**


### 3. Evidence of the Github work

Since my GitHub account may require a long time to complete the reinstatement request, this section presents my work for this module.

This repository contains two folders corresponding to Coursework 1 and Coursework 2.

![Figure c.3.1](Appendix_C/c.3.1.png)\
**Figure C.3.1**


#### 3.1 CW1

*The details of CW1 follow on the next page. The evidence, such as screenshots, has already been inserted in the report and submitted in DLE previously.*

**Attached Documents:**
- 📄 [CW1 Assessment](Appendix_C/COMP3010HK_ASSESSMENT_CW1-CW2%3ACW1%20at%20main%20%C2%B7%2010957621%3ACOMP3010HK_ASSESSMENT_CW1-CW2.pdf)
- 📄 [CW1 Evidence](Appendix_C/COMP3010HK_ASSESSMENT_CW1-CW2%3ACW1%3AEvidence%20at%20main%20%C2%B7%2010957621%3ACOMP3010HK_ASSESSMENT_CW1-CW2.pdf)
- 📄 [CW1 Virus Total](Appendix_C/COMP3010HK_ASSESSMENT_CW1-CW2%3ACW1%3AVirus%20Total%20at%20main%20%C2%B7%2010957621%3ACOMP3010HK_ASSESSMENT_CW1-CW2.pdf)

#### 3.2 CW2

**Important Note**: The following Git commit records and repository evidence 
were captured on 2026-02-16 after the incident occurred, but the commit 
timestamps themselves reflect the actual dates when the work was completed 
(prior to the security incident). Git commit metadata is cryptographically 
signed and tamper-evident, ensuring the authenticity of these timestamps.

Despite the public visibility issue, I retain full access to my local 
repository. The following evidence demonstrates my coursework completion 
and authorship:

The CW2 folder contains three subfolders: quiz, Installation & Data Preparation, and Appendix_C. These folder stores the screenshots used as evidence for the CW2 report.

![Figure c.3.2.1](Appendix_C/c.3.2.1.png)\
**Figure C.3.2.1**

![Figure c.3.2.2](Appendix_C/c.3.2.2.png)\
**Figure C.3.2.2**

![Figure c.3.2.3](Appendix_C/c.3.2.3.png)\
**Figure C.3.2.3**

#### 3.3 CW2 Commits History
![Figure commits_1](Appendix_C/commits_1.png)
**Figure 3.3.1**

![Figure commits_2](Appendix_C/commits_2.png)
**Figure 3.3.2**

![Figure commits_3](Appendix_C/commits_3.png)
**Figure 3.3.3**

![Figure commits_4](Appendix_C/commits_4.png)
**Figure 3.3.4**

![Figure commits_5](Appendix_C/commits_5.png)
**Figure 3.3.5**
