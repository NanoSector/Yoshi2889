---
title: "Windows 10 on old Macs without Boot Camp"
date: 2021-02-28T20:09:04+01:00
draft: true
tags: ["howto", "software"]
showtoc: true
description: "Mac touchpads are weird and cause issues when missing."
todo: [
    "add images"
]
---

Installing Windows 10 on older Macs is still possible but may be tricky
because Apple doesn't officially support this. Boot Camp Assistant has an internal
list of Windows 7-only Macs in its Info.plist. With some hackery and foolery you therefore
are able to trick it into installing Windows 10 on an otherwise unsupported Mac. Partially.

I prefer to take the manual route and taking full control over this process. There are
a number of hurdles to overcome.

# Preface: The lazy route
A little trick I often use to skip some steps actually requires a Windows 7 DVD. Doesn't 
matter which one, it just has to be recognised by Boot Camp Assistant.

The reason for this is that Boot Camp Assistant will not proceed to partition your disk
without a valid Windows 7 DVD inserted. Insert one, fire up the Assistant, and proceed all
the way through partitioning the disk as if you would install Windows 7.

Make sure to **stop** when it asks for an administrator password to install a "helper tool".
Cancel this prompt and close the Boot Camp Assistant.
At this point, your disk is properly partitioned, and you can skip to step 4 in this guide.  

Sidenote: While you can tick the box to download Windows 7 drivers, do not just install them
as they come. This will be covered later in the guide.

# Prerequisites
1. Your target Mac
2. A computer running Windows
3. A USB stick with a Linux distro of your choice
4. A way to download or transfer the Boot Camp support software packages 

If you do not already have a Windows 10 DVD:
1. NLite (the free edition is good enough)
2. A Windows 10 ISO
3. A blank DVD

# Hurdle 1: Fitting Windows 10 onto a DVD
Oh boy, Microsoft, what have you done. Windows needs to go on a diet, and fast. Being
obese like this is unhealthy.

Jokes aside, DVDs are 4.7 GB and Windows is well over 5 GB at this point. We can luckily trim
it down.

Let's start by discussing the reason we need Windows onto a DVD. This is because Macs are designed 
to boot USB devices natively in EFI mode, however any DVD can boot in what is called CSM mode;
Compatibility Support Module. This essentially acts as a legacy BIOS emulation layer.
The Mac bootloader marks this partition as "Windows", and its font is slightly off from
other entries.

The reason we need Windows in CSM and therefore on a DVD is because some of Apple's drivers do
not work correctly in EFI mode, most notably the NVIDIA and AMD/ATI GPU drivers which cause various
issues. Booting from USB won't get us into CSM.

Right, trimming the fat. YMMV, but using NLite to trim down the ISO by removing excessive editions
got my ISO below 4 GB. The steps are fairly simple:

1. Mount the Windows 10 ISO by double clicking it,
2. Create a folder (for example on your desktop) and copy the files from the ISO in it,
3. Open NLite and add the folder as a loaded image,
4. From NLite's home screen, unfold the now loaded image so that you can see the editions,
5. For every unwanted edition, right-click it and choose `Delete`. I kept only Home and Pro,
6. Right-click the top level folder for this image tree and select `Create ISO`,
7. You're done! Save this ISO somewhere.

Burn the resulting ISO to a DVD if it fits. If not, cut some more features and editions.

# Hurdle 2: Partitioning
*Note*: All these steps assume you're starting from a clean slate with only 1 macOS installation
and no other OSes on the same drive. If this is not your case, YMMV.

TLDR: Create an exFAT (or NTFS) partition. 

On your Mac, open Disk Utility. Make sure to select `Show all devices` from the View menu.
This stupid default design warrants a whole different rant, but luckily that's the fix.

Now:
1. Select your boot drive,
2. Select `Partition`,
3. Click the Plus icon and adjust the partition size for what you want Windows to take up,
4. Set Format as exFAT,
5. Give it an easy to recognise name. Boot Camp Assistant would've named it `BOOTCAMP` so that's what I'm rolling with,
6. Select `Apply`,
7. Sit back and hope Disk Utility does its job.

This might take a while depending on how fragmented your disk is (it might have to move chunks
of data around).

The Format does not truly matter, but it makes our life easier later on. If you're following along on Linux,
just create an NTFS partition like you normally would.

That's the end of easy mode.

# Hurdle 3: Hybrid MBR/GPT
The fun begins here. You'd really wish you had Boot Camp Assistant at this point.

