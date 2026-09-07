# Standard Operating Procedure: Hardware Provisioning via Windows Autopilot & Intune

## Document Information
* **Role/Scope:** Cloud Operations / Endpoint Administration
* **Purpose:** Procedure for capturing endpoint hardware hash IDs, uploading them to Microsoft Intune, assigning security groups and executing initial cloud image deployment.
* **Target OS:** Windows 11

---

## Technical Requirements
* Windows 11 target endpoint
* Active network connection (Ethernet recommended during initial image download)
* Access to Microsoft Intune Admin Center
* Execution script: `Get-AutopilotInfo.ps1`

---

## Execution Steps

### Step 1: Extract Hardware Hash ID
1. Connect to the target machine via remote desktop or remote support tools.
2. Copy the directory containing `Get-AutopilotInfo.ps1` to the desktop.
3. Open **PowerShell** as Administrator and run the hash extraction script:

```
$scriptPath = "C:\Users\Public\Desktop\HASH\Get-AutopilotInfo.ps1"
$serial = (Get-WmiObject -Class Win32_BIOS).SerialNumber
$output = "C:\Users\Public\Desktop\HASH\HASH_$serial.csv"

PowerShell -NoProfile -ExecutionPolicy Unrestricted -Command "& '$scriptPath' -OutputFile '$output' -Append"
Remove-Item $scriptPath -Force
```
*(Note: If logged in as Administrator, adjust $scriptPath and $output to use C:\Users\Administrator\Desktop\HASH\...)*

4. Transfer the generated HASH_<SerialNumber>.csv file to your local workstation.
5. Power off the target machine.

### Step 2: Register the device in Microsoft Intune
1. Log in to the Microsoft Intune Admin Center.
2. Navigate to Devices > Enrollment > Windows Autopilot > Devices.
3. Select Import. You can stage multiple CSV files before initiating the batch upload.
4. Monitor processing via the notifications menu (ingestion takes approximately 1–2 minutes).
5. Confirm successful import once the notification updates to completed status.

### Step 3: Group Assignment & Profile Deployment
1. In the Intune Admin Center, navigate to Groups > All Groups.
2. Locate the baseline Autopilot group and the target department security group (e.g., Technical Apps - Tech, Healthcare Apps - Clinical, Administrative Apps - Admin, or Management Apps - Exec).
3. Open the assigned group, select Members > Add Members, search by device Serial Number, and confirm selection.
4. Verify that group assignments update correctly under Devices > Enrollment > Windows Autopilot > Devices.

### Step 4: Endpoint Recovery & Image Deployment
1. Power on the endpoint and press F11 repeatedly during boot to enter the Recovery Environment.
2. Select the following sequence:
    - Troubleshoot > Reset this PC
    - Remove everything > Cloud download
    - Fully clean the drive > Reset
3. Ensure the device remains connected to power and wired network during the build.

### Step 5: Post-Deployment Configuration & Verification
1. Prompt the user to sign in with their corporate account credentials at the Out-of-Box Experience (OOBE) screen.
2. Allow automated application pushes to finish via Intune policies.
3. Complete final onboarding checks:
    - Remote Support: Configure remote access agent and set up unattended access password.
    - Peripherals: Map required local and network printers.
    - Cloud Storage: Launch OneDrive and verify user directory sync.
    - System Maintenance: Run Windows Update, perform an Intune compliance sync and uninstall unapproved third-party utilities
4. Record completed details on the onboarding checklist.

### Troubleshooting & Known Issues
**Legacy Application Permission Flags**

If custom internal applications fail to launch due to Windows execution flags:
1. Copy the target software directory to %USERPROFILE%\Documents.
2. Inspect individual file properties and check Unblock under Security if flagged.
3. Copy the verified application directory to C:\Program Files\.  