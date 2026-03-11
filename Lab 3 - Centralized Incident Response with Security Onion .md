# Centralized Incident Response with Security Onion - Lab 3 #
This is a walkthrough and an explanation on executing lateral movement and persistence attacks, and investigating these attacks with Security Onion.

## Step 1: Generate Telemetry (The Attack) ##

 **Rerun the attacks mentioned in the last labs from the Kali VM to make sure that there are fresh logs in the Security Onion.**
 The attacks are as follows:
   - `crackmapexec smb <DC_IP> -u Administrator -p 'Passw0rd!' -x 'whoami /all'`
   - `impacket-psexec lab.local/Administrator:'Passw0rd!'@<DC_IP>`
       - Inside the shell run `sc create "Maintenance" binPath= "cmd.exe /c echo 'Backdoor > C:\temp.exe"`
    
### Navigate to the Security Onion ###

Security Onion &rarr; Alerts &rarr; Options &rarr; Enable advanced interface features 

## Step 2: The Investigation (Centralized Console)

### Task A: Checking for High-Level Alerts ###

Lab Specific Findings:
  -  Filter for the specific event using `event_data.host.name: "DC VM NAME"` and `destination.ip:<Your_DC_IP>`
       - In my case this would be `event_data.host.name: DC-sheehare23` and `destination.ip:192.168.1.70`
  - This surfaced Sigma alerts such as *HackTool - CrackMapExec Execution Patterns* and *Suspicious New Service Creation*

Key Data Captured in the Log:
- event_data.host.hostname: Identifies the target machine
- event_data.process.command_line: Shows the exact command executed.
- event_data.process.name: Indicates the file or program that was launched
- event_data.winlog.user.name: Indicates the account that executed the child process
- event_data.winlog.event_data.ParentUser: Indicates the account that exectued the parent process

*In my case these fields are as follows:*

`impacket-psexec`
<img width="975" height="48" alt="image" src="https://github.com/user-attachments/assets/e339ed1f-1c3f-4f03-ac37-7ac5200fb4e0" />
<img width="969" height="58" alt="image" src="https://github.com/user-attachments/assets/54094d2a-a9ac-42dc-a2bf-ee098da0ca9c" />
<img width="975" height="43" alt="image" src="https://github.com/user-attachments/assets/997b8345-56cb-4d4a-bc60-950ad265369d" />
<img width="975" height="56" alt="image" src="https://github.com/user-attachments/assets/cace73a8-2b76-48e9-b0c9-55ba057ec278" />
<img width="975" height="60" alt="image" src="https://github.com/user-attachments/assets/30ff6087-17c0-4bc7-a679-13b908879bfe" />
<img width="975" height="43" alt="image" src="https://github.com/user-attachments/assets/1492e3c6-23fa-43fa-9358-9961a4c6d30c" />
<img width="975" height="53" alt="image" src="https://github.com/user-attachments/assets/c01dfcf6-8157-4bb7-9343-b3a4f1b63b13" />

`crackmapexec`
<img width="975" height="48" alt="image" src="https://github.com/user-attachments/assets/0a49f48f-27cc-4851-a637-fe06be3d9074" />
<img width="969" height="58" alt="image" src="https://github.com/user-attachments/assets/86524589-1471-465f-a48e-c2ef73ab38a4" />
<img width="975" height="43" alt="image" src="https://github.com/user-attachments/assets/f2a7b5b8-9eba-4ca0-83c6-1728e2889629" />
<img width="975" height="56" alt="image" src="https://github.com/user-attachments/assets/1d5d6ada-10ee-4365-85a0-6b1ff9b40e16" />
<img width="975" height="60" alt="image" src="https://github.com/user-attachments/assets/fcf97189-7ade-490f-a385-94833477b2e2" />
<img width="975" height="43" alt="image" src="https://github.com/user-attachments/assets/a19be2ad-9d4f-4233-8f0f-eafc2605efe6" />
<img width="975" height="53" alt="image" src="https://github.com/user-attachments/assets/74c725f2-630c-4b70-a1b6-60b6b4917fd2" />

### Task B: Deep Dive with Kibana Discover ###

Security Onion &rarr; Kibana &rarr; Discover &rarr; Data View (ensure index Patterns is set to "logs-*") &rarr; Make sure time range is correct

**Overview: Kibana allows you to see exactly what happened by looking at the raw data. It is best for deep forensics such as granular filtering, visualization, and raw log parsing.**

Investigating Lateral Movement(whoami):

- Filter for specific events using `process.name: "whoami.exe"` and `host.name: "Your_DC_NAME"`
  - In my specific case my DC name would be `dc-sheehare23`

____The resulting log should appear similar to this:___
<img width="1275" height="802" alt="image" src="https://github.com/user-attachments/assets/21817620-643d-46ac-afeb-7babd40294d9" />

Investigating the Malicious Service:
- Filter for specific events using `process.name: "sc.exe"` and `winlog.event_id:1`

___The resulting log should appear similar to this:___
<img width="1270" height="464" alt="image" src="https://github.com/user-attachments/assets/28712896-a883-4607-adc9-a59cda791c43" />

- In this case the columns for user.name, process.name, and process.command_line were added on the left side of the screen (Available Fields sidebar) to create columns for the logs, making the data easier to read.
___

## Interface Comparison: Hunt vs. Kibana 

Hunt: 
- Best for quick metadata search and seeing pre-built detections. It is better for a professional to see if their machine or network is under attack because it labels events based on importance

Kibana: 
- Best for deep forensics. It is better for when you know an attack has occurred, and you need to search for exactly what the attacker did while they were in.
___

## Forensic Correlation Table 

|Attacker tool|Security Onion Alert|Kibana Search Field|Key Evidence(Command Line)|
|----|----|----|----|
|smbexec|HackTool - CrackMapExec Execution Patterns|process.name|whoami /all|
|psexec /sc|Suspicious New Service Creation|process.command_line|sc create "Maintenance" binPath="cmd.exe /c echo 'Backdoor'>C:\temp.exe"|









