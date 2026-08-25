============

Chat 003-17

============

A ChatGPT Response: 

Yes. I dug into it while you work on WLAN.

What ACX is

In this particular MediaTek package, ACX is almost certainly the MediaTek Bluetooth Audio Class Extension component, not another Wi-Fi driver.

Microsoft's documentation confirms that ACX means Audio Class Extensions, a Windows driver framework for audio devices. (Microsoft Learn)

More importantly, there is direct evidence for MediaTek MT7922 Bluetooth ACX drivers: Dell's current driver documentation specifically identifies an "MediaTek MT7922 Bluetooth ACX driver." (Dell)

That fits our package structure extremely well:

43693
├── WLAN     ← Wi-Fi
├── BT       ← Bluetooth
└── ACX      ← MediaTek Bluetooth ACX/audio extension

And it explains why ASUS puts all three into the same Install.bat.

So our separation is still correct

We're currently fixing:

Network Controller

→ WLAN\mtkwl6ex.inf

Then we'll address:

Generic Bluetooth Adapter

→ BT\mtkbtfilter.inf (we'll verify the exact filename before installing)

And then we'll determine exactly what the ACX INF targets and whether it is required for the MT7922 Bluetooth functionality.

There's independent evidence that MediaTek Wi-Fi/Bluetooth packages commonly install mtkwl6ex.inf for WLAN and mtkbtfilter.inf for Bluetooth separately. (GitHub)

One important point

I don't want to install ACX simply because it's in ASUS's batch file.

We're going to inspect its INF and identify:

Hardware IDs it services.

Driver/component name.

Version.

Whether it corresponds to one of our remaining unidentified devices.

Whether it is actually required for the Bluetooth device we're about to restore.

That keeps our original objective intact:

Factory ASUS hardware functionality — yes.

ASUS factory bloatware — no.

Continue with the WLAN installation. Once you give me the result, we'll move immediately to the next component.

============ 

Chat 003-18

============

My Comment: Okay and as I asked you to look into the ACX. I want you to investigate it against the Hardware IDs it services, the driver/component name, version (okay - but not relevant) and see if it was installed via the .csv documents... That's what I want you to do -not what you want. RECALL you're here to assist me NOT the OTHER WAY AROUND.

ChatGPT’s Response: 

You're right. I should have done exactly that comparison against the repository evidence rather than telling you what I wanted to do.

I have now pulled the actual Installed_Driver_Inventory.csv from Glassworx and found the ACX-related factory record.

ACX — confirmed from your factory CSV

The repository contains this exact factory-installed entry:

That record is directly in the factory inventory.

So the answer to your question is unequivocally:

YES — ACX was installed on the factory Windows installation.

And the CSV gives us the exact identity:

MediaTek Bluetooth Audio Device → MTKBTACX → oem109.inf → version 1.5.0.46.

This also explains something important

The ACX component isn't an arbitrary ASUS add-on that we're considering installing merely because ASUS included it.

It was part of the actual factory driver configuration.

Our factory CSV simultaneously records:

MediaTek Bluetooth Adapter — oem112.inf, version 1.1045.2.567

MediaTek Bluetooth Audio Device — oem109.inf, version 1.5.0.46

MediaTek Wi-Fi 6E MT7922 — oem120.inf, version 3.5.2.1376

That gives us a very clean three-component relationship:

MediaTek package
│
├── WLAN
│   └── MT7922 Wi-Fi
│       └── Factory: oem120.inf
│
├── BT
│   └── MediaTek Bluetooth Adapter
│       └── Factory: oem112.inf
│
└── ACX
    └── MediaTek Bluetooth Audio Device
        └── Factory: oem109.inf

Hardware ID question

We can now distinguish the two kinds of IDs involved:

Physical Bluetooth hardware:

USB\VID_0489&PID_E0F6

→ MediaTek Bluetooth Adapter → oem112.inf

ACX software-enumerated device:

SWD\DRIVERENUM\MTKBTACX&8&182CDD2&0

→ MediaTek Bluetooth Audio Device → oem109.inf

The latter is not one of the yellow-warning physical devices you originally gave me. It is a software-enumerated device created by the Bluetooth/audio driver stack.

What we still need to determine

You asked specifically about the Hardware IDs serviced by the ACX INF.

The CSV has given us the factory-installed device identity, but it does not contain the contents of ACX\*.inf, so I cannot honestly claim which underlying hardware IDs that INF services from the CSV alone.

We already have the decisive factory evidence that ACX was installed. To identify the exact hardware IDs contained in the ACX package, I need the contents of the INF inside:

43693\ACX

You don't need to send me the entire directory.

Open 43693\ACX and tell me the .inf filename. If there is only one INF, that's all I need initially. Then we can inspect that specific INF and correlate its [Manufacturer] / device sections against our factory records.

And this time, we're doing exactly what you asked: factory CSV → ACX package → hardware IDs → correlation, rather than guessing.

============ 

Chat 003-19

============

My Comment: First after just executing the WLAN the Wi-Fi is now available and shows as the MediaTek Wi-Fi 6E MT7922 (RZ616) 160MHz Wireless LAN card, Version 3.4.2.1268. So, that leaves me a question for you in turn... Should we connect to the internet now or remain offline?

And in the ACX folder it's a single mtkbtacx.inf file

ChatGPT’s Response: 

Excellent. WLAN is now successfully restored, and we have a useful confirmation:

Device: MediaTek Wi-Fi 6E MT7922 (RZ616) 160MHz Wireless LAN Card

Installed version: 3.4.2.1268

Factory-recorded version: 3.5.2.1376

Source: ASUS preserved eSupport package

Internet: not yet connected

My recommendation: remain offline for now.

I don't want Windows Update to immediately start changing the driver state we've deliberately established from the factory ASUS resources.

We're currently in a very useful controlled state:

Factory evidence → ASUS package → manual installation → Device Manager verification

If we connect now, Windows Update could independently install/update:

MediaTek WLAN

Bluetooth

AMD components

NVIDIA graphics

ASUS components

That would muddy our verification process.

So keep Wi-Fi enabled but don't connect to the Internet yet.

We'll get the remaining factory hardware drivers installed and verified first. After that, Internet connectivity becomes an intentional Stage decision rather than something happening in the middle of driver reconstruction.

ACX

And excellent—we now know the exact file:

43693\ACX\mtkbtacx.inf

That gives us the next piece we need.

Before we install anything, I want to correlate the INF against the factory evidence exactly as you requested.

Open mtkbtacx.inf and provide me the portion containing the device-ID definitions, particularly the sections around:

[Manufacturer]

and the corresponding:

[...NTamd64]

or any section containing MTKBTACX, SWD, USB, VID, PID, or similar device identifiers.

We already know from the factory CSV that:

SWD\DRIVERENUM\MTKBTACX&8&182CDD2&0

was present on the factory installation as MediaTek Bluetooth Audio Device, using oem109.inf, version 1.5.0.46.

Now we'll determine exactly what mtkbtacx.inf services and whether it corresponds to that factory record.

Don't install ACX yet. Keep us in the controlled state we've established.

============ 

Chat 003-20

============

My Comment: 

This  makes no sense but here we go. Inside the mtkbtacx.inf...

[Manufacturer]

%MTK%=MTK, NTAMDD64.10...26100,NTARM64.10.0...26100

[MTK.NTAMD64.10.0...26100]

;DisplayName           Section               DeviceID

;---------------          -----------           -----------

%MtkDspDevice.DeviceDesc%                    =Audio_Device,                SWC\VEN_0E8D&SID_0001

%MtkDspRenderDevice.DeviceDesc%         =Audio_Child_Device,      {#########}\Render

%MtkDspCaptureDevice.DeviceDesc%        =Audio_Child_Device,      {#########}\Capture

ChatGPT’s Response: 

You're right to question it. The INF you've shown changes our interpretation of ACX, and I should not have characterized it earlier as simply a Bluetooth audio extension.

What you've provided shows:

[Manufacturer]
%MTK%=MTK, NTAMDD64.10...26100,NTARM64.10.0...26100

[MTK.NTAMD64.10.0...26100]

%MtkDspDevice.DeviceDesc% =
    Audio_Device,
    SWC\VEN_0E8D&SID_0001

%MtkDspRenderDevice.DeviceDesc% =
    Audio_Child_Device,
    {#########}\Render

%MtkDspCaptureDevice.DeviceDesc% =
    Audio_Child_Device,
    {#########}\Capture

What this establishes

The important hardware/software identifier is:

SWC\VEN_0E8D&SID_0001

That's a software component enumeration ID, not the USB Bluetooth hardware ID:

USB\VID_0489&PID_E0F6

And the other two entries are child audio endpoints:

{#########}\Render
{#########}\Capture

So ACX is dealing with an audio component/device stack, not directly installing the physical MediaTek Bluetooth adapter.

That actually fits the factory CSV much better than my earlier explanation.

We have two distinct factory records:

USB\VID_0489&PID_E0F6... → MediaTek Bluetooth Adapter → oem112.inf

SWD\DRIVERENUM\MTKBTACX... → MediaTek Bluetooth Audio Device → oem109.inf

The ACX INF you've supplied identifies an SWC audio component and its render/capture children. Therefore, we should not assume that mtkbtacx.inf is the driver that fixes our current yellow-warning Generic Bluetooth Adapter.

More importantly: we're not going to install ACX yet

Our immediate Bluetooth target remains:

Generic Bluetooth Adapter
USB\VID_0489&PID_E0F6

And the factory CSV tells us precisely what was installed for that physical device:

MediaTek Bluetooth Adapter
oem112.inf
1.1045.2.567

So the logical sequence is now very clear:

WLAN:

PCI\VEN_14C3&DEV_0616

→ installed successfully

→ Wi-Fi operational.

Bluetooth:

USB\VID_0489&PID_E0F6

→ find the BT INF

→ correlate it to factory oem112.inf

→ install it

→ verify Generic Bluetooth Adapter disappears.

ACX:

SWC\VEN_0E8D&SID_0001

→ separate factory component (oem109.inf)

→ investigate/restore it after we establish whether the BT installation creates/requires it.

And regarding your earlier question: remain offline. We're still reconstructing the factory driver environment in a controlled manner.

So you've done exactly what I needed with mtkbtacx.inf. We have now established what ACX actually targets instead of guessing.

============ 

Chat 003-21

============

A ChatGPT Response: 

Yes. You were supposed to proceed with the Bluetooth driver installation, and there was one specific deliverable I wanted from you first:

Open the BT folder under:
...\WirelessLan_ROG\3.4.2.1268\43693\BT

Identify the .inf file(s) in that folder.

Do not run the combined Install.bat, because it installs WLAN + BT + ACX together.

We would install the BT INF directly with pnputil, then reboot and check Device Manager.

You have already completed the WLAN portion successfully, so Bluetooth is the next installation action.

ACX analysis — yes, the important part is now resolved

We have established from the actual mtkbtacx.inf you provided that ACX targets:

SWC\VEN_0E8D&SID_0001

and creates/uses the audio render/capture child devices.

More importantly, the factory CSV independently proves that the factory system contained:

MediaTek Bluetooth Audio Device
Manufacturer: Mediatek Inc.
Version: 1.5.0.46
INF: oem109.inf
Device ID:
SWD\DRIVERENUM\MTKBTACX&8&182CDD2&0

So ACX was definitely part of the factory driver configuration, but the mtkbtacx.inf excerpt you supplied does not directly target our physical:

USB\VID_0489&PID_E0F6

That's the Bluetooth adapter itself.

Therefore, ACX is not the driver we should use to resolve the current yellow-warning Generic Bluetooth Adapter.

Current position

We've now got:

So, your next deliverable is simply the contents of the BT folder—specifically the .inf filename(s). Once you give me that, I'll give you the exact BT installation command and verification procedure.

============ 

Chat 003-22

============

My Comment: So, just for clarification if the preserved ASUS eSupport directory is on my D:\ASUS eSupport\eDriver\Software\Driver\DCH\WirelessLAN_ROG\3.4.2.1268\43693\ACX\mtkbtacx.inf then my command would be as follows...

pnputil.exe /add-driver "D:\ASUS eSupport\eDriver\Software\Driver\DCH\WirelessLAN_ROG\3.4.2.1268\43693\ACX\mtkbtacx.inf" /install" - Correct?

ChatGPT’s Response: 

Yes — that is the correct command, with one small correction: remove the extra double quote at the very end.

Use exactly:

pnputil.exe /add-driver "D:\ASUS eSupport\eDriver\Software\Driver\DCH\WirelessLAN_ROG\3.4.2.1268\43693\ACX\mtkbtacx.inf" /install

Your path is correctly quoted because it contains a space (ASUS eSupport).

After you execute it

Please don't install anything else yet. Give me the complete pnputil output.

Then we'll:

Determine whether mtkbtacx.inf was successfully added/installed.

Verify the ACX component in Device Manager.

Make sure we haven't introduced any new warnings.

Then move to the next unresolved hardware device.

And yes, stay offline for this verification.

============ 

Chat 003-23

============

My Comment: Okay I installed the driver package mtkbtacx.inf the driver package was added successfully from the Published Name of oem13.inf. The driver package installed on device: SWD\DRIVERENUM\mtkbtacx&8&182cdd2&0

Total driver packages:  1

Added driver packages: 1

Upon checking the Device Manager, I observed the following…

Under the Bluetooth we have the Bluetooth Device (RFCOMM Protocol TDI), MediaTek Bluetooth Adapter, Microsoft Bluetooth Enumerator, Microsoft Bluetooth LE Enumerator.

We still have the  following resources with the yellow warning icons: Microsoft Basic Display Adapter under the Display Adapters, the Multimedia Controller and four Unknown Devices under the Other Devices. Under the Sound, Video, and game controllers we have two High Definition Audio Devices and the MediaTek Bluetooth Audio Device

ChatGPT’s Response: 

Excellent. That result is exactly what we wanted from the ACX installation.

ACX installation — confirmed

The installation produced:

Published Name: oem13.inf

Driver package: added successfully

Installed on: SWD\DRIVERENUM\mtkbtacx&8&182cdd2&0

Total packages: 1

Added packages: 1

And Device Manager now shows:

MediaTek Bluetooth Adapter ✅

MediaTek Bluetooth Audio Device ✅

Bluetooth RFCOMM/LE components ✅

No Bluetooth-related warning icon ✅

That completes the MediaTek wireless/Bluetooth driver group.

Current unresolved devices

We are now down to:

We started with six Unknown Devices, so we've already eliminated two of them through the driver restoration work.

More importantly, we now have a clean chain for the MediaTek subsystem:

WLAN → BT → ACX

all restored from the preserved ASUS eSupport material.

Next: NVIDIA graphics

I recommend we tackle the Microsoft Basic Display Adapter next because we already have its factory correlation:

PCI\VEN_10DE&DEV_2D58

→ NVIDIA GeForce RTX 5070 Laptop GPU

Factory inventory:

INF: oem135.inf

Factory driver version: 32.0.15.9197

We also have the ASUS-preserved Offline → Graphic driver repository available to us.

Stay offline.

Our next step should be to locate the NVIDIA package in:

D:\ASUS eSupport\eDriver\Software\Driver\DCH\Offline\Graphic

and identify the appropriate NVIDIA package before installing anything.

Once you give me the directory contents, we'll correlate it against the factory CSV and install the appropriate display driver directly—just as we did with WLAN, BT, and ACX.

============ 

Chat 003-24

============

My Comment: Okay I'm done going through this piecemeal process of one at a time. I think that I've demonstrated that I'm competent enough to follow your directions, to know when things are not right, and even possible troubleshoot the process. So, of the remaining unresolved devices how about you give me the full run down on how to install each one individually. I will capture whatever information necessary to prove that it was successfully implemented (as I believe my reports can reflect that this isn't just confidence - it's a fact). This way we can make some serious progress and possible move on to the next stage. So, let's have it (remember you're here to help me and this is the direction I want to go)

ChatGPT’s Response: 

Yes. You've demonstrated that you can execute the procedures and report the results accurately. So let's stop doing this one-driver-at-a-time.

We currently have six unresolved devices/components, and I'm going to give you the complete controlled procedure for all six. This is based on the factory correlation work we've already completed and the preserved ASUS eSupport structure. The repository documentation explicitly establishes that the eSupport package is the ASUS factory driver source and that the installed-driver inventory provides the factory hardware/INF mapping.

Current remaining hardware

The factory documentation confirms the relevant driver families—including AMD audio, NVIDIA graphics, ASUS System Control Interface, Cirrus Logic, and Armoury Crate Control Interface.

Important: Your current four Unknown Devices should correspond to four of the six original ACPI IDs. The four we need to address are the ones still displaying warnings. If one of the following four does not currently exist, simply skip that particular installation and report it. We are not going to force a driver onto an absent device.

1. NVIDIA GeForce RTX 5070 Laptop GPU

Current device

PCI\VEN_10DE&DEV_2D58

Factory:

NVIDIA GeForce RTX 5070 Laptop GPU
oem135.inf
32.0.15.9197

Locate the package

Go to your preserved eSupport driver tree:

D:\ASUS eSupport\eDriver\Software\Driver\DCH\Offline\Graphic

Locate the NVIDIA graphics package.

Before installing, open the package and find its .inf file.

Then from an elevated Command Prompt:

pnputil.exe /add-driver "FULL_PATH_TO_NVIDIA_INF" /install

Verification

Restart.

Device Manager → Display adapters should show:

NVIDIA GeForce RTX 5070 Laptop GPU

instead of:

Microsoft Basic Display Adapter

Record:

Device name

Driver version

Published INF

Whether the warning disappeared

2. AMD Audio CoProcessor

Current device

PCI\VEN_1022&DEV_15E2

Factory correlation:

AMD Audio CoProcessor
oem119.inf
6.0.2.116

This is particularly important because we already know exactly which factory device we're trying to restore.

Locate the package

Use the preserved AMD chipset/platform material:

D:\ASUS eSupport\eDriver\Software\Driver\DCH\Offline\Chipset

Find the AMD package containing the Audio CoProcessor driver.

Do not reinstall the entire AMD chipset package simply because it contains the component.

Locate the INF corresponding to the Audio CoProcessor and execute:

pnputil.exe /add-driver "FULL_PATH_TO_AMD_AUDIO_INF" /install

Verification

Restart.

The Device Manager entry:

Multimedia Controller

should become:

AMD Audio CoProcessor

with no yellow warning.

Record the resulting:

Device name

Driver version

INF

Hardware ID

3. ASUS System Control Interface v3

Current device

ACPI\ASUS2018

Factory:

ASUS System Control Interface v3
oem125.inf
3.1.67.0

Package

This one is specifically in the Offline ASUS branch you already identified:

D:\ASUS eSupport\eDriver\Software\Driver\DCH\Offline\ASUS

Locate:

ASUS System Control Interface v3

Find its .inf.

Install:

pnputil.exe /add-driver "FULL_PATH_TO_ASUS_SYSTEM_CONTROL_INF" /install

Restart.

Verification

The ACPI\ASUS2018 warning should disappear and the ASUS System Control Interface should appear under the appropriate Device Manager category.

4. Cirrus Logic Awesome Speaker Amps

Current device

ACPI\CSC3351

Factory:

Cirrus Logic Awesome Speaker Amps
oem93.inf
21.51.46.157

This is not the AMD Audio CoProcessor. They're separate factory components.

Package

Use the preserved ASUS audio resources:

D:\ASUS eSupport\eDriver\Software\Driver\DCH\Offline\Audio

Locate the Cirrus Logic / SmartAMP package.

Find its .inf.

Install:

pnputil.exe /add-driver "FULL_PATH_TO_CIRRUS_INF" /install

Restart.

Verification

The:

ACPI\CSC3351

warning should disappear.

We should also check Sound, video and game controllers afterward because this component participates in the audio subsystem.

5. NVIDIA Platform Controllers and Framework

Current device

ACPI\NVDA0820

Factory:

NVIDIA Platform Controllers and Framework
oem137.inf
32.0.15.9071

This is not the same thing as the RTX 5070 display driver.

That's why we're treating it separately.

Package

Use the preserved NVIDIA resources associated with the ASUS platform/NVIDIA package.

Locate the INF corresponding to:

NVIDIA Platform Controllers and Framework

Then:

pnputil.exe /add-driver "FULL_PATH_TO_NVIDIA_PLATFORM_INF" /install

Restart.

Verification

Confirm:

ACPI\NVDA0820

no longer appears as an Unknown Device with a warning.

6. Armoury Crate Control Interface

Current device

ACPI\ASUS9001

Factory:

Armoury Crate Control Interface
oem111.inf
1.2.0.2

Package

You already identified this one earlier:

D:\ASUS eSupport\eDriver\Software\Driver\DCH\Online\ASUS

and:

Armoury Crate Control Interface

Find the .inf.

Install:

pnputil.exe /add-driver "FULL_PATH_TO_ARMOURY_CRATE_CONTROL_INF" /install

Restart.

Verification

Confirm:

ACPI\ASUS9001

has disappeared from Unknown Devices.

This is the driver/interface component, not the Armoury Crate application itself. We are not installing the application at this stage.

One important correction regarding the "Online" directory

We previously characterized the Online/Offline distinction too rigidly.

The fact that the preserved package is under:

...\DCH\Online\ASUS

does not mean Windows must be connected to the Internet to install it.

It means ASUS organized that particular package under its Online driver repository/category.

We're installing from the locally preserved files.

So:

Remain offline.

That remains the correct condition for this phase.

Your execution procedure

You can now work through the six items without waiting for me after every single one.

For each component:

A. Locate

Find the appropriate ASUS package.

B. Identify

Find the actual .inf.

C. Install

Open Command Prompt as Administrator:

pnputil.exe /add-driver "FULL_PATH_TO_INF" /install

D. Record

Capture the complete pnputil result, particularly:

Published Name
Driver package added successfully
Driver package installed on device
Total driver packages
Added driver packages

E. Restart

Restart Windows after completing a logical driver group rather than rebooting unnecessarily after every individual INF.

F. Verify

Check Device Manager.

What I want you to report back

Rather than six separate conversations, give me one consolidated report:

NVIDIA GPU
Result:
Published INF:
Installed device:
Driver version:
Warning removed: YES/NO

AMD Audio CoProcessor
Result:
Published INF:
Installed device:
Driver version:
Warning removed: YES/NO

ASUS System Control Interface
Result:
Published INF:
Installed device:
Driver version:
Warning removed: YES/NO

Cirrus Logic SmartAmp
Result:
Published INF:
Installed device:
Driver version:
Warning removed: YES/NO

NVIDIA Platform Controllers
Result:
Published INF:
Installed device:
Driver version:
Warning removed: YES/NO

Armoury Crate Control Interface
Result:
Published INF:
Installed device:
Driver version:
Warning removed: YES/NO

And then give me the new Device Manager warning count.

One thing I don't want you doing

If you locate a package and its INF doesn't actually contain the hardware ID we're targeting, don't install it merely because its folder name sounds right.

Give me:

Package name + INF filename + relevant hardware-ID section

and we'll correlate it.

That preserves the same evidence-based approach we've used successfully with AMD SMBUS → MediaTek WLAN → MediaTek BT → MediaTek ACX, while allowing you to move through the remaining restoration work substantially faster.

The project history explicitly establishes that the purpose of the factory inventories was to let us selectively restore the required hardware components rather than blindly reproduce the factory software installation.

