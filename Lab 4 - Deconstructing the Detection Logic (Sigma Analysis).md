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
  - The Logic:
  - The Why:


