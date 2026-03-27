# Endpoint-Monitoring-with-Sysmon: Windows Event Log Analysis & Threat Detection

## Objective

The focus of this lab is to understand how to implement and analyze Windows Event Logs and Sysmon to mimic endpoint detection and response. This project creates hands-on experience with identifying malicious activities using log analysis. This lab concludes with identifying reconnaissance activities such as Nmap scans. This is an extension of basic blue team concepts that are applicable in real-world security.

### Skills Learned 

This project has provided hands-on experience with the following security solutions/skills:

-  Windows Event Log analysis using Event Viewer
-  Sysmon installation, configuration, and log monitoring
-  Process creation analysis (Sysmon Event ID 1 / Windows Event ID 4688)
-  Network connection monitoring and investigation (Sysmon Event ID 3)
-  Network connection monitoring and investigation (Sysmon Event ID 3)
-  Log filtering and event correlation using Event IDs and timestamps
-  Detection of reconnaissance activity (e.g., Nmap port scanning) through log patterns

### Tools Used

- VirtualBox 
- VirtualBox VMs (Windows 10 server, Kali Linux) 
- Sysmon 
- Event Viewer 
- Command-Line tools (notepad, calc, whoami, ipconfig, and ping) 
- Nmap

## Current lab environment Architecture/Setup

Below is a screenshot of my current lab Architecture (IM1). To quickly summarize, I have three VM's within VirtualBox, a Windows VM, and two Linux VM's, kali Linux and Ubuntu. They are all routed to my pfSense firewall, which has a LAN and WAN interface, in which the WAN interface is used to route my VM's to the internet. 

<img width="736" height="424" alt="image" src="https://github.com/user-attachments/assets/ab3bd631-5733-4577-8a3d-a47646aa2a04" />

IM1: Current lab architecture

## Steps/Procedure

### Step 0: Pre-lab Checklist/Setup

Before Starting This Sysmon lab/Project, I had to ensure that my lab environment was set up correctly to ensure that everything would operate smoothly and that Sysmon was correctly installed and configured. There was a few things I had to set up and check.

  1. **ENSURING WINDOWS VM WAS FUNCTIONAL.** This step was simple; my Windows VM has been working, so nothing extra to check here (IM2).
  2. **CONFIRMING THAT KALI VM CAN COMMUNICATE WITH WINDOWS VM.** To check this, I simply pinged the Windows IP address from Kali Linux, and it was successful (IM3). This is because later I will be doing some Nmap Scans against Windows to generate logs. 
  3. **DOWNLOADING SYSMON.** To download Sysmon onto my Windows VM, I first had to go to the official Microsoft Sysinternals Sysmon page to download the Sysmon zip file (IM4). Once I did that, I extracted the file to the C:\Sysmon to retrieve the contents
  4. **DOWNLOADING SYSMON CONFIGURATION FILE.** After downloading Sysmon, I needed to download its configuration file. Without it, Sysmon is as good as useless. To do this, I went to my browser and searched "SwiftOnSecurity Sysmon config GitHub" to find the configuration file (IM5). I downloaded the file, then saved it to C:\Sysmon.
  5. **INSTALLING SYSMON ON WINDOWS VM.** Now I needed to install Sysmon. to do this, I went into the command prompt (as an administrator), navigated to the sysmon folder, and used the command "Sysmon64.exe -accepteula -i sysmonconfig-export.xml." This command essentially accepts the license and installs Sysmon. The download was successful (IM6).
  6. **VERIFY SYSMON LOGS ARE VISIBLE.** After successfully downloading Sysmon, I needed to ensure that the logs were actually visible. To do this, I opened up Event Viewer (where most of this work was done) and followed this path: Applications and Services > Microsoft > Windows > Sysmon > Operational. Sure enough, the logs were there, verifying that Sysmon was correctly installed (IM7&8).
  7. **TAKE SNAPSHOT OF WINDOWS VM BEFORE STARTING LAB.** The last thing I had to do before starting was taking a snapshot of my Windows VM just in case anything crashed during the execution of this project (IM9).

<img width="834" height="465" alt="Screenshot 2026-03-23 145140" src="https://github.com/user-attachments/assets/9f9757de-4fa4-4b88-8e07-cfaa7b33330f" />

IM2: Windows 10 VM is fully functional

<img width="521" height="188" alt="Screenshot 2026-03-23 145302" src="https://github.com/user-attachments/assets/32e9c8ed-ca88-4630-8066-af113441b2e8" />

IM3: Pinging Windows VM from Kali Linux to ensure VMs can communicate 

<img width="598" height="347" alt="Screenshot 2026-03-23 151332" src="https://github.com/user-attachments/assets/1cd1b109-d997-4c7b-b301-978b46631711" />

IM4: Downloading Sysmon from the Sysinternals Sysmon page

<img width="600" height="350" alt="Screenshot 2026-03-23 151433" src="https://github.com/user-attachments/assets/942ebe31-c400-4b99-b66b-f9cbe2002cf9" />

IM5: Downloading Sysmon config file 

<img width="700" height="299" alt="Screenshot 2026-03-23 151719" src="https://github.com/user-attachments/assets/9fb689b7-239e-49b1-94ab-2489892f7a23" />

IM6: Successfully Installing Sysmon into Windows VM 

<img width="700" height="550" alt="Screenshot 2026-03-23 152250" src="https://github.com/user-attachments/assets/2a3600ea-6e23-45f6-ba68-fae2b2d7d5b9" />
<img width="700" height="450" alt="Screenshot 2026-03-23 152402" src="https://github.com/user-attachments/assets/fc6df08b-f1eb-455c-a051-88826731b2ff" />

IM7&8: Verifying Sysmon logs exist in Event Viewer

### Step 1: Understanding Windows & Sysmon Logs

  The first thing I wanted to do in this project, before generating logs, was familiarize myself with the different types of logs in Windows and specifically Sysmon. Here is how I did that
    1. **NATIVE WINDOWS LOGS.** The native Windows logs 













