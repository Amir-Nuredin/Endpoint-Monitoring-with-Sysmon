# Endpoint Monitoring with Sysmon: Windows Event Log Analysis & Threat Detection

## Objective

The focus of this lab is to understand how to implement and analyze Windows Event Logs and Sysmon to mimic endpoint detection and response. This project creates hands-on experience with identifying malicious activities using log analysis. This lab concludes with identifying reconnaissance activities such as Nmap scans. This is an extension of basic blue team concepts that are applicable in real-world security.

### Skills Learned 

This project has provided hands-on experience with the following security solutions/skills:

-  Windows Event Log analysis using Event Viewer
-  Sysmon installation, configuration, and log monitoring
-  Process creation analysis (Sysmon Event ID 1 / Windows Event ID 4688)
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
  3. **DOWNLOADING SYSMON.** To download Sysmon onto my Windows VM, I first had to go to the official Microsoft Sysinternals Sysmon page to download the Sysmon zip file (IM4). Once I did that, I extracted the file to the C:\Sysmon folder to retrieve the contents
  4. **DOWNLOADING SYSMON CONFIGURATION FILE.** After downloading Sysmon, I needed to download its configuration file. Without it, Sysmon is as good as useless. To do this, I went to my browser and searched "SwiftOnSecurity Sysmon config GitHub" to find the configuration file (IM5). I downloaded the file, then saved it to the C:\Sysmon folder.
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

<img width="313" height="211" alt="Screenshot 2026-03-27 182609" src="https://github.com/user-attachments/assets/c3daa658-5798-4240-aa89-7a20f977d48e" />

IM9: Snapshot of Windows VM before starting lab




### Part 1: Understanding Windows & Sysmon Logs

  The first thing I wanted to do in this project, before generating logs, was familiarize myself with the different types of logs in Windows and specifically Sysmon. Here is how I did that. 
    
  1. **NATIVE WINDOWS LOGS.** The native Windows logs are the main logs for Windows for Event Viewer. I viewed the different types of Windows logs and the Specific logs in each of them. The first I viewed was the security log (IM10). The security log in windows contains things like logins, failed logins, and special logon's among other things. Next I looked at the system log (IM11). This Windows log mainly contains different OS events like shutdowns, reboots, and driver issues. The last log category I looked at was the application log (IM12). This category of Windows logs shows different events related to running programs, such as app crashes, app installs, and errors. These are the native Windows logs that I looked at.
  2. **PRACTICE WITH FILTERING EVENT IDs.** Once I looked at the native Windows logs, I wanted to filter specific event IDs to look into one kind of event more. To do this, I went to any of the native Windows log catergories (for this example I chose security) and on the right side of the screen chose "Filter Current log." After that in the event ID box I chose the specific event I wanted to filter. For this example I chose 4624 (ID for successfull logins) and filtered it out. Once I filtered it out, all the logs were only with the Event ID of 4624 (IM13&14).
  3. **LOCATING DIFFERENT TYPES OF SYSMON LOGS.** Once I got familier with the native Windows logs, I then went to the Sysmon logs in Event Viewer to see the different kinds of logs that were in there and confirm that the logs existed. There were two main types of logs I was looking for. The first was Event ID 1, which is the ID for process creation (for example opening up command prompt or any application like that). For any of these logs, I could scroll through until I find it or just filter the Event ID. Luckily I found ID 1 right away, confirming the log exists (IM15). The second kind of log was Event ID 3. This ID is for any kind of network connection (pinging something or browsing a website). I did the same process as ID 1 and found events for ID 3.
  4. **IDENTIFY DIFFERENT PROPERTIES OF LOGS.** The last thing I did for this section of the project was analyzing the properites of a log. This will be done throughout the project as it is essential to know the inner details of what is going on, espeically in the real world when doing threat detection and log aggregation. to do this, I simpliy open any log in Sysmon (ID 1 in this case) and went to the "general" section to inspect it further. for this case, I looked for the hostname (computer), the user, and the Process GUID (used for tracking activity) (IM16).

<img width="824" height="454" alt="Screenshot 2026-03-24 094148" src="https://github.com/user-attachments/assets/3efa73f1-5c34-49ba-975e-f53fb5dd26c6" />

IM10: Security section in native Windows logs

<img width="824" height="452" alt="Screenshot 2026-03-24 094744" src="https://github.com/user-attachments/assets/72d18e4d-f950-4410-80b5-4e8113970816" />

IM11: System section in native Windows logs

<img width="827" height="457" alt="Screenshot 2026-03-24 094859" src="https://github.com/user-attachments/assets/81dd622d-a339-4966-bb2f-17e3714b86ba" />

IM12: Application section in native Windows logs

<img width="778" height="545" alt="Screenshot 2026-03-24 095218" src="https://github.com/user-attachments/assets/a8cd5f0a-a634-462b-b2c8-165cbcf6a290" />
<img width="623" height="377" alt="Screenshot 2026-03-24 095348" src="https://github.com/user-attachments/assets/090fdd2d-cb24-48e7-8a86-fc242fbc91ca" />

IM13&14: Filtering Event ID 4624 (successfull logins)

<img width="826" height="204" alt="Screenshot 2026-03-24 100151" src="https://github.com/user-attachments/assets/c91ff54a-8f8c-4d3f-a690-eca8bd71f78a" />

