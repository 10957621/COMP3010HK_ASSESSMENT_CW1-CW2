Zui Query
_path == "dhcp" | sort ts
Check the victim base information

event_type == "alert"|sort ts | fuse
Review the alert

_path=="http" method=="GET" | sort ts 
|cut _path, ts, uis, id,method, host, uri 
Review the what is the “Get” request by victim


_path=="http" or _path=="ssl" or _path=="https"
| ts >= 2021-09-24T16:44:40Z
| ts <= 2021-09-24T16:49:40Z
| id.orig_h == 10.9.23.102
| cut ts, id.resp_h, host, server_name, uri, resp_bytes
| sort ts
| fuse

Check the traffic after the zip file download next 5 minutes


WireShark

http contains "documents.zip"
Find the zip file


http.request.method == "POST"
Check the “POST” require by victim

http.host == "maldivehost.net"
beacon evident

_path=="dns" query=="api.ipify.org" | sort ts

smtp



# Forensic Investigation Query Log
**Investigator:** [Your Student ID]
**Date:** January 2026
**Subject:** CW1 Incident Analysis (SquirrelWaffle)

---

##Zui  Queries

### 1. Victim Identification
**Goal:** Identify the victim's IP, MAC address, and Hostname.
**Query:**
_path == "dhcp" | sort ts

### 2. Alert Triage
**Goal:** Review initial IDS alerts to establish a timeline of suspicious activity.
**Query:**
event_type == "alert" | sort ts | fuse

### 3. Initial Infection Vector
**Goal:** Identify the specific HTTP GET request used to download the payload.
**Query:**
_path=="http" method=="GET" | sort ts 
| cut _path, ts, uid, id, method, host, uri 

### 4. Post-Infection Traffic Analysis
**Goal:** Analyze network traffic in the 5-minute window following the download to identify Command & Control (C2) channels.
**Query:**
_path=="http" or _path=="ssl"
| ts >= 2021-09-24T16:44:40Z
| ts <= 2021-09-24T16:49:40Z
| id.orig_h == 10.9.23.102
| cut ts, id.resp_h, host, server_name, uri, resp_bytes
| sort ts
| fuse

### 5. IP Check
**Goal:** Locate the external IP lookup performed by the malware before the spam campaign.
**Query:**
_path=="dns" query=="api.ipify.org" | sort ts

---

## Wireshark Filters

### 1. Payload Extraction
**Goal:** Locate and extract the malicious zip file.
**Filter:**
http contains "documents.zip"

### 2. C2 Data Exfiltration
**Goal:** Filter for HTTP POST requests to identify data being sent to the attacker.
**Filter:**
http.request.method == "POST"

### 3. C2 Beaconing Evidence
**Goal:** Isolate traffic to the confirmed C2 domain (`maldivehost.net`) to observe beaconing patterns.
**Filter:**
http.host == "maldivehost.net"

### 4. Action on Objectives (Spam)
**Goal:** Analyze SMTP traffic to identify the spam campaign and stolen credentials.
**Filter:**
smtp

