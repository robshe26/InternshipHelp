# Detecting Persistence (Malicious Service Creation) - Lab 2 #
This is walkthrough and an explanation on a simple attack, creating a system service to maintian access after a system reboot

## Step 1: The Attack (From Kali VM)
On the Kali VM use "psexec" to run a remote command on the DC: 

1. Run `impacket-psexec lab.local/Administrator:'Passw0rd!'@<DC_IP>`
2. Inside the remote shell run `sc create "Maintenance" binPath= "cmd.exe /c echo 'Backdoor' > C:\temp.exe"`

*Upon completion of the attack you should get a similar result as the following:*
<img width="1203" height="711" alt="image" src="https://github.com/user-attachments/assets/fe965ad4-bf9a-4066-815f-ed501048596c" />

### Command Breakdown: ```psexec``` ###

impacket-psexec
- A Python based tool used to execute commands and gain an interactive shell on remote Windows systems from a Linux machine
- Mimics the offical Windows psexec utility.
- It works by using the SMB protocol to upload a randomized executable to the hidden ADMIN$ share, registers it as a Windows service, and pipes input/output back to the attacker.

lab.local
- Specifies the target Active Directory domain
- Ensures the authentication request is routed to the domain's authority rather than attempting to authentice against local machine accounts.

Administrator
- Targets the user account holding the higest level of privilege.
- Grants unrestricted accress to the AD environment.

'Passw0rd!'
- The plaintext secret used to validate autehntication via the NTLM Challenge-Response process.

@ <DC_IP>
- The network routing desstination.

### Payload Execution Breakdown: Establishing Persistence

sc create
- `sc` is a native Windows administrative tool acting as the command line interface for the Service Control Manager.
- Combined with create, it acts as an RPC cleint instructing the SCM to create a new service object.

"Maintenance"
- Sets the display name registry value for the newly created service.
- Designed to blend in as a normal patch or update to bypass security monitoring.

binPath=
- Fills in the image path registry value, defininf what executable runs when the service starts.

cmd.exe /c
- Opens a Windows command shell
- The `/c` flag executes the following string and immediately terminates the process thread.

echo 'Backdoor' > C:\temp.exe
- Outputs the string "Backdoor" and uses the `>` operator to redirect that output to the disk.
- Allocates hard drive space and writes the file to `C:\temp.exe`.
- Etablishes persistence, allowing the attacker to maintian acess to the Domain Controller even after a system reboot. In this case it is "Backdoor" but in other instances it can be replaced with an actual payload.

***

## Step 2: Evidence Investigation (On the DC VM) ##

Use the Event Viewer (eventvwr.msc) on the DC VM to find all evidence 
- Windows Security Log: Windows Logs -> Security
- Windows Sysmon Log: Windows Log: Windows Logs -> System
- Sysmon Operation Log: Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational

### Event ID 7045 (System Log): Service Installed ###

- The Attack Correlation: This log is tied directly to the persistence phase of the attack, specifically triggered by the `sc create "Maintenance"` portion of the commmand.
- Event ID 7045 is a Windows log that is generated whenever a new service is installed on the system. It is a primary indicator for secuirty professionals checking if an attacker has established a persistence mechanism.

Key Data Captured in the Log:
- System Changes: Documents major configuration changes related to the Windows Service Control Manager (SCM).
- Service Name: The exact name assigned to the newly installed service.
- Service File Name (Image Path): The full path to the executable or command that the service will run when started.

Lab Specific Findings:
- Service Name: The log shows the service was installed as "Maintenance".
- Service File Name: The log recorded `cmd.exe /c echo 'Backdoor' > C:\temp.exe`.

___The corresponding log should appear similar to this:___
<img width="975" height="600" alt="image" src="https://github.com/user-attachments/assets/9fceea9a-a8ae-4fb6-b546-8030001b04ec" />

***

### Event ID 1 (Sysmon Log): Process Creation for Persistence) ###

- The Attack Correlation: This log captures the exact sequence of the process executions used to estbalish the malicious persistence mechanism on the host
- Event ID 1 Overview: The Sysmon log records hihgly detialed process creation events, providing insight into how an attacker is interacting with the system

Lab Specific Findings:
- Target Host: The amlicious activity was logged on the host: `DC-sheehare23.lab.local`.
- Parent/Child: The parent process is `cmd.exe` whihc spawned the child process `sc.exe`
- Service Creation: Confims the use of `sc.exe` to create the fake "Maintenance" service. It also shows the malicious `binPath` which used a commmand shell to write a backdoor string into a new file, guaranteeing the attacker's access survives a system reboot.
- Privilege Level: These processes are running under the SYSTEM account, which means that the attacker has achieved maximum administrative control over a machine. 

___The corresponding log should appear similar to this:___
<img width="975" height="571" alt="image" src="https://github.com/user-attachments/assets/c3622d59-528a-4e5f-abbf-93849e26dc42" />

***

### Event ID 11 (Sysmon Log): File Create ###

- Event ID 11: Overview: This is a log that monitors and records file creation operations. This is useful for detecting when attackers download tools, extract malware, or establish persistance mechanisms

**\*This log was in the lab instructions as a Sysmon Log to find however it does not technically exist\***
  - This is due to the fact that while the command was ran, it was not executed. This means that without the execution no file will be created and no corresponding log will come as a result 

However, even without the corresponding log, you can see that a service/process was created in services as shown below 
<img width="975" height="664" alt="image" src="https://github.com/user-attachments/assets/0e0d61f6-d3c3-41b1-98a5-58e327e7b163" />
This means that the command did run correctly but did not produce a log as the service was not started 

***

### Evidence Correlation ###

|Attack Step|Command Arguement|Corresponding Log|Purpose|
|:----:|:----:|:----:|:----:|
| 1. Authentication | *sc create "maintenance"* | Event ID 7045 | New Service Creation |
| 2. Initial Execution | *sc.exe after smd.exe* | Event ID 1 | Process of sc.exe after spawn of cmd.exe |
| 3. Payload Execution | *echo 'backdoor'>C:\temp.exe* | Event ID 11 | Creation of the file |






