# Splunk Home Lab

## Overview
This project documents a **home lab built to simulate attacker activity and detect it using Splunk**. The lab environment consists of two Windows 11 virtual machines and a Kali Linux attacker machine:

- **SOC Workstation** - Runs Splunk Enterprise and performs log analysis
- **Victim Workstation** - Generates logs and forwards them to Splunk using Splunk Universal Forwarder
- **Kali Workstation** - Attacks the victim machine
  
The objective of this lab was to:

- Deploy **Splunk Enterprise and Splunk Universal Forwarder**
- Configure **Sysmon for enhanced endpoint telemetry**
- Simulate attacker activity from **Kali Linux**
- Detect and investigate the attack using **Splunk searches and log analysis**
	


### Skills Learned

- Splunk deployement and log analysis
- Sysmon telemetry investigation
- Windows Defender log analysis
- Attack detection and investigation
- Network reconnaissance detection
- Malware execution monitoring
- Log forwarding and ingestion configuration
- Security lab environment deployment

### Tools Used

- Windows 11 Enterprise
- Splunk Enterprise
- Splunk Universal Forwarder
- Windows Defender
- Sysmon
- Kali Linux
- Nmap
- Metasploit
- msfvenom

## Lab Architecture
### Virtual Machines

**SOC Workstation (192.168.121.147)**
| Resource | Allocation |
|----------|------------|
|RAM       |8 GB        |
|CPU       |4 Cores     |
|Disk      |100 GB      |

**Purpose**
- Hosts **Splunk Enterprise**
- Recieves logs from victim system
- Performs investigation and analysis

**Victim Workstation (192.168.121.148)**
| Resource | Allocation |
|----------|------------|
|RAM       |6 GB        |
|CPU       |2 Cores     |
|Disk      |60 GB       |

**Purpose**
- Runs **Sysmon**
- Runs **Splunk Universal Forwarder**
- Generates logs from simulated attacker activity

**Kali Workstation (192.168.121.128)**
| Resource | Allocation |
|----------|------------|
|RAM       |4 GB        |
|CPU       |2 Cores     |
|Disk      |50 GB       |

**Purpose**
- Attacks Victim Workstation
- Generates payload with msfvenom
- Runs Metasploit
