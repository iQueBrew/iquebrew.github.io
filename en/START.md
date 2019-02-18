## A complete guide to hacking the iQue Player, from stock to cipher block-jack and recrypt.sys stuff

Guide to installing Jbop’s HackIt Patcher on a stock iQue Player

This guide assumes that you are familiar with the Windows command processor (or the equivalent on your OS of choice), have access to either a virtual machine or secondary PC running Windows XP 32 bit, have the iQue@Home drivers installed on Windows XP and are familiar with connecting your iQue Player to Windows XP using a USB cable. In addition, familiarising yourself with transferring files between Windows XP and your main OS will be helpful. Finally, download and install a hex editor such as **HxD** (Windows) and familiarise yourself with its use.
Note: step 4 of the process requires **Python 3** on your main OS. Please visit [python.org](https://www.python.org/ "Python's homepage") to download and install it, if your main OS does not already have it installed.

The process can be broken down into several steps:
1. Initially achieving code execution on the console through Stuckpixel’s **ique_cbc_attack** program
2. Dumping your console’s keys and related information with an eSKape payload, as well as your console’s ticket.sys file to be edited
3. Inspecting your console’s firmware version and selecting and installing a System Menu patcher payload accordingly
4. Editing your console’s ticket.sys file with the software to be installed on the system

This guide will work through each step, one at a time. It is assumed that, before following a step, you have correctly followed each preceding step.
It is recommended to read the guide through at least once before proceeding.

Step 1: Initial code execution

In this step, we will use Stuckpixel’s **ique_cbc_attack** program to patch
Dr. Mario (马力欧医生) to achieve code execution.
Before following this step, please ensure that Dr. Mario has been transferred to your console and has been opened at least once.
In this step, your save data for Dr. Mario will be overwritten. Follow [this guide](example.com) (which hasn’t been written yet) to back it up if it is important to you.
Instructions for some parts of this step are specifically intended for Windows 10 64 bit. You may need to change some things for the process to work correctly on your OS of choice.

1. Connect your iQue Player to Windows XP, turn it on and open **ique_diag**. Enter `B` to initialise the connection.
2. Enter `3 005d1870.rec` to dump Dr. Mario’s encrypted game file to Windows XP.
3. Copy **005d1870.rec** to your main OS, into the folder containing **ique_cbc_attack**.
4. On your main OS, open a **command prompt** window and navigate to the folder containing **ique_cbc_attack** and **005d1870.rec** and run this command:
```
ique_cbc_attack -p 3C088001350818E03C09000135298ED0 -r 005d1870.rec -d 081F0000000000000000000000000000 -o 1000
```

You should see this:

```
AES-CBC attack, by stuckpixel
successfully injected 1 blocks!
```



5. Copy **005d1870.rec** from the folder containing **ique_cbc_attack** to the folder containing **ique_diag** on Windows XP.
6. In **ique_diag**, enter `4 005d1870.rec` to write the modified Dr. Mario encrypted game file back to your console.
7. On your main OS, rename **v2_dump.sta** to **005d1870.sta**.
8. Copy **005d1870.sta** from your main OS to the folder containing **ique_diag** on Windows XP.
9. In **ique_diag**, enter `4 005d1870.sta` to write the key dumper payload to your console.
10. In **ique_diag**, enter `Q` to close the connection to the console while keeping **ique_diag** open.
11. Turn off your console and disconnect it from your Windows XP PC.

Step 2: Dumping your console’s keys

1. Turn the console back on (without a USB cable inserted) to boot to the main iQue Menu.
2. Open Dr. Mario (马力欧医生) from the games list (‘游戏’ on the main menu). The game should boot to a black screen.
3. After a few seconds, press the power button on the console **once** to return to the iQue Menu. When the main menu loads, press the power button on the console **once** to turn the console off.
4. Connect your console back to your Windows XP PC with a USB cable and turn it on.
5. In **ique_diag**, enter `B` to initialise the connection, then enter `3 005d1870.sta` to dump the save file containing your console’s keys to Windows XP.
6. Copy **005d1870.sta** from the folder containing **ique_diag** on Windows XP to your main OS, and open it in a hex editor.
7. Copy the bytes from **0x600** to **0x700** to a new file. Save it as **V2.bin** or similar in a known location on your main OS.
8. In **ique_diag** on Windows XP, enter `3 ticket.sys` to dump your console’s ticket file.
9. Copy **ticket.sys** from the folder containing **ique_diag** on Windows XP to a known location your main OS.

Step 3: Installing Jbop’s HackIt Menu patcher

There are several ways to determine your console’s firmware version. For the purposes of this guide, it is assumed that you have not made a NAND backup.
There are exactly two SKSA versions that support USB. Both have the iQue@Home logo displayed in the top left of the main menu, in Chinese (神游在线). If the main menu of your console does not have this logo, it is not compatible with iQue@Home.

Rename hackit_patcher.sta to 005d1870.sta.
Copy 005d1870.sta to the folder containing ique_diag on Windows XP
In ique_diag, enter 4 005d1870.sta to write the System Menu patcher to your console.

Step 4: Editing your console’s ticket.sys file

On your main OS, open ticket.sys_editor.py (using Python).
Click File, Open file, then navigate to the ticket.sys file dumped from your console earlier, and click Open.
As an initial test, choose 塞尔达的传说 from the list on the right. Click the Ticket data tab, then press Ticket ID: to bring up the ticket ID editor. Uncheck the box next to Is trial ticket:, then close the ticket ID editor window.
Click File, Save as, then navigate to a known directory on your main OS, enter hackit.sys as the filename, then press Save to save the edited file.
Copy hackit.sys from your main OS to the folder containing ique_diag on Windows XP.
In ique_diag, enter 4 hackit.sys to write the modified file to your console, then enter Q to close the connection to your console while keeping ique_diag open.
Turn off your console and disconnect it from your Windows XP PC.
Turn the console back on (without a USB cable inserted) to boot to the main iQue Menu.
Open Dr. Mario (马力欧医生) from the games list (‘游戏’ on the main menu). The game should boot to a black screen.
After a few seconds, press the power button on the console once to return to the iQue Menu. When the main menu loads, enter the games list (‘游戏’ on the main menu), and scroll down until you reach Ocarina of Time (塞尔达传说：时光之笛). Near the right-hand edge of the screen, the small box between the block indicator (114) and the icon displaying whether the game is on the console or PC should be red, indicating that the game is no longer a trial. This demonstrates that the patcher worked and the system menu’s signatures have successfully been patched.

...anything else I should add?