IM15: Locating log with Event ID 1

<img width="624" height="500" alt="Screenshot 2026-03-24 100630" src="https://github.com/user-attachments/assets/e1b28e8a-d91a-4b59-a3a8-7ad4dca74de5" />

IM16: General Properties of ID 1 log

### Part 2: Generate and Analyze Activity in Logs

  Now that I got familier with Sysmon and its properties, I am now going to generate different types of activity within my Windows and Kali Linux VM to generate logs and inspect what properties they have within them. 

  1. **GENERATING PROCESS CREATION LOGS.** The first kind of log I generated was Event ID 1, which is process creation. There are many was to do this, but a simple way to do this is to just run a few commands in the command prompt. For my example, I entered the commands notepad, calc, whoami, and ipconfig to generate log activity (IM17). After this I went back into the sysmon logs to see if any activity was generated by filtering for ID 1. Sure enough, there was log activity for those commands. The two I will show are ipconfig (IM18) and whoami (IM19). Some properties I was looking for were the image path, command line to show which command was used, and the user field.
  2. **GENERATING NETWORK CONNECTION LOGS.** The next kind of network log I wanted to generate was Event ID 3, which were network connection logs. I tried pinging my Kali Linux VM and Google, but those didn't work, so I tried browsing the web on the Windows VM. Once I did that, I went into the Sysmon logs and filtered out ID 3 to find the specific log. Once I found it, I went into the details to further analyze the log. In there we can see things the like the source IP (Wndows VM IP), destination IP (IP for browsed website), and the destination port (443, indicating a HTTPS connection to a secure website) (IM20).
  3. **DETECTING A FAILED LOGIN.** The final log I tried to generate was a log for a failed login, which is Event ID 4625. To do this, I locked my VM and purposely typed my password incorrectly three times to generate a few logs (IM21). Once I logged back in I went to Windows logs and then security to filter out ID 4625. In the log we can see the account name for my VM and the logon type, which was 2 (local login) (IM22).

<img width="539" height="300" alt="Screenshot 2026-03-25 182640" src="https://github.com/user-attachments/assets/70562392-5dd2-449f-a74a-29b45286df93" />

IM17: Generating Proccess Creation logs with basic commands 

<img width="630" height="500" alt="Screenshot 2026-03-25 183054" src="https://github.com/user-attachments/assets/9f3e4a2a-e483-4434-9619-9eb5ed508b33" />
<img width="632" height="500" alt="Screenshot 2026-03-25 183255" src="https://github.com/user-attachments/assets/a11257e2-55ec-42f9-8b9b-4784dbb147cb" />

IM18&19: Log Output and Details for ipconfig and whoami (ID 1)

<img width="832" height="550" alt="Screenshot 2026-03-25 185242" src="https://github.com/user-attachments/assets/e9df9278-94cb-4a89-9ce4-484c7d3c29cf" />

IM20: Network Connection through HTTPS web browse log details (ID 3)

<img width="425" height="400" alt="Screenshot 2026-03-25 185613" src="https://github.com/user-attachments/assets/2e96d831-3933-4f5a-b968-e74638561ab3" />

IM21: Failed login attempt to generate logs

<img width="630" height="500" alt="Screenshot 2026-03-25 185742" src="https://github.com/user-attachments/assets/d0ebdb77-7b85-4a96-99dc-07ff09bac8d3" />

IM22: Failed login logs and Details

### Part 3: Simulated Attack/Nmap Scan

  The last part of this lab was to run a simulated attack scenario by running and Nmap scan from my Kali linux VM against my Windows VM and detect in the Sysmon logs. 

  1.  **RUN NMAP SCAN.** To PERFORM THIS, I first ran the Nmap scan "nmap -sS -Pn 10.10.10.102 to perform a stealth scan without host discovery on my Windows VM (IM23).
  2.  **IDENTIFY LOGS FOR SCAN.** After the can was complete, I went to the Sysmon logs to see if it got reported. The information I was looking for in the log included the source IP (Kali Linux VM) and mutiple connection attempts since I scanned multiple times
  3.  **CORRELATE TIMESTAMPS.** To ensure that the time I performed the scan in Nmap and the time it was logged were correlated, I looked at the time the scan was done (first line of scan results) and what time it had said in the log, and fortunately they lined up.

## Conclusion

  This lab was a success in demonstrating the implementation and utilization of Windows Event Logging and Sysmon for monitoring and analyzing activities within a virtualized environment. Through the simulation of real-world scenarios for system activities, the lab was able to provide valuable insights into the generation of security events at the host level.

  During the implementation of the lab, a number of challenges were overcome, such as setting up Sysmon for the generation of event logs, utilizing Event Viewer for log analysis, troubleshooting missing log information, and validating network communications for virtual machines for the generation of event logs.

  This project has solidified the link between foundational cybersecurity concepts and their practical application within a real-world environment for endpoint monitoring and analysis. It has also emphasized the importance of analyzing process execution, network connection analysis, authentication analysis, and log analysis for suspicious activities, which are critical for a SOC environment.

  Some possible extensions for the current lab implementation could include integrating a SIEM solution for log analysis, developing a set of custom rules for analysis, creating a set of simulated attacks for analysis, and implementing a set of automated alerts for enhanced response capabilities.

















