
# **How to Back Up MongoDB Using `mongodump` with `zstd` and SquashFS for Efficient and Reliable Backups**

Backing up MongoDB databases is crucial for data protection and disaster recovery. In this article, we will explore how to efficiently back up a MongoDB database using **`mongodump` with zstd** for fast and compressed backups and **SquashFS** for creating a snapshot that can be restored while maintaining data integrity.

---

## **📋 Prerequisites**

Before we begin, make sure you have the following installed on your system:
- **MongoDB**: The database software.
- **mongodump**: The MongoDB database backup tool.
- **zstd**: The compression tool for faster and more efficient backups.
- **SquashFS**: A compressed read-only filesystem tool.

You can install `zstd` and `SquashFS` with the following commands:

```bash
# Install zstd
sudo apt update
sudo apt install zstd -y

# Install SquashFS tools
sudo apt install squashfs-tools -y
```

---

## **1️⃣ Backing Up MongoDB Using `mongodump` and `zstd`**

### **Step 1: Create a Backup with `mongodump`**

The first step is to dump your MongoDB database using `mongodump`. This command will create a backup of the database in BSON format. By default, it will store the backup in the `/dump` directory.

```bash
# Create a backup of your MongoDB database
mongodump --db your_database --out /tmp/mongo_backup
```

- **`your_database`**: Replace with the name of your MongoDB database.
- **`/tmp/mongo_backup`**: The directory where the backup will be stored.

### **Step 2: Compress the Backup Using `zstd`**

Once the backup is created, we can compress it using `zstd`. This will reduce the file size, making it easier to store and transfer.

```bash
# Compress the backup using zstd
tar -czf /tmp/mongo_backup.tar.gz -C /tmp mongo_backup
zstd /tmp/mongo_backup.tar.gz -o /tmp/mongo_backup.tar.gz.zst
```

- **`tar -czf`**: Creates a tarball (archive) of the backup.
- **`zstd`**: Compresses the tarball with `zstd` into a `.zst` file.

### **Step 3: Verify the Backup**

You can verify that the backup has been compressed properly and check the file size using:

```bash
ls -lh /tmp/mongo_backup.tar.gz.zst
```

The file size should be significantly smaller than the original backup size, thanks to `zstd` compression.

---

## **2️⃣ Backing Up MongoDB Using SquashFS**

SquashFS is another method to take a snapshot of the MongoDB data, which is particularly useful for creating read-only backups that are compressed and easy to store.

### **Step 1: Dump MongoDB Data (Same as Above)**

Use `mongodump` to create a backup of the database:

```bash
# Create a backup of your MongoDB database
mongodump --db your_database --out /tmp/mongo_backup
```

### **Step 2: Create a SquashFS Snapshot**

SquashFS allows us to create a **read-only snapshot** of the database backup, which is ideal for archival purposes. You can specify the compression level (e.g., using XZ, LZ4, or Zstd). Here, we will use XZ compression, which offers a good balance of compression ratio and speed.

```bash
# Create a SquashFS snapshot of the backup
sudo mksquashfs /tmp/mongo_backup /tmp/mongo_snapshot.squashfs -comp xz
```

- **`/tmp/mongo_backup`**: The directory containing the MongoDB backup.
- **`/tmp/mongo_snapshot.squashfs`**: The output compressed snapshot file.

### **Step 3: Verify the Snapshot**

After creating the snapshot, you can check its size:

```bash
ls -lh /tmp/mongo_snapshot.squashfs
```

This compressed snapshot can be stored efficiently, and since it’s a read-only snapshot, it ensures that the data cannot be modified.

---

## **3️⃣ Restoring the Backup**

### **Restoring from `mongodump` with `zstd`**

To restore the MongoDB database from the backup, follow these steps:

1. **Decompress the backup using `zstd`**:

```bash
zstd -d /tmp/mongo_backup.tar.gz.zst -o /tmp/mongo_backup.tar.gz
```

2. **Extract the backup using `tar`**:

```bash
tar -xvzf /tmp/mongo_backup.tar.gz -C /tmp
```

3. **Restore the MongoDB backup**:

```bash
mongorestore --db your_database /tmp/mongo_backup/your_database
```

This will restore the database from the backup.

### **Restoring from SquashFS Snapshot**

To restore from the SquashFS snapshot:

1. **Mount the SquashFS snapshot**:

```bash
sudo mount /tmp/mongo_snapshot.squashfs /mnt -t squashfs -o loop
```

2. **Restore the MongoDB data**:

```bash
mongorestore --db your_database /mnt/your_database
```

3. **Unmount the snapshot** after restoration:

```bash
sudo umount /mnt
```

---

## **4️⃣ Advantages of Each Method**

### **`mongodump + zstd`**
- **Compression Ratio**: `zstd` provides **faster** compression with a **smaller file size**.
- **Speed**: Faster backup and restore compared to other methods.
- **Portability**: Easy to transfer the `.zst` compressed file across systems.
- **Reliability**: High data integrity with no risk of corruption.

### **SquashFS**
- **Snapshot Integrity**: SquashFS creates a **read-only snapshot**, ensuring the data cannot be modified, which makes it perfect for long-term archival.
- **Compression**: Offers great compression using **XZ** or **Zstd**.
- **System Snapshots**: Ideal for capturing an entire file system, not just MongoDB data.
- **Restoration**: Provides efficient restore functionality with minimal overhead.

---

## **Conclusion**

Both **`mongodump + zstd`** and **SquashFS** are excellent methods for backing up MongoDB data, each with its own benefits:

- **Use `mongodump + zstd`** for **fast, efficient backups** that are easy to restore and transfer.
- **Use SquashFS** for **read-only snapshots** that provide **high compression** and **integrity** for long-term archival purposes.

By using these methods, you ensure that your MongoDB data is backed up reliably, with minimal file size, and with no risk of corruption. Choose the best method based on your specific needs for speed, compression, and data integrity.

---

### **Automate Backups with Scripts**

To simplify this process, you can automate backups with the following script:

```bash
#!/bin/bash

# Set variables
DB_NAME="your_database"
BACKUP_DIR="/tmp/mongo_backup"
TAR_FILE="/tmp/mongo_backup.tar.gz"
ZSTD_FILE="/tmp/mongo_backup.tar.gz.zst"
SQUASHFS_FILE="/tmp/mongo_snapshot.squashfs"

# Create MongoDB dump
mongodump --db $DB_NAME --out $BACKUP_DIR

# Compress the backup using zstd
tar -czf $TAR_FILE -C $BACKUP_DIR $DB_NAME
zstd $TAR_FILE -o $ZSTD_FILE

# Create SquashFS snapshot
sudo mksquashfs $BACKUP_DIR $SQUASHFS_FILE -comp xz

echo "Backup complete!"
```

This script will back up the MongoDB database, compress it with `zstd`, and create a SquashFS snapshot.

---
