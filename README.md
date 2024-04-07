# restore-synology-raid-data


**Disclaimer:** The following guide involves operations that can lead to data loss if not executed carefully. Ensure you have backups of all important data before proceeding. Use this guide at your own risk, and consider consulting a professional if the data is critical or if you're unsure about any steps.

### Complete Step-by-Step Guide

#### 1. Preparing the Environment

-   Use Rufus on a Windows machine to create a bootable Ubuntu USB drive. Follow Rufus's prompts with your selected Ubuntu ISO to create the drive.
-   Boot your PC from this USB drive by selecting it as the boot device in your PC's BIOS/UEFI settings.

#### 2. Removing NAS Drives

-   **Safely Shut Down NAS**: Ensure your Synology NAS is properly shut down to avoid data corruption.
-   **Remove Drives**: Open the NAS drive bays and carefully remove each HDD. Note their original order and positions for reinstallation later.

#### 3. Connecting Drives to a Separate Recovery PC

-   **About the Recovery PC**: Use a separate PC that can boot into Ubuntu via the live USB. This PC will be used for data recovery, and it should have enough ports or connections to attach all NAS drives simultaneously, using SATA to USB adapters or direct SATA connections.
-   **Connect NAS Drives**: Attach the removed NAS drives to the recovery PC. Make sure all drives are securely connected and recognized by the system.

#### 4. Identifying the Drives

-   In Ubuntu, open a terminal and list the connected drives and partitions:
        
    `lsblk` 
    
-   Identify your NAS drives in the list based on size, model, or expected partition structure.

#### 5. Installing Necessary Tools

-   Install `mdadm` and `lvm2` for RAID and LVM management:    
    
    `sudo apt update && sudo apt install mdadm lvm2 -y` 
    

#### 6. Assembling the RAID Array

-   Attempt to automatically assemble the RAID array:
        
    `sudo mdadm --assemble --scan` 
    
-   Check the RAID assembly status with:
        
    `cat /proc/mdstat` 
    

#### 7. Activating Logical Volumes

-   Activate all detected LVM volume groups:
        
    `sudo vgchange -ay` 
    

#### 8. Identifying the Filesystem

-   Use `blkid` to identify the filesystem type on the logical volume:
        
    `sudo blkid` 
    

#### 9. Checking Filesystem Integrity (Optional)

-   If the filesystem is Btrfs and suspected of being corrupted, check its integrity:
        
    `sudo btrfs check /dev/mapper/vg1000-lv` 
    
-   **Optional Repair (Use with Caution):** For repairable `btrfs check` errors, use:
        
    `sudo btrfs check --repair /dev/mapper/vg1000-lv` 
    
    **Warning:** The `--repair` option can result in data loss. Use it only as a last resort and preferably with expert consultation.

#### 10. Preparing the Destination Drive

-   Connect a 6TB USB HDD to the recovery PC and identify it using `lsblk`.
-   If necessary, format the USB HDD (after ensuring no needed data is on it):
        
    `sudo mkfs.ext4 /dev/sdx2  # Replace 'sdx2' with the correct identifier` 
    
-   Mount the USB HDD:
        
    `mkdir /mnt/usbhdd`
    
    `sudo mount /dev/sdx2 /mnt/usbhdd  # Replace 'sdx2' with the correct identifier` 
    

#### 11. Data Recovery

-   Start the recovery process with:
        
    `sudo btrfs restore -iv /dev/mapper/vg1000-lv /mnt/usbhdd/` 
    
-   Monitor the recovery and check the amount of data recovered:
        
    `du -sh /mnt/usbhdd/` 
    

#### 12. Post-Recovery Actions

-   Verify the integrity of the recovered data.
-   To reuse the NAS drives:
    -   **Option 1:** Reinsert the drives into the Synology NAS in their original order and perform a factory reset via DSM, which includes drive formatting and DSM reinstallation.
    -   **Option 2:** Optionally, format the drives before reinserting them into the NAS, although this is generally not necessary unless addressing specific issues.
