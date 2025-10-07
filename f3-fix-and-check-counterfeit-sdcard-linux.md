# How to Detect and Fix Counterfeit SD Cards in Linux

Counterfeit SD cards are a frustrating reality. They often claim to hold far more storage than they actually do — for example, a “128 GB” card may only contain 8 GB of real memory. The rest is virtual capacity faked by the card’s firmware. When you fill it past that point, files vanish or corrupt silently.

Fortunately, Linux users have an excellent toolset to detect and repair such cards: **F3 (Fight Flash Fraud)**. This guide explains how to identify fake SD cards and reformat them to their *true* usable capacity.

---

## ⚙️ What You’ll Need

- A Linux system (any distribution)
- Your SD card and a card reader
- The `f3` package installed

Install F3 with:
```bash
sudo apt install f3
```

---

## 🔍 Step 1: Identify Your SD Card

First, plug in the SD card and list your storage devices:
```bash
lsblk
```

Look for your SD card, typically named `/dev/sdb` or `/dev/sdc`.  
Example:
```
/dev/sdc      59.6G
└─/dev/sdc1   59.6G
```

In this case, the device is `/dev/sdc`. We’ll use that for testing — **never** use the partition (like `/dev/sdc1`) in destructive tests.

---

## 🧪 Step 2: Run a Destructive Test with f3probe

To find out the card’s true size, use:
```bash
sudo f3probe --destructive --time-ops /dev/sdc
```

This will completely erase all data on the SD card and analyze its real memory capacity.  
You’ll see output similar to:

```
F3 probe: Checking device size...
Real capacity: 7.8 GB (7800000000 bytes)
Fake capacity: 64 GB (64000000000 bytes)
```

This means the card reports 64 GB but actually contains only 7.8 GB of usable storage.

---

## 🧰 Step 3: Fix the Partition Table with f3fix

The beauty of F3 is that it can automatically fix the partition table for you.  
At the end of the `f3probe` report, you’ll get a message like:

```
You can fix this device using:
  sudo f3fix --last-sec=15269887 /dev/sdc
```

Simply copy and run that command:
```bash
sudo f3fix --last-sec=15269887 /dev/sdc
```

This rewrites the partition table to match the card’s *real* capacity.

---

## 🧹 Step 4: Reformat the SD Card

Now that the fake capacity is removed, create a new filesystem.

For FAT32 (most cameras and portable devices):
```bash
sudo mkfs.vfat -F 32 -n FIXED_SD /dev/sdc1
```

For Linux-only use (better for servers or embedded boards):
```bash
sudo mkfs.ext4 -L fixed_sd /dev/sdc1
```

---

## ✅ Step 5: Verify the Repair

After reformatting, mount the SD card:
```bash
sudo mkdir /mnt/sdtest
sudo mount /dev/sdc1 /mnt/sdtest
```

Then run a quick verification:
```bash
f3write /mnt/sdtest
f3read /mnt/sdtest
```

You should now see a clean report:
```
Data OK: 100.00%
Average writing speed: 12.34 MB/s
Average reading speed: 18.56 MB/s
```

If that’s the case, congratulations — your SD card has been repaired to a safe, usable state.

---

## ⚠️ Step 6: Understand the Limitations

Even after fixing, a counterfeit SD card remains unreliable for critical storage. Use it only for **temporary files or testing**, not for backups or important projects.  
The problem isn’t just fake capacity — such cards often use lower-grade flash chips with shorter lifespans.

---

## 🧩 Summary

| Step | Command | Purpose |
|------|----------|----------|
| 1 | `lsblk` | Identify SD card device |
| 2 | `sudo f3probe --destructive /dev/sdX` | Detect real capacity |
| 3 | `sudo f3fix --last-sec=XXXX /dev/sdX` | Fix partition table |
| 4 | `sudo mkfs.vfat /dev/sdX1` | Reformat the card |
| 5 | `f3write` + `f3read` | Verify data integrity |

---

## 🧠 Final Thoughts

Counterfeit SD cards exploit a simple trick — false firmware capacity. But Linux tools like F3 let you see the truth and reclaim the usable space. You might end up with a smaller card, but it’ll be *real* and reliable.

If you regularly test or resell SD cards, you can even automate this process in a short Bash script that detects, probes, fixes, and formats in one go.
