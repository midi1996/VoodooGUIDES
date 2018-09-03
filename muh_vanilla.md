# Vanilla macOS installer from Windows

This is a sort-of-guide, that will show you how to make a macOS installer on Windows, and keeping it vanilla. This is complementary with /u/corpnewt Vanilla guide, minus some stuff that I dont like like Clover Configurator (ewewewewewewew).

### LAPTOP USERS, ATTENTION!

This guide mainly (and mostly only) **FOR DESKTOP USERS**. Laptop users get to [this guide from rehabman](https://www.tonymacx86.com/threads/guide-booting-the-os-x-installer-on-laptops-with-clover.148093/).

***
### Disclamer:
Neither I nor the people here helping or attempting to help take any responsiblity of your actions. You're following this guide on your own accord and anything that occures as damages are under your resposibility.
### End of Disclamer.
***

# To get started, make sure you:

0) know your hardware to the core. (saying that you have Intel HD Graphics is not an answer, saying that you have a Core i5 is not an answer, you need to know your specific hardware, from the external case [if applicable], down to every component of the computer, namely: CPU - GPU(s) - RAM - Disks - Motherboard - ...)
1) have a 4GB minimum USB drive 
	Note: if you have a rooted android phone, look for DriveDroid, and make sure you have a shared internal storage (no separate /data partition) usually all phones made after 2012 should be like that, so if yours is fairly new it will handle it just fine.
	Note2: use a USB2.0 drive, HDDs may not be a good choice, also if you dont have any USB2.0, plug the USB in a USB2.0 port if available, or use a USB extension cord that doesnt support USB3.0, this way the USB3.0 drive will run in USB2.0 mode.
2) a LAN connection (no wifi, no wifi dongles, Ethernet USB adapter *may* work depending on macOS support) and you must know your LAN card's model [CRUTIAL]
3) a fast internet connection (20Mbps downlink may take about an hour for the install procedure, the faster the better).
4) a Windows environment (can be VM, installed on a real machine, or even WinPE): Windows 7 SP1 or later.
5) some googling skills, which a lot of you lack.
6) brain and patience and reading capabilities [CRUCIAL]

# Ok, for the programs/software needed:

1) cVad's BootDiskUtility from his [official website](http://cvad-mac.narod.ru/index/bootdiskutility_exe/0-5) or [InsanelyMac thread](https://www.insanelymac.com/forum/topic/283190-bootdisk-utility/)
2) Minitool partition wizard [Google]

# For the steps:
Depending on your hardware, you will need to make some modification or choices follwing this guide.

1) Plug the USB in the Windows machine
2) make sure your LAN cable is plugged and there is internet
3) make sure your USB is recognized
4) make sure you dont have anything on your USB
5) make sure you backed up your computer data (dont come cry when you fuck up)
6) Get cVad's BootDiskUtility (we'll call it BDU from now on) and extract it
7) open BDU
8) select your USB device
9) press format
10) When done, press DL Center
11) press Update
12) Select from OS X Recovery list the macOS version you want to install
13) press DL
14) when done and extracted (the software will do that, not you), expand your usb list in BDU, select teh second partition (the first is named clover, no touch!)
15) press Restore
16) choose 4.hfs from the list (it should be inside the extracted BDU folder)
17) wait


**Now your USB has clover and a macOS recovery. For the second part we will create a config.plist, fix driver64UEFI for clover and add kexts.**

1) Open [CCC](http://cloudclovereditor.altervista.org) : Cloud Clover Configurator: an open-source web-based clover configurator, and better than the app in some ways.
2) Create a new config
3) Under ACPI:
	- if you have a 3rd Gen intel Core: select Generate Plugin Type under SSDT
	- if you have a 2nd Gen intel Core: select Generate P-States and C-States
	- Select FixRTC FixTMR FixIPIC under DSDT > Fixes
	- [For people with iGPU] Select the Blue Globe under Patches and choose GFX0 to IGPU
