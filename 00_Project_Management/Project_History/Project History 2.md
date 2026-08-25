============

Chat 001-06

============

Absolutely. Since you're logging off, let's turn this into a well-defined work package for your next session. You can work through it at your own pace and bring the results back.

And your Macrium/Ventoy/Rufus additions are excellent. I especially want to keep Macrium Reflect in reserve as a possible full factory-state image, before we destroy the factory partitions. I don't want to make that decision until we've inspected the factory data.

One correction to my earlier wording: I called the drive-letter assignment "random." It should actually be deliberate and temporary. Microsoft documents assign as assigning a drive letter/mount point to the volume in focus, but also notes that some OEM/GPT partitions cannot accept a drive letter through DiskPart. (Microsoft Learn) So if Windows refuses one of these assignments, that is not a failure and we will not force it.

Also, Microsoft itself demonstrates assigning letters to partitions during recovery/capture work, so the technique is legitimate when used carefully. (Microsoft Learn)

Factory Preservation Phase — Step 2

ASUS Factory Partition Examination & Preservation

Our objective in this phase is:

Determine exactly what ASUS put into RESTORE and MYASUS, preserve everything useful, and create a documented inventory before the SSD is erased.

We are still in read-only / preservation mode.

Absolutely do NOT do these things

Do not:

delete a partition

format a partition

initialize Disk 0

run clean

run clean all

run convert

run create

run delete partition

run format

change partition types

modify Secure Boot

clear TPM

run AsusInsWiz.exe

We're only looking.

Part A — First identify the USB drive

When you reconnect your Factory Preservation USB, plug it in and open PowerShell as Administrator.

Run:

Get-Volume |
    Sort-Object DriveLetter |
    Format-Table DriveLetter, FileSystemLabel, FileSystem, SizeRemaining, Size

This will show us which drive letter Windows assigned to the USB.

For example, you might see:

DriveLetter FileSystemLabel FileSystem SizeRemaining Size
----------- --------------- ---------- ------------- ----
C           OS              NTFS       ...
D           RecoveryDrv     NTFS       ...
E           FACTORY-BACKUP  NTFS       800 GB       1 TB

Do not assume E:.

Use whatever letter your USB actually receives.

For the examples below I'll call it:

E:

but substitute your actual USB letter.

Part B — Create the preservation directory structure

On the Factory Preservation USB:

New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\01_eSupport"
New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\02_RESTORE"
New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\03_MYASUS"
New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\04_System_Inventory"
New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\05_Driver_Inventory"
New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\06_Display_Dolby_Color"
New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\07_ASUS_Software"
New-Item -ItemType Directory -Force -Path "E:\Factory_Preservation\08_Documentation"

Don't worry if some directories aren't used yet.

We're creating a logical container for everything we're going to discover.

Part C — Preserve C:\eSupport

You already found this:

C:\eSupport
└── eDriver
    ├── LIB
    ├── Software
    ├── AsusInsWiz.exe
    ├── AsusInsWizMonitor
    ├── Autorun
    ├── FileList
    └── MD5

This is sufficiently important that I want the entire directory preserved, not just selected drivers.

We'll use robocopy, which is much better suited to preserving a large directory tree than ordinary drag-and-drop.

Run:

robocopy "C:\eSupport" "E:\Factory_Preservation\01_eSupport" /E /COPY:DAT /DCOPY:DAT /R:2 /W:2 /XJ /V /TEE /LOG:"E:\Factory_Preservation\08_Documentation\eSupport_Copy.log"

What this does

/E — copies all subdirectories, including empty ones

/COPY:DAT — copies data, attributes and timestamps

/DCOPY:DAT — does the same for directories

/R:2 — only retries twice if something is inaccessible

/W:2 — waits two seconds between retries

/XJ — avoids junction-related recursion

/V — verbose output

/TEE — displays output while also logging it

/LOG: — saves the copy log

It does not delete anything from C:.

