# SOC-Home-Lab

## Introduction
This document provides a guide for setting up a Security Operations Center (SOC) home lab. The lab consists of two virtual machines (VMs): an attacker machine running Kali Linux and a victim machine running Windows 10. The primary objective of this lab is to simulate real-world security scenarios, analyze attacks, and enhance practical cybersecurity skills.

## Objective
The objective of this lab is to:  
- Simulate cyber attacks in a controlled environment.  
- Monitor and analyze system and network logs using SIEM tools.  
- Gain hands-on experience with Sysmon, Splunk, and Windows Event Logs.  
- Understand attack techniques and learn defensive strategies.  
- Develop practical cybersecurity skills applicable to a SOC analyst role.

## Prerequisites

| Category                    | Requirements                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| **Operating Systems**        | Windows 10 ISO, Kali Linux Pre-built VM                                      |
| **System Requirements**      | RAM: 8GB or Higher, Internet Connectivity for downloads/updates |
| **Logging & Monitoring Tools** | Splunk (SIEM & security analytics), Sysmon (System monitoring & event logging) |
| **Virtualization Software**   | VirtualBox                                                                  |

## Network Topology
Below is a simple diagram that illustrates the network topology for this lab setup:
[Kali Linux (Attacker)] ---> [Windows 10 VM (Target)] ---> [Splunk (Log Monitoring)]
The Kali Linux machine attacks the Windows VM, and logs are collected by Splunk for analysis.

## Step 1: Setting Up Virtual Machines

### 1.1 Install Kali Linux (Attacker Machine)
1. Download Kali Linux ISO from the [Kali Official Website](https://www.kali.org/).  
2. In VirtualBox, create a new virtual machine by clicking **Tools → Add**, and follow the prompts to install Kali Linux.

### 1.2 Install Windows 10 (Target Machine)
1. Download Windows 10 installation media from [Microsoft's website](https://www.microsoft.com/en-us/software-download/windows10).  
2. Create ISO file from the installation media.  
3. Create a new virtual machine in VirtualBox and install Windows 10 using the ISO file.

## Step 2: Install Sysmon on Windows 10
1. Download Sysmon from [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).  
2. Download a pre-configured `sysmonconfig.xml` from [Sysmon Modular](https://github.com/SwiftOnSecurity/sysmon-config).  
3. Extract the Sysmon files and place the `sysmonconfig.xml` file into the extracted directory.  
4. Open PowerShell as an administrator and navigate to the Sysmon directory:
  ```powershell
cd "C:\Users\Downloads\sysmon"
```
5. Verify Sysmon is running:

```powershell
Get-Process sysmon64
```
6. Install Sysmon with the configuration:

```powershell
.\sysmon64.exe -i sysmonconfig.xml
```
## Step 3: Install Splunk on Windows 10

1. **Download Splunk Free** from the Splunk website.  
2. **Install Splunk** on your Windows 10 VM.  
3. Go to the folder where Splunk is installed:  
   `splunk → system → local`  
   and create a file named **inputs.conf** with the following content:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

[WinEventLog://Microsoft-Windows-Windows Defender/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-Windows Defender/Operational
blacklist = 1151,1150,2000,1002,1001,1000

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
index = endpoint
disabled = false
source = Microsoft-Windows-PowerShell/Operational
blacklist = 4100,4105,4106,40961,40962,53504

[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false
```
4. **Launch Splunk** and log in with administrative credentials.
5. **Enable data collection** in Splunk.  
3. To parse Sysmon data, go to **Apps → Find More Apps** and install the **Splunk Add-on for Sysmon**.  
4. **Create a new index** named `endpoint` for Sysmon logs:  
   - Navigate to **Settings → Indexes**  
   - Click **New Index**  
   - Enter the name `endpoint`  
   - Save the configuration  


## Step 4: Setting Up a Separate Network  

 Configure both VMs (Windows 10 and Kali Linux) to use the **same Internal Network** in VirtualBox settings. Assign **static IP addresses** to the respective network adapters on each VM, ensuring they are on the **same subnet**.
 
1. To enable interaction between the Windows 10 and Kali Linux VMs, create a **private network**:  
   - Open **VirtualBox → Settings** for each VM  
   - Click on **Network**  
   - Change **Adapter 1**'s attachment type to **Internal Network**  
   - Give the internal network a name  
   - Click **OK**
  
      <img width="512" height="315" alt="1_network_1" src="https://github.com/user-attachments/assets/373dfc0d-44a2-424c-84e5-ff126e1531dd" />
        <img width="512" height="315" alt="1_network_2" src="https://github.com/user-attachments/assets/cc8f4785-2508-483e-884b-16b8213752fb" />


2.

This setup allows the VMs to communicate with each other on an isolated network.  
