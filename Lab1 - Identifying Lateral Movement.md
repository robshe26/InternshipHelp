# Identifying Lateral Movement (Whoami Discovery) - Lab 1 #
This is walkthorugh on how to run a simple attack, identifying privilages on another device 

## Step 1: The Attack (From Kali VM)
On the Kali VM use "crackmapexec" to run a remote command on the DC: 

*crackmapexec smb <DC_IP> -u Administrator -p 'Passw0rd!' -x 'whoami /all"*
___
Command Breakdown: CrackMapExec Lateral Movement 
- crackmapexec
  - An overarching post-exploitation toolkit used to automate network attacks.
  - Evaulates Active Directory security by authenticating as an admin to display identity and network permissions
- smb
  - The module selector that specifies the protol for communication.
  - Driects CME to target the Server Message Block (SMB) service (TCP port 445) to look for exposed administrative shares
- <DC_IP>
  - For this specific example, the IP address of the targeted machine is 192.168.1.70
  - The target argument defining the specific host, CIDR range, or IP list to be accessed.
  - Aimed at Domain Controllers beccuase they act as the central autenticatopn authority and house the Active Directory      database.
- -u administrator
  - Defines the authentication identity (username) for the attack
  - Without a specified domain, CME attempts a local login; however, a DC will automatically interpret this as the Domain Administrator
- -p 'Passw0rd!
  - Provides the secret/password required to validate the claimed identity.
  - Uses the NTLM Challenge-Response process to authenticate rahter than sending the plaintext password directly.
  - Can be swapped with a password file to perform "password spraying" or brute-force attakcs.
- -x 'whoami /all'
  - Specifies the remote command to execute immediately after successful authentication.
  - Uses Windows management features to silently run the command in the background.
  - The /all flag prints the Access Token, reveailing SIDs, gorup memberships, and high level privileges.

___Upon Completion of the attack it should appear like this:___
<img width="1200" height="803" alt="image" src="https://github.com/user-attachments/assets/b8e48f35-c725-4fb1-88b4-1644e5456732" />