There is an excellent guide on this: [https://www.rodsbooks.com/gdisk/hybrid.html](https://www.rodsbooks.com/gdisk/hybrid.html)

Do read it, but it boils down to the following steps. Make sure to get them right to
not accidentally cause damage to your other partitions.

1. Boot your Linux USB stick
    - If this fails with a black screen, add `nomodeset` to your kernel parameter list
2. Open your disk in `gdisk`, for example: `gdisk /dev/sda`
3. Open the recovery/transformation menu: `r`
4. Enter the `p` command and check what number your freshly created exFAT/NTFS partition has
5. Create a hybrid MBR by entering `h`
    1. Enter the number of your new partition when it asks for one
    2. Answer Yes (`y`) when it asks about putting an EFI GPT partition first
    3. Accept the default MBR hex code by pressing enter
    4. Answer Yes (`y`) when it asks to set a bootable flag
    5. Answer No (`n`) when it asks about unused partition space
        - Make SURE not to accidentally answer Yes here as macOS gets confused without
          free space in between partitions!
6. Confirm your changes by viewing them with the `o` command
7. If all looks good, write the changes with `w`

Reboot your Mac once this is done. If you head into the boot menu, you should see that an option
for the weird-font `Windows` has popped up. This won't work for now.

# Hurdle 4: Install Windows
Pop in your Windows DVD. Wait for the installer to boot, this might take a long while.

Follow its instructions and select the partition created in step 1 as your installation target.

Format this partition using the options at the bottom of the window. Then click Next to continue the installation.

Everything plays out as normal here. The only inconvenience is that your Mac will default to booting
macOS since Boot Camp Assistant would also set the startup disk correctly.
You can fix this in System Preferences -> Startup disk.

# Hurdle 5: Installing drivers & Boot Camp Control Panel
Thought creating a hybrid GPT was bad?

Your Boot Camp installer downloaded from Boot Camp Assistant will not work on Windows 10.
Apple has conveniently locked this to the designated OS version, usually Windows 7.
You can't run MSIs in compatibility mode, only msiexec if you truly desire.

But this does not matter. The drivers aren't packaged in this installer, and you can
install these separately, no problem. Hold your horses before clicking on every item like
a madman though, Windows 10 has support for a lot of hardware out of the box.

What I like to do is let Windows figure out the majority of drivers. It'll usually find more
up-to-date drivers for major things like your GPU, wireless, etc.

## 5.1: What drivers are you talking about?
If you did not obtain drivers from Boot Camp Assistant, you can still obtain them using
[a tool called brigadier](https://github.com/timsutton/brigadier).
At the time of writing this tool has not been updated in years but it still works perfectly fine.

Make sure to install [7-zip](https://7-zip.org). Brigadier requires it to unpack the Boot Camp .dmg files.

1. Download the latest .exe from the Releases page
   
2. Open a PowerShell window in the directory where you put it
   
3. Enter the following: `.\brigadier.exe`
    - If you're running on a Mac, it'll automatically find your Mac model and download the
      latest Boot Camp support software. If not, append your model like so: `.\brigadier.exe -m iMac11,1`
      
4. Take note of its output:
    `Making directory [...]/BootCamp-041-89042`

5. Also download the Boot Camp support package for a Mac which natively supports Windows 10.
    - I like to use my own MacBook Pro for this, so my command becomes `.\brigadier.exe -m MacBookPro14,2`
   
6. Open Device Manager and look for unknown devices
   
7. Install drivers for any unknown devices from the directory Brigadier mentioned from the first command
   
8. **DO NOT SKIP THIS STEP**: Copy the `Drivers/Apple/BootCamp.msi` file *from the package for the newer Mac* 
   to a different folder, like your desktop.
    - The point of this is to prevent it from launching the installers for other drivers.
    - This might therefore throw an error for 'an installation package' which cannot be opened. Ignore this,
    the installer should proceed.

9.  Run the `BootCamp.msi` installer from its new home

10. Run the `AppleSoftwareUpdate.msi` installer for the Apple Software Update utility
    - Considering your Mac wasn't officially supported to begin with, this might not be all that useful.
      
## 5.2: Problems I've experienced

- I have no audio!
    - Install Apple's audio drivers, usually under the Cirrus directory

- What bluetooth package do I need?
    - In my experience, usually AppleBroadcomBluetooth. If in doubt, try them until one works.

- Where's brightness?
    - Apple doesn't implement hotkeys in Windows, you'll have to use the Boot Camp control panel for this.

- My trackpad doesn't work
    - Replace Apple's drivers with the excellent [open source Mac precision drivers](https://github.com/imbushuo/mac-precision-touchpad)
    - These drivers may feel wiggly. Not sure how to fix that, but `defuzz` is on the to-do list.
    Other than that they feel superior to the Apple drivers.
      
# Hurdle 6: Conclusion
By now you should have a (mostly) functional Windows installation on your system. As hardware gets older,
support for it deteriorates, even in Windows.

I hope this article was useful in providing a relatively complete guide to install Windows on your old Mac.
If you like throwing unsupported software at your Mac, check out [dosdude1's patchers](http://dosdude1.com/software.html).

Linux might be another option, however support for many Macs is flaky at best.

If you have any suggestions or questions, feel free to open an issue or pull request on [the repository for this site](https://github.com/Yoshi2889/Yoshi2889).

Thanks for reading all the way through!