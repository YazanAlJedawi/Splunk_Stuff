# Splunk Universal Forwarder Configuration Guide



This guide provides step-by-step instructions for configuring a **Splunk Universal Forwarder (UF)** to monitor system logs and forward them to a Splunk indexer. The configuration includes creating a custom app, setting up inputs and outputs, and verifying successful data ingestion.

---

## Table of Contents
1. [🔧 Prerequisites](#-prerequisites)  
2. [🛠️ Configuration Steps](#️-configuration-steps)  
   - [1. Create App Directory Structure](#1-create-app-directory-structure)  
   - [2. Configure Inputs](#2-configure-inputs)  
   - [3. Configure Outputs](#3-configure-outputs)  
   - [4. Set Proper Permissions](#4-set-proper-permissions)  
   - [5. Restart Splunk Universal Forwarder](#5-restart-splunk-universal-forwarder)  
3. [✅ Verification](#-verification)  
   - [1. Check Forwarder Status](#1-check-forwarder-status)  
   - [2. Verify Data Ingestion on Indexer](#2-verify-data-ingestion-on-indexer)  
   - [3. Test Connectivity to Indexer](#3-test-connectivity-to-indexer)  

---

## 🔧 Prerequisites
- Splunk Universal Forwarder installed on the host  
- Access to a Splunk indexer (example: `192.168.0.101:9997`)  
- `sudo` privileges for configuration and service management 
- Create an index via Settings > Indexes if missing (Ex:"linux" index in this demo). 

---

## 🛠️ Configuration Steps

### 1. Create App Directory Structure
```bash
sudo mkdir /opt/splunkforwarder/etc/apps/UF_base_inputs
cd /opt/splunkforwarder/etc/apps/UF_base_inputs
sudo mkdir local metadata
```

### 2. Configure Inputs
Create inputs.conf:
```bash
sudo nano /opt/splunkforwarder/etc/apps/UF_base_inputs/local/inputs.conf
```

Add:

```ini
[monitor:///var/log/]
disabled = 0
index = linux
sourcetype = linux_secure
```
- monitor:///var/log/ → Watches all files in /var/log/ recursively

- disabled = 0 → Enables the input

- index = linux → Target index on the indexer

- sourcetype = linux_secure → Custom sourcetype for easier filtering

### 3. Configure Outputs

Create outputs.conf:
```bash
sudo nano /opt/splunkforwarder/etc/apps/UF_base_inputs/local/outputs.conf
```
Add:
```ini
[tcpout]
defaultGroup = indexers

[tcpout:indexers]
server = 192.168.0.101:9997
compressed = false
```
- defaultGroup = indexers → Routes all data to indexers
- server = 192.168.0.101:9997 → Points to indexer’s receiving endpoint
- compressed = false → Disables compression (set to true if bandwidth is limited)

Note: Replace 192.168.0.101:9997 with your actual Splunk indexer IP and receiving port.

### 4. Set Proper Permissions
```bash
sudo chown -R splunk:splunk /opt/splunkforwarder/etc/apps/UF_base_inputs/
```
- Ensures Splunk user owns the app directory.

### 5. Restart Splunk Universal Forwarder

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```
## Verification

### 1. Check Forwarder Status
```bash
sudo /opt/splunkforwarder/bin/splunk status
```
Check logs:
```bash
sudo tail -f /opt/splunkforwarder/var/log/splunk/splunkd.log
```
Look for:
- Successful connection to indexer
- No permission or parsing errors

### 2. Verify Data Ingestion on Indexer

verify in Splunk Web search head:
```sql
index="_internal" | stats count by source
```
Check forwarded data:
```bash
index=linux | head 10
```

### 3. Test Connectivity to Indexer:
```bash
nc -zv 192.168.0.101 9997
```
If fails, check:

- Firewall (allow TCP 9997)
- Receiver enabled on indexer
- Splunk service running on indexer

## Conclusion

With this setup, your Splunk Universal Forwarder should successfully forward logs to the indexer and make them searchable in Splunk.

Y.