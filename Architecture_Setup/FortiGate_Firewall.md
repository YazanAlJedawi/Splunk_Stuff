# Fortigate Firewall Log Forwarding to Splunk via Universal Forwarder

## Overview

This guide documents the process of configuring a Fortigate firewall running on VMware to forward logs to a Splunk indexer through a Universal Forwarder (UF). The setup successfully captures Fortigate traffic logs and forwards them to Splunk for analysis and monitoring.

## Architecture
```bash
Fortigate Firewall (VMware VM) → Splunk Universal Forwarder → Splunk Indexer
                                 192.168.1.108:514            192.168.1.104:9997
```

## Configuration 

### 1. Fortigate Firewall Configuration

Web Interface Setup:
```bash
Navigate to Log & Report > Log Settings

Enable Send logs to syslog

Set IP Address/FQDN: 192.168.1.104 (UF IP address)

Configure logging preferences:

Local Traffic Log: Enabled

Log Local Out Traffic: Enabled
```

## CLI Verification (Optional):

```bash
config log syslogd setting
    set status enable
    set server 192.168.1.104
    set port 514
    set mode udp
end
```

### 2. Splunk Universal Forwarder Configuration
Inputs Configuration:
File: /opt/splunkforwarder/etc/apps/UF_base_inputs/local/inputs.conf

```ini
[udp://514]
index = fgt
sourcetype = fgt
source = fortigate
connection_host = ip
queuesize = 10MB
disabled = 0
```
Key Configuration Parameters:

Port: UDP 514 (default syslog port)

Index: fgt (custom index for Fortigate logs)

Sourcetype: fgt (custom sourcetype)

Source: fortigate (identifies Fortigate logs)

### 3. Universal Forwarder Management
Restart Splunk Service:

```bash
sudo ./bin/splunk restart
```
Verify Port Listening:

```bash
ss -an | grep 514
```
Network Traffic Verification:

```bash
sudo tcpdump -n -vv -i any port 514
```
## Verification and Monitoring
### 1. Log Flow Verification
tcpdump Output Confirmation:

```bash
05:10:04.480996 eth0 In IP 192.168.1.108.15475 > 192.168.1.106.514: SYSL06
```
Confirms successful log transmission from Fortigate to UF.

### 2. Splunk Search Validation
Search Query:

```bash
index=fgt
```
Expected Results:

Events showing Fortigate traffic logs

Proper field extraction (srcip, dstip, action, app, etc.)

Host field populated with Fortigate IP (192.168.1.108)

### 3. Key Log Fields Extracted
host: 192.168.1.108 (Fortigate IP)

source: fortigate

sourcetype: fgt

srcip: Source IP address

dstip: Destination IP address

action: Traffic action (accept/deny)

app: Application protocol

policyid: Firewall policy ID


## Performance Optimization
- Queue Size: Adjust queuesize = 10MB based on log volume

- Batch Processing: Configure appropriate batching settings

- Network Bandwidth: Monitor network utilization during peak log generation

## Conclusion
This setup successfully demonstrates Fortigate log forwarding to Splunk using a Universal Forwarder. The configuration provides:

- Real-time log collection from Fortigate firewall

- Proper field extraction for security analysis

- Scalable architecture for enterprise environments

- Centralized log management and monitoring

The solution enables security teams to monitor firewall traffic, detect anomalies, and perform security investigations using Splunk's powerful analytics capabilities.

Y.