#### UAC Bypass

1. Check UAC Config

   ```powershell
   reg query HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\
   ```

   - Check the value of the key EnableLUA, if it's 1 then UAC is activated, if its 0 or it doesn't exist, then UAC is inactive.
   - Check the value of the key **`ConsentPromptBehaviorAdmin`**in the same entry of the registry as before (info from [here](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gpsb/341747f5-6b5d-4d30-85fc-fa1cc04038d4)):
     - If **`0`** then, UAC won't prompt (like **disabled**)
     - If **`1`** the admin is **asked for username and password** to execute the binary with high rights (on Secure Desktop)
     - If **`2`** (**Always notify me**) UAC will always ask for confirmation to the administrator when he tries to execute something with high privileges (on Secure Desktop)
     - If **`3`** like `1` but not necessary on Secure Desktop
     - If **`4`** like `2` but not necessary on Secure Desktop
     - if **`5`**(**default**) it will ask the administrator to confirm to run non Windows binaries with high privileges
   - Check the value of **`LocalAccountTokenFilterPolicy`**  If the value is **`0`**, then, only the **RID 500** user (**built-in Administrator**) is able to perform **admin tasks without UAC**, and if its `1`, **all accounts inside "Administrators"** group can do them.
   - Check the value of the key **`FilterAdministratorToken`** If `**0**`(default), the **built-in Administrator account can** do remote administration tasks and if `**1**` the built-in account Administrator **cannot** do remote administration tasks, unless `LocalAccountTokenFilterPolicy` is set to `1`.

the UAC is activated, your process is running in a medium integrity context, and your user belongs to the administrators group