4) Under Boot:
	- Boot Arguments (the big zone): `-v debug=0x100 nv_disable=1 kext-dev-mode=1 dart=0` (these are generic boot args for: Verbose `-v debug=0x100` - Disabling Nvidia drivers from loading `nv_disable=1` - unsigned kexts allowin for 10.10.x `kext-dev-mode-1` - disable VT-d on macOS `dart=0`)
	- XPM detection: NO
5) Under Devices:
	- USB: Inject - Add Clock ID - Fix Ownership
	- Audio: Inject : 1 (type it inside Layout ID)
6) Under GUI:
	- Scan Options: Custom - Scan Entries - Scan Tools - Scan Kernel: Disabled - Scan Legacy: Disabled.
	- [optional] Mouse: Enabled
	- [optional] Screen Resolution - Language - Theme (I recommend Embedded Theme Type: Dark)
7) Under Graphics:
	- If you have Intel HD device, you can try Inject > Intel, if it doesnt work, use a bogus fakeID back under Device sections, under Intel GFX = 0x12345678. Usually intel works OOB on many systems.
	- If you have AMD/Nvidia, DO NOT TICK INJECT (unless you're on Kepler or older on Nvidia, no idea about AMD).
8) Under Kernel And Kext Patches:
	- Apple RTC - Kernel PM
	- Select the Blue Globe in Kernel Patches: add both patches
	- [optional] IF YOU NEED IT, choose a FakeCPU ID (only if you're using an older macOS version on a new hardware system, eg: running Sierra on a CoffeeLake system, choose either KabyLake if it's 10.12.6, or SkyLake id =<10.12.6)
9) Under RT Variables:
	- Booter Config: 0x28
	- Csr Active Config: 0x67 (some may use 0x3E7)
10) Under SMBIOS:
	- Coffee Lake - *iMac18,2/18,3*
		- Use *iMac18,1* if you are using the iGPU only
	- Kabylake - *iMac18,2/18,3*
	- Skylake - *iMac17,1*
	- Broadwell - *iMac16,1* (rarely used, if ever)
	- Haswell Refresh (Devil\'s Canyon) - *iMac15,1*
	- Haswell With NVIDIA GPU - *iMac14,2*
	- Haswell With iGPU - *iMac14,1*
	- Ivy Bridge - *iMac13,2*
	- Sandy Bridge - *iMac12,2* (although recently I've had better success with *iMac13,2*)
	- X79/X99/X299 - *MacPro6,1*
		(~source: Vanilla guide from /u/corpnewt)
11) Under System Parameters:
	- Inject Kexts: Yes
12) Under Boot Graphics:
	- [optional] for high resolution pannel users (3k~4k, 1440p...) change UI Scale to 2
13) Select the two squares on the top right corner
14) Select Download
15) name: `config` (no extension)
16) Download
17) Open the plist with a text editor like: Notepad++, Geany, Sublime Text. DO NOT USE WINDOWS NOTEPAD (until release 1809 anyways).
18) Under
```xml
	<key>DSDT</key>
	<dict>
```

add

```xml
		<key>AutoMerge</key>
		<true/>
		<key>FixHeaders</key>
		<true/>
```
19) Save
20) Copy the resulting plist file and paste it in CLOVER (partition)> EFI > CLOVER and replace the one already there.

