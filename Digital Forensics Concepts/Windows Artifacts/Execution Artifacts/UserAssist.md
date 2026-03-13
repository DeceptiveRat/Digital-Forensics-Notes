## 1. UserAssist
- tracks usage of executable files and applications launched by user

![[UserAssist artifact.png]]

## 2. location
`Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}`
- GUID correlates to type of tracked user activity:<sup>[1]</sup>
	- {CEBFF5CD‑ACE2‑4F4F‑9178‑9926F41749EA}: programs run directly via .exe files
	- {F4E57C4B‑2036‑45F0‑A9AB‑443BCFE33D9F}: initiated via shortcuts

## 3. data
- each entry is ROT13-encoded
- data includes:
	- application path or identifier
	- run count
	- last execution timestamp
	- focus time: how long app remained in focus
	- focus count: how often window was brought to foreground

## 4. focus time
- cumulative amount of time in miliseconds an application window was active in foreground
- value is not broken down into session-level detail

## 5. limitations
- lcaks command-line or background process visibility
- user settings can disable or clear history

## reference
[1] (2025/09/18) UserAssist Forensic Artifacts: What they are and how to use them, 2026/03/13, UserAssist Forensic Artifacts: What they are and how to use them
