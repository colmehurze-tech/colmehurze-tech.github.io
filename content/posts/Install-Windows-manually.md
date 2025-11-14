---
title: "Install Windows Manually"
date: 2025-11-14T19:31:07+05:30
draft: false
---

## How to manually install windows on a disk drive using diskpart and bcdboot

### Q. Why do this?
Sometimes you need to manually partition your drive and install windows, either because windows won't let you install normally or simply because your pc "does not support windows 11" (All hail microsoft!)

### What you'll need:

1. A bootable windows install media (The process is very trivial so I'm skipping it)
2. Basic knowledge of Windows CMD
3. A little patience :)

Note: If you're planning on upgrading to Windows 11 on an unsupported pc without losing data, this method is not for you, as ALL DATA WILL BE ERASED on your device!

### Step 1: Booting and starting diskpart

1. Boot from your windows install media. This process should be fairly easy, so I'm not going into further details.
2. Press Shift+F10 on your keyboard to open cmd.
3. Enter the following command: 
```batch
diskpart
```

We have successfully started diskpart. This is the built-in tool for manually partitioning your disks.

### Step 2: Partitioning the Drive

Be careful with this one. If you delete/format the wrong partition, you may lose data on your disk!

1. List the available disks using the command:
```batch
list dis
```
You should see all your disks in the output section. If you're only using one disk, your disk is usually "disk 0". 

2. Select your preffered disk by executing:
```batch
select disk <disk number>
```

3. Clean the selected disk and convert it to GPT (works even on MBR systems):
```batch
clean
```
```batch
conv gpt
```

4. Create, format and assign a drive letter to the Windows Recovery Partition:

Windows requires a recovery partition to boot properly, and it's size is around 512MB, so we will create an EFI partition of said size and format it.
```batch
create par efi size=512
```
```batch
format fs=fat32
```
We also need to add a drive letter to the partition. Do it by using:
```batch
assign letter <any capital letter>
```
I have not included a drive letter in this code to prevent confusion with the user's internal drive letters. In short, users can select any drive letter that is not already occupied by an existing parititon.

5. Create and format the primary windows partition:

We will now create the primary partition in which our new Windows installation will reside. If you want to devote your entire disk to windows, use:
```batch
create par primary
```
If you want to allocate a specific amount of space to your Windows installation, use:
```batch
create par primary size=<mention size in megabytes>
```
Regardless of which command you use, format the newly created partition in the NTFS filesysytem, and assign a drive letter:
```batch
format fs=ntfs quick
```
```batch
assign letter <any capital letter>
```
Usually, the Windows installation partition is assigned the drive letter "C", but to avoid conflict with the user's drive letters, I have not included this in my code.

Next, simply type "exit" in the diskpart prompt to exit DISKPART.

6. Install Windows using dism:

Windows has a built-in imaging tool that allows you to Install Windows to a specified partition. But first, we need to select which version of Windows we want to install:
```batch
dism /Get-ImageInfo /imagefile:D:\sources\install.wim
```
Note: Please replace the letter "D" with the drive letter of your windows install media. I included "D" as default since it's the most common drive letter for windows install media.

After executing this code, DISM will show you the index number of the Windows edition you want to install. Say, you want Pro, which is usually index number 6. Now, execute:
```batch
dism /Apply-Image /ImageFile:D:\Sources\install.wim /index:6 /ApplyDir:C:\
```
Note: Please replace the letter "C" with the drive letter of your windows primary partition (The one we created in step 5). I included "C" as default since it's the most common drive letter for windows primary partition.

Wait patiently for Windows to install onto your specified partition. 

6. Almost Done! Installing the bootloader:

We are almost done with manually installing Windows onto our disk. We need to install the windows bootloader next:
```batch
bcdboot C:\Windows /s S: /f ALL
```
Note: Once again, please replace the drive letters with the corresponding letters in your case.

After this command successfully executes, you're all set! Exit the command prompt, and restart your system. This time, do not boot from the Windows installation media, but from your disk drive. You should be greeted with the OOBE wizard.
