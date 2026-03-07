# Identifying Lateral Movement (Whoami Discovery) - Lab 1 #
This is walkthrough and an explanation on a simple attack, identifying privileges on another device 

## Step 1: The Attack (From Kali VM)
On the Kali VM use "crackmapexec" to run a remote command on the DC: 

*crackmapexec smb <DC_IP> -u Administrator -p 'Passw0rd!' -x 'whoami /all'*
___
Command Breakdown: CrackMapExec Lateral Movement 
- crackmapexec
  - An overarching post-exploitation toolkit used to automate network attacks.
  - Evaluates Active Directory security by authenticating as an admin to display identity and network permissions
- smb
  - The module selector that specifies the protocol for communication.
  - Directs CME to target the Server Message Block (SMB) service (TCP port 445) to look for exposed administrative shares
- <DC_IP>
  - For this specific example, the IP address of the targeted machine is 192.168.1.70
  - The target argument defining the specific host, CIDR range, or IP list to be accessed.
  - Aimed at Domain Controllers because they act as the central authentication authority and house the Active Directory database.
- -u administrator
  - Defines the authentication identity (username) for the attack
  - Without a specified domain, CME attempts a local login; however, a DC will automatically interpret this as the Domain Administrator
- -p 'Passw0rd!'
  - Provides the secret/password required to validate the claimed identity.
  - Uses the NTLM Challenge-Response process to authenticate rather than sending the plaintext password directly.
  - Can be swapped with a password file to perform "password spraying" or brute-force attacks.
- -x 'whoami /all'
  - Specifies the remote command to execute immediately after successful authentication.
  - Uses Windows management features to silently run the command in the background.
  - The /all flag prints the Access Token, revealing SIDs, group memberships, and high level privileges.

___Upon completion of the attack you should get a similar result as the following:___
<img width="1200" height="803" alt="image" src="https://github.com/user-attachments/assets/b8e48f35-c725-4fb1-88b4-1644e5456732" />

## Step 2: Evidence Investigation (On the DC VM) ##


Using the Event Viewer (eventvwr.msc) on the DC VM to find all evidence 
- Windows Security Log: Windows Logs -> Security
- Windows Sysmon Log: Windows Log: Windows Logs -> System
- Sysmon Operation Log: Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational

#### Event ID 4624 (Successful Logon) // (Source Kali IP) - 192.168.1.69 ####

- The Attack Correlation: This log is triggered by the authentication phase of the attack, or the * -u Administrator -p 'Passw0rd!'* portion of the CME command.
- Event ID 4624 Overview: A Windows Security log generated for every successful authentication attempt, whether by a physical user, background service, or remote connection.

Key Data Captured in the Log: 
- Account Information: The specific username and domain of the logged in account
- Network Information: The source IP address and originating workstation name
- Authentication Details: The specific protocol package used
- Logon Type: A numerical code defining how the system was accessed:
  - Type 2: Physical login using the computer's keyboard and screen.
  - Type 3: Accessed remotely from across the network
  - Type 5: A background service started by the Windows Service Control Manager

Lab Specific Findings:
- Source Identified: The login originated from the Kali IP address (192.168.1.69) over Source port 49490
- Authentication Package: Logs show NtLmSsp and NTLM V2, verifying CME's use of the NTLM Challenge-Response protocol over SMB
- Logon Type Recorded: Type 3; confirming the attacker authenticated over the network rather than physically at the machine

___The corresponding log should appear similar to this:___
<img width="1174" height="705" alt="image" src="https://github.com/user-attachments/assets/c681d926-4de1-4c3c-bea2-5408664eef2c" />

#### Event ID 4688 (New Process Creation) ####

***
**Before Event ID 4688 appears: Windows does not audit all process creations by defualt**
- You may have to enable process creation
  - This can be done by running `auditpol /set /subcategory:"Process Creation" /success:enable /failure:disable`
***

- The Attack Correlation: This log is triggered when the compromised account begins a new process on the system, or the *-x* flag in the CME command
- Event ID 4688 Overview: A Windows Security log generated every time a new program or process is started. It is crucial for security professionals, as every piece of software must start as a process of execution 

Key Data Captured in the Log: 
- Creator: Identifies the exact user account, domain, and SID that requested the process to start.
- Launched Commands: List the full file path of the executed program
- Parent Process: Identifies the parent program that spawned the new process, providing critical context as to why the program was started.

Lab Specific Findings:
- Creator Subject Identified: The log shows the account name as Administrator within the Lab domain, revealing the exact account used to execute the command.
- Execution Chain: The log captures ```whoami.exe``` running immediately following the creator process ```cmd.exe```. This reveals CME's exact behavior, creating a command prompt to execute the ```whoami``` payload.

**Standard system logs like 4688 provide a less detailed, higher-level view compared to Sysmon logs**

___The corresponding log should appear similar to this:___
<img width="975" height="465" alt="image" src="https://github.com/user-attachments/assets/a3c5579d-ca98-44e4-8ed0-bfea15a356ba" />

#### Event ID 1 (New Process Creation) ####

- The Attack Correlation: This log is the direct result of the commands passed after the *-x* argument in CME
- Sysmon Overview: An advanced logging tool that provides deeper visibility into system activity. It contains all the baseline information of an Event ID 4688, but with significantly more detail

Key Data Captured by Sysmon:
- Image: The full file path of the launched executable.
- Command Line: The exact command and all arguments passed to the executable.
- Hashes: The cryptographic hashes of the executed file (useful for threat intelligence and malware identification).
- Execution Chain: The exact file path and command line of the program that spawned the new process.

Parent versus Child Processes:
- Parent Process: The program that initiates another program. Identifying this reveals how the attacker executed their code.
- Child Process: The newly created program that carries out the specific commands

Lab Specific Findings:
- Parent Image: ```C:\Windows\System32\cmd.exe```. This indicates that a command prompt was silently opened in the background to facilitate the next step.
- Child Image: ```C:\Windows\System32\whoami.exe```. Once the hidden command prompt was opened by the parent, it spawned this child process to run the ```whoami /all``` command.

___The corresponding log should appear similar to this:___
<img width="975" height="567" alt="image" src="https://github.com/user-attachments/assets/320d8d0e-7464-41ca-9f8d-7ebaccf983e5" />


