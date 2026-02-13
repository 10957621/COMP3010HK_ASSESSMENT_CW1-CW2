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
This report presents an incident investigation using the BOTSv3 dataset, focusing on AWS and S3 bucket events in the Frothly environment. Using Splunk as the primary analysis platform, the investigation reconstructs the cyber kill chain, explains how the incident unfolded, and identifies key vulnerabilities and misconfigurations. 

The primary goal of this investigation is to locate potential data leakage points in Frothly’s AWS environment and understand how they could be exploited, and to derive actionable recommendations that reduce the likelihood of similar incidents in the future. 

Splunk Enterprise was deployed on an Ubuntu virtual machine as an independent SOC environment for ingesting and analysing BOTSv3 logs. The following sections discuss SOC roles and incident handling, installation and data preparation, guided question analysis, and finally the overall conclusions and presentation strategy.

## 2. SOC Roles & Incident Handling Reflection
In this report, the analysis is carried out from the perspective of a Tier 2 SOC analyst handling an escalated incident. 

### Preventive Controls Analysis
In this incident, several preventive controls should have been in place, including mandatory MFA, least‑privilege IAM policies, S3 public access blocks, and alerting on high‑risk configuration changes. Without these measures, the misconfiguration of a public S3 bucket and weak authentication could easily remain unnoticed, leaving Tier 1 with no meaningful alerts to act on. 

### Detection & Response
With proper prevention and monitoring, Tier 1 and Tier 2 could use CloudTrail, S3 access logs, and winhostmon data to quickly detect that a bucket had been made public and that an endpoint was behaving abnormally.

*   **Tier 2 Responsibility:** Once the incident is confirmed, Tier 2 is responsible for response actions such as isolating affected resources, revoking or rolling back risky access controls, notifying the relevant owners, and documenting findings and actions. 

*   **Tier 3 Responsibility:** Tier 3 then uses this report as input for the recovery and improvement phase, ensuring that new access controls are fully implemented and that lessons learned are translated into updated security policies, detection rules, and playbooks. 

### Conclusion on SOC Visibility
Currently, BOTSv3 provides raw CloudTrail and S3 access logs, it seems without predefined alerts. In a real SOC, these patterns would be implemented as alerts.
This case illustrates that without CloudTrail and Splunk monitoring, and without controls around S3 public access, a data‑exposure risk could easily be overlooked; a layered SOC model helps ensure that each tier contributes to monitoring, deep analysis, and continuous improvement of the security posture.

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
**Figure 3.7:** Verify that the Splunk package is suitable for installation using `dpkg`
 
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

**Splunk Query:**\  
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
index=botsv3 source="aws:cloudtrail" 
| spath output=mfa_status path=userIdentity.sessionContext.attributes.mfaAuthenticated
| search mfa_status=false
| stats count by userIdentity.userName
| sort - count

The Figure 4.2.1 is showing  the  MFA status as false, and though the query Figure 4.2.2 show the all the user who the MFA status as false and count by descending.

![Figure 4.2.2](Quiz/Q2.2.jpg)\
**Figure 4.2.2** 

In a production SOC, lack of MFA on administrative accounts is a critical vulnerability. Alerts should be configured to trigger immediately if a privileged user logs in without MFA tokens.

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

It is known that this incident was triggered by an S3 bucket being configured as public. Therefore, we need to locate the Event ID of the API call to determine the trigger time, identify the user who performed the misconfiguration, and confirm the S3 bucket name.

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

When drilling down into the JSON event under requestParameters, there are two grantees. For each grantee, the uri is set to AllUsers, and the permissions are set to READ and WRITE (with xsi:type set to Group). This indicates that the event configured the S3 bucket ACL to be public accessible by everyone.

In the same JSON event shown in Figures 4.5.1 and 4.6.1, the configuration applies to the bucketName: **"frothlywebcode"** and the userName: **"bstoll"**. This indicates that on **"2018-08-20 at 13:01:46"**, user bstoll configured the frothlywebcode bucket, resulting in it being accessible by everyone.

### Question 7: Impact Assessment

After identifying that the "frothlywebcode" bucket has been misconfigured, as a SOC Tier 2 analyst, I am required to determine if the exposure was exploited. Use the following query to look for PUT and GET requests from external IP addresses targeting the "frothlywebcode" bucket.

**Splunk Query:**\
index=botsv3 sourcetype="aws:s3:accesslogs" "frothlywebcode" "PUT"
| rex field=_raw "(?<client_ip>\d+\.\d+\.\d+\.\d+)"
| rex field=_raw "^\S+\s+(?<bucket>\S+)\s+\["
| rex field=_raw "\"PUT\s+/(?<object_key>[^ ]+)\s+HTTP/1\.1\""
| table _time client_ip bucket object_key

![Figure 4.7.1](Quiz/Q7.1.jpg)\
**Figure 4.7.1**

**Splunk Query:**\
index=botsv3 sourcetype="aws:s3:accesslogs" "frothlywebcode" "REST.GET.OBJECT"
| rex field=_raw "(?<client_ip>\d+\.\d+\.\d+\.\d+)"
| rex field=_raw "^\S+\s+(?<bucket>\S+)\s+\["
| rex field=_raw "\"GET\s+/(?<object_key>[^ ]+)\s+HTTP/1\.1\""
| table _time client_ip bucket object_key

![Figure 4.7.2](Quiz/Q7.2.jpg)\
**Figure 4.7.2**

In Figure 4.7.1 is show someone put a. text file **“OPEN_BUCKET_PLEASE_FIX.txt” **into the bucket from IP address 52.66.146.128, and IP address 35.182.46.222 put the **frothly_html_memcached.tar.gz”** into the bucket.  From  Figure 4.7.3 in the event detail of IP address 107.77.212.175 can prove the user as bstoll configure the acl and using the pre-signed URL to upload the **”frothly_html_memcached.tar.gz”** From  Figure 4.7.4.

![Figure 4.7.3](Quiz/Q7.3.jpg)\
**Figure 4.7.3**

![Figure 4.7.4](Quiz/Q7.4.jpg)\
**Figure 4.7.4**

Figure 4.7.1 shows that someone uploaded a text file "OPEN_BUCKET_PLEASE_FIX.txt" to the bucket from IP address 52.66.146.128, while IP address 35.182.46.222 uploaded "frothly_html_memcached.tar.gz" to the bucket. As shown in Figure 4.7.3 and 4.7.4, the event details from IP address 107.77.212.175 indicate that user bstoll configured the ACL and used a pre-signed URL to upload **"frothly_html_memcached.tar.gz”**.

Figure 4.7.2 shows GET requests to the "frothlywebcode" bucket from external IP addresses. "frothly_html_memcached.tar.gz" was downloaded by 35.182.246.222, 35.183.247.244, and 52.53.233.88, confirming the bucket exposure was exploited.

This evidence proves that bstoll misconfigured the "frothlywebcode" bucket, which enabled unauthorized external users to both upload and download files.

