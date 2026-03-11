# Deconstructing the Detection Logic (Sigma Analysis) - Lab 4

- This lab focuses on the transition from seeing the alerts to understanding the specific YAML logic whithin Secuirty Onion that flags malicious activity

## Step 1: Finding the Sigma Rules in Secuirty Onion 

Navigate to the Detections Interface
 - Open the Detections Tab: On the left side of the sidebar of the Secuirty Onion, click on Detections
 - Filter for Sigma: Ensure that the rule type is filtered to Sigma using the filter to find "Detection Type - Sigma".
 - Search by Alert Name: Use the search bar to find the specific rules identified in Lab 3 (CrackMapExec, Impacket, sc.exe)
   - The fetech limit may need to be increased to 5000 to make sure the rule appears
 - Click on the rule name and select View Soruce to see the YAML definition

___The YAML code should appear similar to this:___
<img width="975" height="428" alt="image" src="https://github.com/user-attachments/assets/9a23b32b-44c4-4faa-afd1-91e3a22c5ce1" />
___

## Step 2: Analysis and Comparison

### Part A: Lateral Movement (smbexec)

Analysis of CrackMapExec/Impacket Alerts:
- The rule targets the command_line field using the contains modifier. In Sigma, when a list of values follows a field without specific hyphens for AND logic, it typically defualts to OR logic, meanin if any of the specific strings appear, the alerts trigger
   - The Logic: The rule searches for specific executions pattersn, such as putput redirection used by tools like `smbexec`
   - The Why: The `whoami /all` command triggered an alter not becuase of the `whoami` payload, but becuase of the parent process and the syntax. Specifically, the Sigma rule flags patterns like `cmd.exe /Q /c * 1>\\\\*\\*\\* 2>&1`, which indicates a tool sending the command output back over SMB.
 
### Part B: Persistance (sc.exe)

Analysis of Suspicious Service Creation:
- This detection is often triggered by a combination of conditions, meaning it uses AND logic.
  - The Logic: The event is flagged when the logic matches a selection for service creation AND a susp_binpath
  - The Why: The command met the criteria becuase the binPath attempted to silently execture a command prompt, which is a common pattern for establishing backdoors rather than performing standard maintenance. The rule caught the behavior rather than the specific service name "Maintenance"

___Examples of these YAML files:___

Lateral Movement:

<img width="975" height="428" alt="image" src="https://github.com/user-attachments/assets/0e5dcd9d-f655-4dea-978d-aa66accf84a1" />

Persistance:

<img width="513" height="867" alt="image" src="https://github.com/user-attachments/assets/0feabf67-ac8e-4673-b91e-2ac0c623553d" />
___

## Step 3: Deliverables and Forensic Correlation

### Logic Correlation Table 

|Alert Name|Sigma 'Selection' Criteria (YAML)|Matching Value From Lab3 Log|
|----|----|---|
|**Impacket Lateral Movement**|'CommandLine|cmd.exe/Q/c*1>\\\\*\\*\\*2>&1|
|**Suspicious Service Creation**|1 of selection* AND susp_binpath|sc create "Maintenance" binPath="cmd.exe /c echo 'Backdoor' > C:\temp.exe"

### Understanding False Positive 
- A False Positive occurs when a secuirty rule incorrectly flags legitimate activity as malicious
  - How Sigma avoids them: To distinguish a legitimate administrator using `sc create` from an attacker, the rule doesn't just look for the command itself. It correlates the command with a suspicious payload. A standard backup service would point to a known, legitimate executable, faling the `susp_binpath` portion of the logic and thus not triggering the alert.

___This is an exmaple screenshot of the Detections interface showing the YAML source code for the `Hack Tool - Potential Impacket` rule:___
<img width="1002" height="528" alt="image" src="https://github.com/user-attachments/assets/76d3c6ac-c6d9-4654-9d89-96e629fde1d2" />

