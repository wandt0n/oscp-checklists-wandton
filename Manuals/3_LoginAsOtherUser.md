
# Windows
- If we have access to the GUI -> `RunAs /user:$user cmd`
- User is in group _Remote Desktop Users_ -> RDP
- User is in group _Remote Management Users_ -> WinRM (Better use [evil-winrm](https://github.com/Hackplayers/evil-winrm) from non-interactive shells)
- User has _Log on as a batch job_ right -> Schedule task to execute a program of our choice as this user
- User has active session -> Use PsExec (SysInternals)