### Fixing your clover install now:
1) delete `doc` and `OEM` and `drivers64` (for UEFI users), or `drivers64UEFI` (for legacy users).
2) for UEFI users:
	- Open `drivers64UEFI`, *deled* everything inside
	- go to `drivers-off > drivers64UEFI`, copy ApfsDriverLoader and AptioMemoryFix to `drivers64UEFI` that we emptied earlier.
	- Download (HFSPlus.efi)[https://github.com/JrCs/CloverGrowerPro/blob/master/Files/HFSPlus/X64/HFSPlus.efi]
	- Put it in drivers64UEFI
3) Go to kexts > Other
	- Go to (Goose's Kext Repo)[https://1drv.ms/f/s!AiP7m5LaOED-mo9XA4Ml-69cwAsikQ]
	- Download these:
		- FakeSMC
		- Lilu
		- WhateverGreen
		- USBInjectAll
		- AppleALC
		- [optiona, for PS2 devices] VoodooPS2 (2018, doesn anyone still uses PS2 on their desktops, pff)
		- For your LAN card:
			- AppleIntele1000 for some old cards
			- IntelMausiEthernet for most Intel NICs
			- AtherosE2200Ethernet for Atheros/QualcommAtheros/Killer(some) NICs
			- BCM5722D for Broadcom BCM5722 NetXtreme and NetLink family
			- RealtekRTL8100 for 10/100 Realtek Cards
			- RealtekRTL8111 for Gigabit Realtek Cards
			Note: if you're not sure, get all of them, but it may create issues later on.
4) Extracte every zip
Note: a kexts is a macOS driver, and it's in a form of a `a_folder_name.kext`, on windows it will show as a folder, on macOS it will show as a file.
5) now copy FakeSMC.kext - Lilu.kext - WhateverGreen.kext - USBInjectAll.kext - AppleALC.kext - [your_ethernet_driver].kext and put it in CLOVER > EFI > CLOVER > kexts > Other. [skip the sensor kexts, they may cause kernel panics, aka KP]
6) now you're mostly done with Clover and macOS installer preparation.

### Firmware changes

Now depending on your computer motherboard make sure of:
- XHCI Handoff: Enabled (if applicable, some may not have this option)
- OS Type: Other (if applicable)
- Secure Boot: Disabled (refer to google or your motherboard's manual on how to disable it)
- Legacy/CSM support: Disabled [if you see a garbled screen, enable this]
If you have an Nvidia GPU:
- NO DISPLAY is connected to the motherboard's DP/HDMI/VGA/DVI ports
- under System Agent (SA) [or some other menu], Graphics Settings, Main Display: PEG or PCIE
- Disable anything like: Hybrid Graphics, Dual Graphics, DVMT size ... that are related to Intel GPU
If you have Intel GPU:
- make sure your DVMT pre-alloc (not to be confused with DVMT size), is set to 64MB (or higher, 64 is enough), DMVT size/apertures/whatever, to the max.

### Now you're ready to plug-n-play.

To start the installation:
1) Plug the Ethernet
2) plug the usb
3) start the computer into the boot menu (refer to your motherboards manual or google)
4) select your USB device (make sure it says UEFI or PX before it, some motherboards with only legacy may not find UEFI, it's fine, tho I'm not supporting Legacy booting atm).
5) select Boot macOS Recovery from ...
6) wait for the wall of text to make you feel like a hackerman
	Note: if it gets stuck with a Stop sign, change your USB drive's port, use a USB2.0 drive, or use a USB2.0 extension cord.
	Note2: if you get a black screen at then end of the installer on Intel HD GPUs, reboot to Clover, Press `O` (letter), go to Graphics Injection, change FakeID to 0x12345678, go back and boot. You will have to do this on every reboot until you get to the desktop. The Graphics will be slow and sliggish, fix it asap. Use google then.
7) when you get to the Installer screen (it may take several minutes for it to load), open Utilities > Network Utility. Check if there is a connection, if not, check your LAN cable, and your LAN drivers from above.
8) Go back, select Disk Utility, select the drive to format, nuke it as MacOS Extended Journaled, name it something and go back.
	- Note: if you want to multiboot, go [here](Multiboot.md)
	- Note2: DATA == GONE, BYE
9) Click on Disk Utility on top > Quit Disk Utility.
10) Select Reinstall macOS
11) YES YOU AGREE
12) Select your disk
13) Let it install (the faster the internet the better).

After that, it will reboot for the second stage of the install, boot clover, it should now autoselect Boot `Install macOS from <your hdd name>`, if not, then select it on your own and boot it up, let it finish.

Boom now you have macOS installed. Go to the bottom part of the Vanilla guide to get more information and fixes and extra elements for your config.

This guide will still be updated for additional hacks and information.