If Robocopy reports errors, don't start troubleshooting by changing permissions. Bring the output back to me.

Part D — Now we examine RESTORE

This is where we need to be deliberate.

From your confirmed partition map:

Partition 5 = Primary = 28 GB

and your volume listing showed:

Volume 1 = RESTORE = NTFS = 28 GB

So we're very confident that:

Partition 5 = RESTORE

We don't need to "randomly" guess.

First, inspect it with DiskPart

Open Command Prompt or Terminal as Administrator:

diskpart

Then:

select disk 0
select partition 5
detail partition

Stop there.

Before assigning anything, record the output of:

detail partition

It should tell us additional information about the partition.

If it looks correct and identifies the 28 GB RESTORE partition, then:

assign letter=R

Important

If DiskPart says something like:

DiskPart successfully assigned the drive letter...

then exit:

exit

Now look in File Explorer for:

R:\

If Windows can read it, do not modify anything inside it.

Part E — Inventory RESTORE before copying it

Open an Administrator PowerShell and run:

Get-ChildItem -LiteralPath "R:\" -Force -Recurse -File -ErrorAction SilentlyContinue |
    Select-Object FullName, Length, LastWriteTime, Attributes |
    Sort-Object FullName |
    Out-File -FilePath "$env:USERPROFILE\Desktop\RESTORE_FileInventory.txt" -Encoding UTF8

Then:

tree "R:\" /F /A > "$env:USERPROFILE\Desktop\RESTORE_Tree.txt"

Don't execute anything you find.

Especially don't launch:

.exe

.bat

.cmd

.msi

recovery utilities

ASUS utilities

We're cataloguing them.

Part F — Preserve RESTORE

Assuming R: successfully mounted and is readable:

robocopy "R:\" "E:\Factory_Preservation\02_RESTORE" /E /COPY:DAT /DCOPY:DAT /R:2 /W:2 /XJ /V /TEE /LOG:"E:\Factory_Preservation\08_Documentation\RESTORE_Copy.log"

Again:

No deletion or formatting occurs on R:.

Part G — Remove the temporary RESTORE drive letter

Once the copy is complete:

diskpart
select disk 0
select partition 5
remove letter=R
exit

This is important.

We're not leaving the ASUS recovery partition exposed as a normal drive after we're finished examining it.

Part H — Now MYASUS

Your earlier volume listing gave us:

Volume 4
Label: MYASUS
FAT32
260 MB
Hidden

And the partition map gave us:

Partition 6
Type: Recovery
Size: 260 MB
Offset: 953 GB

So we're very likely looking at:

Partition 6 = MYASUS

But we're going to verify it before touching it.

Run:

diskpart
select disk 0
select partition 6
detail partition

Stop and record the output.

If it corresponds to the 260 MB FAT32 MYASUS volume, try:

assign letter=M

If DiskPart refuses with an error indicating that the partition type doesn't permit a drive letter:

STOP.

Don't change the partition type.

Don't use set id.

Don't convert it.

Don't use third-party partition software.

Just bring me the error.

Microsoft specifically notes that OEM/GPT partitions may not be eligible for drive-letter assignment. (Microsoft Learn)

Part I — If MYASUS accepts the M: assignment

If:

assign letter=M

succeeds, exit DiskPart:

exit

Then:

Get-ChildItem -LiteralPath "M:\" -Force

And:

tree "M:\" /F /A > "$env:USERPROFILE\Desktop\MYASUS_Tree.txt"

Then:

Get-ChildItem -LiteralPath "M:\" -Force -Recurse -File -ErrorAction SilentlyContinue |
    Select-Object FullName, Length, LastWriteTime, Attributes |
    Sort-Object FullName |
    Out-File -FilePath "$env:USERPROFILE\Desktop\MYASUS_FileInventory.txt" -Encoding UTF8

Don't execute anything.

Part J — Preserve MYASUS

If it's readable:

