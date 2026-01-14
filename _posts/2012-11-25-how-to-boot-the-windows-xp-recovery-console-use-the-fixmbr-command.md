---
title: "How to use fixmbr in the Windows XP Recovery Console"
---
The Windows XP Recovery Console is either an installable start-up option for a computer running Windows XP or a bootable media included with the Windows XP install disc. This tutorial details how to load the Recovery Console from the original Windows XP install disc.

The `fixmbr` command can be used in the Windows Recovery Console to re-install the default master boot record so that Windows can be successfully booted. For example, you will need to use this command to remove the Linux installation from a Windows/Linux dual-boot system. On a Windows/Linux dual boot, Linux overwrites the default Windows boot loader with Grub so that both operating systems can be booted from a start-up menu. However, if you remove Linux by simply deleting the Linux partitions, the files that contain Grub will be deleted and you will be greeted with a "Grub rescue" message when you try to boot your computer. Running the `fixmbr` command should solve this problem. Another reason to you would need to use `fixmbr` is if your master boot record has been corrupted for any reason and your system won't boot.

Anyway, here's an explanation - a detailed one - of how to boot into the Windows Recovery Console if you can't boot Windows XP and how to then run `fixmbr` to repair the master boot record.

1. Find your original Windows XP install disc. Yes, you will need this if you can't boot your system.
2. Set your computer's BIOS to boot from the CD, insert the CD, and boot from it. If you don't know how to do this, find your computer's documentation and look up how to access and use the BIOS.
3. You should now see a "Press any key to boot from CD" message. Press any key.
    ![Windows XP fixmbr Step 1](/assets/images/fixmbr1.jpg)
4. Now, a shockingly blue Windows Setup screen will open and start flashing text by on the bottom of it. Don't worry, this is not installing anything or overwriting anything. It's simply scanning your computer and loading the Windows XP setup program.
    ![Windows XP fixmbr Step 2](/assets/images/fixmbr2.jpg)
5. Once Windows XP setup loads, press "R" to open the Recovery Console.
    ![Windows XP fixmbr Step 3](/assets/images/fixmbr3.jpg)
6. You should now see a prompt asking you to select the Windows installation that you would like to log on to. If you don't see any options here, your system is ruined and is undetectable by the Recovery Console. If you do see option(s) here, type the NUMBER of the option (not the drive letter or something).
    ![Windows XP fixmbr Step 4](/assets/images/fixmbr4.jpg)
7. You will be prompted for the administrator password. If you don't have one, just press Enter.
    ![Windows XP fixmbr Step 5](/assets/images/fixmbr5.jpg)
8. Now you're in the Recovery Console! You can type `Help` for a list of available commands or type `Help commandname` to see options for a specific command.
    ![Windows XP fixmbr Step 6](/assets/images/fixmbr6.jpg)
9. If you want to fix the master boot record, type in `fixmbr` with no switches and press Enter. The command shouldn't take long to run, and when it has finished type `EXIT` and press Enter. When the computer reboots, I would suggest you immediately enter the BIOS so that you can reset the boot menu and remove the CD. Otherwise, your computer will boot to a "Press any key to boot from CD..." message. Once you've finished with the boot menu, exit the BIOS and try to boot from the main hard drive.
10. Hopefully, your Windows XP installation is now bootable. If it isn't, something beyond the scope of fixing the master boot record has gone wrong and you will need to take more drastic methods to recover your data. This could be removing the hard drive and hooking it up to another computer to see if the files are still accessible.
