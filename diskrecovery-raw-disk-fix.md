
# How to Fix a RAW Disk and Reformat it to exFAT or NTFS Using Diskpart

A RAW disk occurs when a drive’s file system becomes unreadable by the operating system. This can happen due to corruption, accidental formatting, or partition issues. In Windows, a RAW disk is inaccessible, but you can fix it by cleaning the disk and reformatting it to a recognized file system like **exFAT** or **NTFS**. This article walks you through using **Diskpart**, a built-in Windows tool, to fix a RAW disk and reformat it for use.

## Understanding the Problem: What is a RAW Disk?

A RAW disk is essentially a drive that lacks a recognizable file system. Common causes include:

- **Corrupted File System**: Power failures or unsafe ejection may corrupt the file system.
- **Accidental Formatting**: An incomplete or interrupted format process can result in a RAW disk.
- **Partition Issues**: Deleting a partition or failing to create one properly leaves the disk in a RAW state.

### Symptoms of a RAW Disk:
- The disk shows up in **File Explorer** but is inaccessible.
- The file system is listed as "RAW" in **Disk Management**.
- You get errors like **"The volume does not contain a recognized file system."**

## Fixing a RAW Disk Using Diskpart

Diskpart is a command-line utility in Windows that allows you to manage disks and partitions. We’ll use Diskpart to clean the disk, create a new partition, and format it to either **exFAT** or **NTFS**.

### **Step 1: Open Diskpart**

1. Press `Win + X` and select **Command Prompt (Admin)** or **Windows PowerShell (Admin)**.
2. In the Command Prompt, type `diskpart` and press **Enter**.

Diskpart will now start, and you’ll see a prompt ready to receive commands.

### **Step 2: List and Select the Disk**

You need to identify the correct disk to work on. To list all the connected disks:

```bash
list disk
```

You will see a list of disks attached to your computer, something like this:

```
Disk ###  Status         Size     Free     Dyn  Gpt
--------  -------------  -------  -------  ---  ---
Disk 0    Online         2794 GB  1024 KB        *
Disk 1    Online          119 GB  1024 KB        *
Disk 2    Online          931 GB      0 B
```

Find the disk that corresponds to the RAW drive (by size), for example, **Disk 2**.

To select the disk:

```bash
select disk 2
```

Diskpart will confirm the selection with the message: **"Disk 2 is now the selected disk."**

### **Step 3: Clean the Disk**

Cleaning the disk will remove all partitions, turning the disk into unallocated space. Be aware that this will delete all data on the drive.

To clean the disk:

```bash
clean
```

Diskpart will now wipe the drive, removing any corrupted partitions or file systems.

### **Step 4: Create a New Partition**

After cleaning the disk, you need to create a new partition. This partition will be formatted in the next step.

To create a new primary partition:

```bash
create partition primary
```

This will allocate all the space on the disk to a primary partition.

### **Step 5: Format the Partition to exFAT or NTFS**

Now that the partition is created, format it to the file system of your choice. You can choose **exFAT** (ideal for cross-platform usage between Windows, macOS, and Linux) or **NTFS** (Windows-only but with features like file compression and security).

- To format to **exFAT**:

```bash
format fs=exfat quick
```

- To format to **NTFS**:

```bash
format fs=ntfs quick
```

The `quick` option performs a fast format, skipping the check for bad sectors. If you suspect the drive may have bad sectors, you can omit `quick` for a more thorough format, but it will take longer.

### **Step 6: Assign a Drive Letter**

To make the new partition accessible in **File Explorer**, you need to assign it a drive letter (for example, `E`):

```bash
assign letter=E
```

After assigning the letter, the drive should appear in File Explorer under the letter `E:` (or whatever letter you chose).

### **Step 7: Exit Diskpart**

Finally, exit Diskpart by typing:

```bash
exit
```

Diskpart will close, and your disk is now properly formatted and ready for use.

---

## Troubleshooting Tips

### 1. **Disk is Still Inaccessible After Formatting**

If the disk is still inaccessible or fails to format, it might indicate a hardware issue. Use tools like **CrystalDiskInfo** to check the drive’s SMART status and determine if the drive is experiencing hardware failure.

- **CrystalDiskInfo**: [Download Link](https://crystalmark.info/en/software/crystaldiskinfo/)

### 2. **Using Third-Party Tools**

If Diskpart doesn’t work, you can use third-party partitioning software like:

- **MiniTool Partition Wizard**: [Download Link](https://www.partitionwizard.com/)
- **EaseUS Partition Master**: [Download Link](https://www.easeus.com/partition-manager/)

These tools provide a graphical interface and can sometimes bypass system restrictions to format stubborn RAW drives.

### 3. **Check for Bad Sectors**

If the drive has physical damage (e.g., bad sectors), it may fail during formatting. You can check for bad sectors using **chkdsk** or by running a full format without the `quick` flag.

```bash
format fs=exfat
```

---

## Conclusion

Fixing a RAW disk is a straightforward process using **Diskpart**. By cleaning the drive, creating a new partition, and formatting it to a usable file system like **exFAT** or **NTFS**, you can bring your drive back to life. However, always make sure to back up your data before proceeding, as formatting will erase all files.

If you run into hardware issues or if Diskpart doesn’t resolve the problem, third-party tools or professional recovery services may be required to retrieve data or restore the drive.

By following these steps, you should be able to fix most RAW drive issues and reformat the drive for use in your system.

---

Feel free to reach out if you encounter any specific issues or need further assistance!
