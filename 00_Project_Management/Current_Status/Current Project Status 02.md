Here is the updates since our last time on ChatGPT on 13 August.

============

Status 001-06

============

GitHub Repository has been since updated to the Folder Hierarchy you outlined in within “Project History 3” in “Chat 001-07” under the explanation for “Step A – Organize the GitHub repository”. Currently there are only three to four folders, only because I cannot create empty folders within GitHub Repository without creating issues for future use.

You (ChatGPT) have not checked it out as of yet. Please take this opportunity to do so and let me know what is the accessibility status is.

============

Status 001-07

============

Continuation of the Robocopy verification is also outlined in the “Project History 3” in “Chat 001-07” under the explanation for “Step B – The Robocopy Verification”. It’s also located in “Chat 001-09” under “What I need from you for the three Robocopy logs”.  I’ve taken the liberty to obtain the last 35 to 45 lines of each of the log files for eSupport, MYASUS, and RESTORE.

I will upload the following files into their corresponding locations on the GitHub repository once you have validated that you can access the new Folder Hierarchy.

02_Factory_Preservation (Folder)

|

|- eSupport (Sub-Folder)

|     |-Copy_Log (Sub-Folder)

|           |- eSupport_Copy_LastLines.log

|

|- MYASUS (Sub-Folder)

|     |-Copy_Log (Sub-Folder)

|           |- MYASUS_Copy_LastLines.log

|
|- RESTORE (Sub-Folder)

|     |-Copy_Log (Sub-Folder)

|           |- RESTORE_Copy_LastLines.log

Please take this opportunity to review and analyze these logs so we can determine if the robocopy’s worked and we can now know that they will work when needed to rebuild the laptop with it’s new OS

============

Status 001-08

============

We have been able to create our External Backup of the Internal SSD on the External USB Drive we spoke about using Macrium Reflect’s “Image” operations – We did not create a “Clone” of the Internal SSD, just an “Image”. It is now stored on the External Backup Device, which is the same External USB Drive that holds all of the documents, spreadsheets, drivers, etc.

============

Status 001-09

============

We’ve also taken the liberty of building out the Windows 11 Pro Installation USB with RUFUS, this includes the manufacturing of the ei.cfg file within the “sources” directory. The ei.cfg file contains the following:

[EditionID]
Professional

[Channel]
Retail

[VL]
0

But we haven’t taken any action as of yet, largely because we’re still vetting that we have everything from the Internal SSD prior to wiping it out during the installation process.

============

Status 001-10

============

In our last conversation you inquired about “DriverQuery.cvs” file (83.1 KB), and I mistakenly directed to you to the “Installed_Driver_Inventory.csv” file (which is only 29.6 KB). And due to their different files sizes convinces me that they are not one in the same. Therefore, I’ve included the DriverQuery.cvs file in the sub-folder “Installed_Drivers” found in the parent folder “01_Factory_State”.

