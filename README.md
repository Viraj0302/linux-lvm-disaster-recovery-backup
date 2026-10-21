
# Linux LVM-Based Disaster Recovery and Backup System

## Project Overview

This project demonstrates a Linux-based disaster recovery and backup solution implemented on Red Hat Enterprise Linux (RHEL) 9.8 using VMware.

The project uses Logical Volume Manager (LVM) to manage storage, LVM snapshots to provide a point-in-time recovery source, and compressed TAR archives to back up and restore company data.

A simulated data-loss scenario was performed by deleting a critical server configuration file and recovering it from an LVM snapshot. A separate TAR backup was also extracted and verified to confirm that the archived data could be restored.

## Project Objectives

- Configure Linux storage using LVM.
- Create Physical Volumes, Volume Groups, and Logical Volumes.
- Allocate separate Logical Volumes for company data and backup storage.
- Create an LVM snapshot for recovery testing.
- Simulate accidental deletion of a critical configuration file.
- Recover the deleted file using the snapshot.
- Create a compressed TAR backup.
- Extract and verify the backup contents.
- Troubleshoot insufficient free space during snapshot creation.

## Environment and Technologies

| Technology | Purpose |
|---|---|
| Red Hat Enterprise Linux 9.8 | Operating system |
| VMware | Virtual machine environment |
| LVM | Flexible storage management |
| XFS | Filesystem |
| fdisk | Disk partitioning |
| pvcreate | Physical Volume creation |
| vgcreate | Volume Group creation |
| vgextend | Volume Group expansion |
| lvcreate | Logical Volume and snapshot creation |
| mount | Filesystem mounting |
| TAR | Backup and restoration |
| Linux CLI | Administration and troubleshooting |

## Storage Architecture

The project uses the following storage components:

- `/dev/sdb`: Initial 10 GB virtual disk.
- `/dev/sdc`: Additional 5 GB virtual disk added to provide more storage capacity.
- `company_vg`: LVM Volume Group.
- `data_lv`: 6 GB Logical Volume for company data.
- `backup_lv`: Approximately 4 GB Logical Volume for backup storage.
- `data_snapshot`: 1 GB LVM snapshot.
- `/company/data`: Source data mount point.
- `/company/backup`: Backup storage mount point.

The operating system remains on the original system disk, separate from the project storage.

## Project Directory Structure

The project contains sample business data representing three departments:

```text
/company/
├── data/
│   ├── IT/
│   │   └── server_config.txt
│   ├── operations/
│   │   └── operations_data.txt
│   └── customers/
│       └── customer_data.txt
└── backup/
    └── company_data_backup.tar.gz
```

## Implementation Steps

### 1. Identify Available Disks

```bash
lsblk
fdisk -l
```

These commands were used to identify available disks and verify the project disk partitions.

### 2. Configure LVM Storage

The project involved creating Physical Volumes, a Volume Group, and separate Logical Volumes.

```bash
pvcreate /dev/sdb1
vgcreate company_vg /dev/sdb1
lvcreate -L 6G -n data_lv company_vg
lvcreate -L 4G -n backup_lv company_vg
```

These are representative commands from the initial storage setup. Disk and Volume Group capacity must be checked before executing them.

### 3. Create and Mount Filesystems

The Logical Volumes were formatted and mounted for data and backup storage.

```bash
mkfs.xfs /dev/company_vg/data_lv
mkfs.xfs /dev/company_vg/backup_lv

mkdir -p /company/data
mkdir -p /company/backup

mount /dev/company_vg/data_lv /company/data
mount /dev/company_vg/backup_lv /company/backup
```

### 4. Create Company Data

Sample department directories and files were created under `/company/data` to represent business information and a critical server configuration file.

### 5. Troubleshoot Insufficient Storage Space

The initial snapshot creation attempt failed because the Volume Group did not have enough free space. An additional 5 GB virtual disk was added and incorporated into the existing Volume Group.

Representative commands:

```bash
pvcreate /dev/sdc1
vgextend company_vg /dev/sdc1
vgs
```

The `vgs` command was used to verify the available Volume Group space.

### 6. Create and Mount an LVM Snapshot

A snapshot was created for `data_lv` after additional storage was made available.

```bash
lvcreate -L 1G -s -n data_snapshot /dev/company_vg/data_lv
lvs
```

For the XFS recovery test, the snapshot was mounted using `nouuid` because it has the same filesystem UUID as the original filesystem.

```bash
mkdir -p /mnt/recovery

mount -o nouuid \
  /dev/company_vg/data_snapshot \
  /mnt/recovery

findmnt /mnt/recovery
find /mnt/recovery -type f
```

### 7. Perform Snapshot-Based Recovery

A critical server configuration file was deleted as part of the simulated disaster scenario. The snapshot was mounted and used as the source for recovering the deleted file.

The recovered file was checked to confirm that the required data was available.

### 8. Create a Compressed TAR Backup

```bash
tar -czvf \
  /company/backup/company_data_backup.tar.gz \
  /company/data
```

Options:

- `-c`: Create an archive.
- `-z`: Compress using gzip.
- `-v`: Display processed files.
- `-f`: Specify the archive filename.

### 9. Verify and Restore the TAR Backup

List the archive contents without extracting:

```bash
tar -tzvf /company/backup/company_data_backup.tar.gz
```

Extract the archive into a temporary test directory:

```bash
mkdir -p /tmp/backup_test

tar -xzvf \
  /company/backup/company_data_backup.tar.gz \
  -C /tmp/backup_test
```

Verify the extracted files:

```bash
find /tmp/backup_test -type f
```

The extracted data was checked to verify that the backup contained the expected files.

## Validation and Testing

| Test | Result documented in project |
|---|---|
| Detect project disks | Passed |
| Create LVM Physical Volume | Passed |
| Create Volume Group | Passed |
| Create data and backup Logical Volumes | Passed |
| Troubleshoot insufficient snapshot space | Resolved by adding storage |
| Create LVM snapshot | Passed |
| Recover deleted configuration file | Passed |
| Create compressed TAR backup | Passed |
| Extract and verify backup contents | Passed |

## Key Learning Outcomes

- Linux disk and partition management.
- LVM Physical Volume, Volume Group, and Logical Volume administration.
- XFS filesystem creation and mounting.
- LVM snapshot creation and recovery.
- TAR backup creation and restoration.
- Troubleshooting storage capacity issues.
- Linux command-line administration and verification.

## Limitations

This is a lab-based implementation. The TAR archive is stored in the same Linux environment, and automated backup scheduling, remote replication, encryption, backup retention, and centralized monitoring were not implemented.

An LVM snapshot is a recovery mechanism, not a replacement for an independent backup.

## Future Enhancements

- Automate backups with Cron or a systemd timer.
- Store backups on remote storage.
- Add backup retention and cleanup policies.
- Encrypt sensitive backup archives.
- Implement checksum-based integrity verification.
- Add monitoring and recovery alerts.

## Author

**Viraj Vishwas Pawar**

**Project:** Linux LVM-Based Disaster Recovery and Backup System  
**Environment:** RHEL 9.8 | VMware

