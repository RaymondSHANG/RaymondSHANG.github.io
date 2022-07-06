---
layout: post
title: "Windows ubuntu duo system"
subtitle: "efi_boot"
date: 2022-07-01 13:59:20
header-style: text
catalog: true
author: "Yuan"
tags: [Windows,Ubuntu,Duo boot]
---
{% include linksref.html %}
> <b>corinthians,12:9-10:</b> But he said to me, "My grace is sufficient for you, for my power is made perfect in weakness." Therefore I will boast all the more gladly about my weaknesses, so that Christ's power may rest on me. 10That is why, for Christ's sake, I delight in weaknesses, in insults, in hardships, in persecutions, in difficulties. For when I am weak, then I am strong.

I bought a new PC: Dell Precision3650 Tower for deep learning and large scale computing. This PC has 2 SSD harddrive (256GB + 1TB) and 1 HDD harddrive (4TB). 


In this poster, I will only focus on how the duo boot systems are installed, for my own record. There are a lot tutorials on line, and some of those are really confusing, mainly due to the mixing of UEFI/GPT and Bios/MBR. I found a good one here (win10 +ubuntu20.04双系统安装：双硬盘+nvidia独立显卡_柚子=_=的博客)[https://www.cxyzjd.com/article/weixin_41878226/107386606].

A general process to install the dua boot system:

1. Install Windows on one SSD, using UEFI/GPT. If the hardisk is bitlockedm(BitLocker encrypted disk), you need to save the recovery ID and passport code. Otherwise, you could not log back after ubuntu is installed.
2. Make a USB boot Ubuntu 20.04 disk.
3. Set EFI, boot from USB.
4. "Normal installation", "Something Else..."
5. Select place to install the system.
6. My partitions:
   * For the 1024G SSD harddisk
  
    | Partition | File System | Mount     | Size            | Primary/Logical |
    | :-------- | :---------- | :-------- | :-------------- | :-------------- |
    | EFI       | ESP         | /boot/efi | 1024MB          | Primary         |
    | OS        | EXT4        | /         | 100GB           | Logical         |
    | User Home | EXT4        | /home     | 900GB           | Logical         |
    | SWAP      | SWAP        |           | Remaining ~13GB | Logical         |
    
    * For the 256GB SSD harddisk:
      NTFS, Windows 10, C
    * For the 4TB HDD harddisk:
      NTFS, 500 GB for other documents(Volume D:)
      NTFS, Remaining 3.5TB for data.

To best understand the installation process, we need to know the boot procesure， from (Vaibhav Kandwal)[https://www.freecodecamp.org/news/uefi-vs-bios/]:

1. You press the power button on your laptop/desktop.
2. The CPU starts up, but needs some instructions to work on (remember, the CPU always needs to do something). Since the main memory is empty at this stage, CPU defers to load instructions from the <b>firmware chip</b> on the motherboard and begins executing instructions.
3. The firmware code does a Power On Self Test (POST), initializes the remaining hardware, detects the connected peripherals (mouse, keyboard, pendrive etc.) and checks if all connected devices are healthy. You might remember it as a 'beep' that desktops used to make after POST is successful.
4. Finally, the firmware code cycles through all storage devices and looks for a boot-loader (usually located in first sector of a disk). If the boot-loader is found, then the firmware hands over control of the computer to it.

UEFI and Bios(or Legacy) are two such firmware interfaces used by all most all personal computers.
To begin with, we need to know the latest UEFI

{{tip}}
<b>UEFI/GPT vs Bios(Legacy)/MBR</b></br></br>
1. UEFI is short for Extensible Firmware Interface (EFI) or its version 2.x variant, Unified EFI (UEFI)</br></br>
2. <b>UEFI</b> match with <b>GPT</b> harddisk partitions, which support > 2.2TB (To 9 million TB) spaces of your harddisk. <b>Bios</b> matches with <b>MBR</b> harddisk partitions, which only support less than 2.2TB harddisk space.</br></br>
3. If you installed all OS in the <b>same</b> harddisk, you only need EFI installed in that harddisk. If you installed your OS in two different harddisks, you need EFI installed in both harddisks.</br></br>
4. During the POST procedure, the UEFI firmware scans all of the bootable storage devices that are connected to the system for a valid GUID Partition Table (GPT).The UEFI firmware scans the GPTs to find an EFI Service Partition to boot from. If the EFI bootable partition is not found, the firmware may revert to the old Legacy Boot method. If both UEFI boot and Legacy boot fail, you may receive the disk boot failure error message. </br></br>
5. UEFI is faster than Bios. </br></br>
6. UEFI offers security like "Secure Boot", which prevents the computer from booting from unauthorized/unsigned applications. This helps in preventing rootkits, but also hampers dual-booting, as it treats other OS as unsigned applications. You may need to disable secure boot to install your duo systems. (Currently, only Windows and Ubuntu are signed OS??Not sure)</br></br>
7. During ubuntu installing process, if you use the manual partitioning ("Something else"), the difference is that you will have to set the /boot/efi mount point to the UEFI partition. And if there was not any UEFI partition on your HDD, you first will have to create it.
{{end}}

This PC also has a RTX3090 24GB vedio card, 64GB memory, with Intel i9-11900K CPU. The total power is 1000W. In the next poster, I will update the configurations of pytorch and cuda for this computer, which is tricky.

---