robocopy "M:\" "E:\Factory_Preservation\03_MYASUS" /E /COPY:DAT /DCOPY:DAT /R:2 /W:2 /XJ /V /TEE /LOG:"E:\Factory_Preservation\08_Documentation\MYASUS_Copy.log"

Then remove the temporary letter:

diskpart
select disk 0
select partition 6
remove letter=M
exit

Very important: don't be surprised if MYASUS behaves differently

I'm actually expecting MYASUS to be more restrictive than RESTORE.

A 260 MB FAT32 partition marked as a Recovery partition is not necessarily designed to behave like a normal Windows data partition.

So if this happens:

assign letter=M

→ Access denied / cannot assign / OEM partition / invalid parameter

that's useful information.

Don't fight it.

Bring me the exact DiskPart message.

There are other ways we can inspect it later, including Windows PE/recovery tooling if necessary.

Part K — Search the existing factory installation for the resources you specifically mentioned

While we're still in factory Windows, we're going to locate the important things you originally identified.

Run this PowerShell command:

Get-ChildItem -Path "C:\" -Recurse -File -Force -ErrorAction SilentlyContinue |
    Where-Object {
        $_.Extension -in ".icc",".icm" -or
        $_.Name -ieq "PQConfig.dv"
    } |
    Select-Object FullName, Length, LastWriteTime |
    Sort-Object FullName |
    Out-File "$env:USERPROFILE\Desktop\Display_Dolby_Color_Inventory.txt" -Encoding UTF8

This may take a while because it searches C:.

That's okay.

Do not delete anything it finds.

Part L — Search specifically for ASUS factory licensing/configuration resources

We're also going to search filenames rather than making assumptions about their locations.

Get-ChildItem -Path "C:\" -Recurse -File -Force -ErrorAction SilentlyContinue |
    Where-Object {
        $_.Name -match '(?i)(PQConfig|Dolby|ASUS|eSupport|Color|Calibration|Avatar|Wallpaper)'
    } |
    Select-Object FullName, Length, LastWriteTime |
    Sort-Object FullName |
    Out-File "$env:USERPROFILE\Desktop\ASUS_Resource_Search.txt" -Encoding UTF8

This will produce a potentially large file.

That's intentional.

Don't paste the whole thing to me.

We'll examine it selectively next session.

Part M — Capture the installed driver inventory

This is one of the most important pieces of our preservation work.

Run:

Get-CimInstance Win32_PnPSignedDriver |
    Where-Object {$_.DeviceID} |
    Select-Object DeviceName, Manufacturer, DriverVersion, DriverDate, InfName, DeviceID |
    Sort-Object DeviceName |
    Export-Csv "$env:USERPROFILE\Desktop\Installed_Driver_Inventory.csv" -NoTypeInformation -Encoding UTF8

This gives us a hardware-driver map.

We'll compare this against the ASUS factory eSupport repository and ASUS's current G814FP packages.

That comparison is especially valuable because ASUS's current G814FP support page now contains 25H2-specific packages—for example current MediaTek WLAN, Realtek Audio, AMD graphics, NVIDIA graphics, and ASUS touchpad packages. (ASUS Global)

Part N — Capture the installed ASUS software

Run:

Get-ItemProperty `
    "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*", `
    "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" `
    -ErrorAction SilentlyContinue |
    Where-Object {$_.DisplayName -match '(?i)ASUS|ROG|Armoury|MyASUS|Dolby'} |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate, InstallLocation |
    Sort-Object DisplayName |
    Export-Csv "$env:USERPROFILE\Desktop\Installed_ASUS_Software.csv" -NoTypeInformation -Encoding UTF8

Again, read-only.

This will help us determine what ASUS actually installed versus what merely exists in eSupport.

Part O — Capture Windows edition/licensing information

Before we eventually replace Home, I want a record of the factory state.

Run:

Get-ComputerInfo |
    Select-Object WindowsProductName, WindowsVersion, OsBuildNumber, BiosVersion, BiosReleaseDate |
    Out-File "$env:USERPROFILE\Desktop\Factory_System_Information.txt" -Encoding UTF8

And:

slmgr /dlv > "$env:USERPROFILE\Desktop\Factory_Windows_Licensing.txt"

Do not post a full product key if one appears anywhere.

If you see a partial/default key, that's fine; just don't expose a complete key.

Part P — Capture the ASUS BIOS version

This is important because ASUS currently lists BIOS 316 as the latest G814FP BIOS, dated June 16, 2026, but we are NOT automatically going to update yours right now. (ASUS Global)

Run:

Get-CimInstance Win32_BIOS |
    Select-Object Manufacturer, SMBIOSBIOSVersion, BIOSVersion, ReleaseDate |
    Format-List

Bring me the result.

Do not update BIOS just because a newer version exists.

The factory configuration is our baseline at this stage.

Part Q — The Macrium decision

After we have completed the inventories above, stop.

Don't wipe the SSD.

Don't begin Windows Setup.

Don't create the Pro USB yet.

At that point we'll have enough information to make an informed decision about creating a full factory SSD image with Macrium Reflect.

If we decide to do that, the external image would effectively become our:

"Break glass — restore this laptop to the exact factory state it had before we started."

That's a very valuable safety net.

What I want you to bring back next session

You don't need to dump everything into the conversation.

Bring me these results/files or their relevant contents:

Already requested

eSupport_FileInventory.txt

eSupport_Tree.txt

New

RESTORE_Tree.txt

RESTORE_FileInventory.txt

MYASUS_Tree.txt — if accessible

MYASUS_FileInventory.txt — if accessible

Display_Dolby_Color_Inventory.txt

ASUS_Resource_Search.txt

Installed_Driver_Inventory.csv

Installed_ASUS_Software.csv

Factory_System_Information.txt

Factory_Windows_Licensing.txt

BIOS information

And especially:

The detail partition output for:

Partition 5
Partition 6

If MYASUS refuses the drive-letter assignment, give me the exact error message instead.

One final recommendation before you shut down

Since this machine has a factory installation you intend to destroy, I would not rush the next phase.

ASUS's current G814FP support infrastructure is healthy: ASUS is publishing current Windows 11 25H2 drivers for the platform and firmware updates specific to G814FP. (ASUS Global)

That means we can take our time and build a very good clean-install recipe.

Our eventual workflow will be:

PHASE 1 — Factory Discovery

↓

PHASE 2 — Factory Preservation ← you're working here

↓

PHASE 3 — Optional Macrium Factory Image

↓

PHASE 4 — Build Windows 11 Pro USB

↓

PHASE 5 — Verify Pro edition-selection mechanism

↓

PHASE 6 — Erase ALL six factory partitions

↓

PHASE 7 — Clean Windows 11 Pro installation

↓

PHASE 8 — Drivers in controlled order

↓

PHASE 9 — ASUS/MyASUS/Armoury Crate + required components

↓

PHASE 10 — Dolby/color/calibration/ASUS resources

↓

PHASE 11 — Your applications and data architecture

↓

PHASE 12 — Comprehensive hardware/function test

↓

PHASE 13 — Golden Image

↓

PHASE 14 — Your custom recovery architecture

And I think that is a considerably safer approach than trying to accomplish the whole thing in one sitting.

When you come back, don't start with the Windows installation. Start by giving me the preservation/inventory results. We'll pick up exactly there.

============

Chat 001-06

============

Sounds good. Take your time, and leave the factory SSD untouched until we finish the preservation work.

When you return, just bring back the inventory results and any files/output you collected. We can pick up from Factory Preservation Phase — Step 2 without needing to redo the earlier work.

The key rule for now: observe, inventory, and copy—don't modify. 👍

Good luck with the data gathering, and we'll take the clean Pro installation from there when you're back.

