# iSCSI Reconnect Automation on Windows (Project Log)

## Objective

Set up a Windows system to **automatically reconnect to an iSCSI target** (hosted on a Synology NAS) after reboot — without any manual input — as part of hands-on infrastructure hardening and automation experience.
This was implemented as part of my cybersecurity infrastructure practice to ensure system resilience and hands-free network storage recovery.

---

## Background

While storing Steam games on my Synology DS1815+ NAS via iSCSI, I noticed:

- After rebooting Windows, the iSCSI Initiator would display **"Reconnecting..."** indefinitely
- The mapped iSCSI volume wouldn't appear in File Explorer
- I had to **manually disconnect the session** in iSCSI Initiator → then **reconnect** for the drive to show up

---

## Troubleshooting

### Initial Setup Steps (That Didn't Fully Work):
- Set iSCSI connection as **Favorite Target** with **automatic restore**
- Verified NAS IP was **static**
- Enabled **delayed start** for iSCSI service (failed with Error 87)
- Confirmed no firewall blocks (port 3260 open)
- Disabled **jumbo frames** after they broke connectivity

### Root Cause
Windows was holding onto a **stale iSCSI session** across reboots. This session caused the iSCSI Initiator to hang in a "Reconnecting..." state — even though the target was reachable.

---

## Final Working Solution

### 1. Created a PowerShell Script to:
- Check for stale iSCSI sessions
- Disconnect them if found
- Reconnect the iSCSI target cleanly
- Log only **errors** to a file

**Path:** `C:\Scripts\reconnect-iscsi.ps1`

```powershell
# Replace with your NAS IP (static)
$TargetPortal = "<Your-NAS-IP>"
# Replace with your iSCSI Target IQN
$TargetIQN = "<Your-iSCSI-IQN>"
$logPath = "C:\Scripts\iscsi-errors.log"

function Log-Error {
    param ($msg)
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "$timestamp ERROR: $msg" | Out-File $logPath -Append
}

$session = Get-IscsiSession | Where-Object { $_.TargetNodeAddress -eq $TargetIQN }

if ($session) {
    try {
        Disconnect-IscsiTarget -NodeAddress $TargetIQN -Confirm:$false
        Start-Sleep -Seconds 2
    } catch {
        Log-Error "Failed to disconnect stale session: $_"
    }
}

try {
    New-IscsiTargetPortal -TargetPortalAddress $TargetPortal -ErrorAction Stop
} catch {
    Log-Error "Failed to add portal: $_"
}

try {
    Connect-IscsiTarget -NodeAddress $TargetIQN -IsPersistent $true -ErrorAction Stop
} catch {
    Log-Error "Failed to connect to target: $_"
}
```
## Note: Set your static NAS IP (e.g., 192.168.1.x) and use it consistently in the script.

### 2. Created a .bat File to Launch the Script

**Path:** `C:\Scripts\reconnect-iscsi.bat`

```powershell
@echo off
powershell.exe -ExecutionPolicy Bypass -File "C:\Scripts\reconnect-iscsi.ps1"
```

### 3. Scheduled the Script to Run at Boot
Schedule Task Settings:

- Name: Reconnect iSCSI Target
- Trigger: At system startup, with 1-minute delay
- Action: Run C:\Scripts\reconnect-iscsi.bat
- User Account: Your standard local or domain user account (not SYSTEM)
- Run with highest privileges: Yes

**Conditions:**

"Start only if the following network connection is available" >> Any connection

### 4. Volume Handling
- Used diskmgmt.msc to assign a static drive letter (e.g., S:) to the iSCSI volume
- Used iSCSI Initiator > Volumes and Devices > Auto Configure to improve drive detection

## Outcome
- On reboot, the stale session is auto-disconnected
- Target is reconnected cleanly as a block device via iSCSI
- Volume shows up in This PC after ~1–2 minutes
- Errors (if any) are logged to C:\Scripts\iscsi-errors.log
- iSCSI Initiator now consistently shows Connected, not "Reconnecting..."

---

### Security & Reliability Considerations
- No saved plaintext credentials — iSCSI authentication not used in this test case
- Network reconnection is delayed to ensure readiness
- Script is run with highest privileges but limited in scope
- Minimal logging prevents information leakage unless there's an issue

---

### Key Takeaways
- iSCSI on Windows is prone to stale session issues
- Persistent favorites aren't always reliable for reconnecting
- Automating session teardown + rebuild is the most robust solution
- Scheduling boot-time tasks with network dependency adds resilience



















