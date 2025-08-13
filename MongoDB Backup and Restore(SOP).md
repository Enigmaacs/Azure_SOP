# MongoDB Backup and Restore Standard Operating Procedure (SOP)

This document outlines the steps for backing up and restoring MongoDB databases using supported methods: logical (`mongodump`/`mongorestore`) and filesystem-level. This SOP is designed for a MongoDB replica set (`rs0`) and can be applied to all databases.

## Scope
This procedure applies to the MongoDB replica set `rs0` with the following nodes:
- `replica1`: 172.16.0.101:27017 (Secondary)
- `replica2`: 172.16.0.102:27017 (Primary)
- `replica3`: 172.16.0.103:27017 (Secondary)

The backup will be performed on `backup-server` by the user `backup-user` and stored at `/home/backup-user/backups/`.

## Prerequisites
- MongoDB Database Tools installed on `backup-server`.
- Network access from `backup-server` to all replica set nodes.
- User `newUser` with appropriate roles (e.g., `readWrite` on target databases, authentication database: `admin`).
- Sufficient disk space at `/home/backup-user/backups/`.

## Procedure

### Step 1: Verify MongoDB Database Tools Installation
1. Log in to `backup-server` as user `backup-user`.
2. Open a terminal.
3. Check if `mongodump` is installed:
   ```bash
   dpkg -l | grep mongodb-database-tools
   ```
4. If installed, you should see version `100.12.2`. If not, install the tools:
   - Download the `.deb` package:
     ```bash
     wget https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu2404-x86_64-100.12.2.deb -P /tmp
     ```
   - Install the package:
     ```bash
     sudo apt install /tmp/mongodb-database-tools-ubuntu2404-x86_64-100.12.2.deb
     ```
   - Verify installation:
     ```bash
     mongodump --version
     ```

### Step 2: Prepare the Backup Directory
1. Create a backup directory with a timestamp (e.g., `YYYYMMDD` format):
   ```bash
   mkdir -p /home/backup-user/backups/backup_$(date +%Y%m%d)
   ```
2. Verify permissions:
   ```bash
   ls -ld /home/backup-user/backups
   ```
3. Ensure `backup-user` has write access. If not, fix permissions:
   ```bash
   chown -R backup-user:backup-user /home/backup-user/backups
   chmod -R 755 /home/backup-user/backups
   ```

### Step 3: Logical Backup Using `mongodump`

#### Backup All Databases
To back up all databases in the replica set:
```bash
mongodump --uri "mongodb://newUser:securePassword123@172.16.0.102:27017,172.16.0.101:27017,172.16.0.103:27017/?replicaSet=rs0&authSource=admin" --out /home/backup-user/backups/backup_$(date +%Y%m%d)
```
- Replace `securePassword123` with the actual password for `newUser`.

#### Backup a Single Database
To back up a specific database (replace `<database>` with the database name):
```bash
mongodump --uri "mongodb://newUser:securePassword123@172.16.0.102:27017/<database>?replicaSet=rs0&authSource=admin" --db <database> --out /home/backup-user/backups/backup_$(date +%Y%m%d)
```

#### Backup a Specific Collection
To back up a specific collection (replace `<database>` and `<collection>`):
```bash
mongodump --uri "mongodb://newUser:securePassword123@172.16.0.102:27017/<database>?replicaSet=rs0&authSource=admin" --collection <collection> --db <database> --out /home/backup-user/backups/backup_$(date +%Y%m%d)
```

### Step 4: Restore Using `mongorestore`

#### Restore a Complete Backup
To restore all databases from a backup:
```bash
mongorestore --uri "mongodb://newUser:securePassword123@172.16.0.102:27017,172.16.0.101:27017,172.16.0.103:27017/?replicaSet=rs0&authSource=admin" /home/backup-user/backups/backup_20250610
```

#### Restore a Specific Database
To restore a specific database (replace `<database>`):
```bash
mongorestore --uri "mongodb://newUser:securePassword123@172.16.0.102:27017/?replicaSet=rs0&authSource=admin" --nsInclude=<database>.* /home/backup-user/backups/backup_20250610/<database>
```

#### Restore a Specific Collection
To restore a specific collection (replace `<database>` and `<collection>`):
```bash
mongorestore --uri "mongodb://newUser:securePassword123@172.16.0.102:27017/?replicaSet=rs0&authSource=admin" --nsInclude=<database>.<collection> /home/backup-user/backups/backup_20250610/<database>/<collection>.bson
```

### Step 5: Filesystem Backup (Cold Backup)
For a filesystem-level backup, stop the MongoDB service to ensure consistency. This method is suitable for standalone instances or secondary nodes in a replica set (to minimize downtime on the primary).

#### Steps:
1. Stop the MongoDB service on a secondary node (e.g., `replica1` at `172.16.0.101`):
   ```bash
   ssh backup-user@172.16.0.101
   sudo systemctl stop mongod
   ```
2. Copy the data directory:
   ```bash
   sudo cp -r /var/lib/mongodb /home/backup-user/backups/mongodb_backup_$(date +%F)
   ```
3. Restart the MongoDB service:
   ```bash
   sudo systemctl start mongod
   ```
4. Exit the SSH session:
   ```bash
   exit
   ```

**Note**: If backing up from a secondary, ensure the node is in `SECONDARY` state (`rs.status()`) and there’s minimal replication lag.

### Step 6: Verification Steps After Backup and Restore
1. **Verify Backup**:
   - Check the backup directory for `.bson` files:
     ```bash
     ls /home/backup-user/backups/backup_$(date +%Y%m%d)
     ```
2. **Verify Restore**:
   - Connect to the MongoDB shell:
     ```bash
     mongo --host 172.16.0.102:27017 -u newUser -p securePassword123 --authenticationDatabase admin
     ```
   - Inspect databases and collections:
     ```javascript
     show dbs
     use <database>
     db.getCollectionNames()
     db.<collection>.find().pretty()
     ```
   - Check replica set health:
     ```javascript
     rs.status()
     ```
     Ensure all nodes are in `PRIMARY` or `SECONDARY` state with minimal `optimeDate` differences.

### Step 7: Schedule Regular Backups (Optional)
1. Open the crontab file:
   ```bash
   crontab -e
   ```
2. Add a daily backup job at 2 AM for all databases:
   ```bash
   0 2 * * * mongodump --uri "mongodb://newUser:securePassword123@172.16.0.102:27017,172.16.0.101:27017,172.16.0.103:27017/?replicaSet=rs0&authSource=admin" --out /home/backup-user/backups/backup_$(date +\%Y\%m\%d) --gzip
   ```
3. Save and exit.

## Notes
- Ensure backups and restores are **tested regularly** in a non-production environment.
- Store backups in a **separate disk/location/cloud** to prevent data loss from hardware failure.
- Always monitor disk space before initiating backups.
- Secure backup files using appropriate permissions and encryption if needed:
  ```bash
  chmod -R 600 /home/backup-user/backups
  ```
- For production systems, consider enabling TLS/SSL for `mongodump` and `mongorestore`:
  ```bash
  mongodump --uri "mongodb://newUser:securePassword123@172.16.0.102:27017,172.16.0.101:27017,172.16.0.103:27017/?replicaSet=rs0&authSource=admin&ssl=true" --out /home/backup-user/backups/backup_$(date +%Y%m%d) --gzip
  ```

## Troubleshooting
- **Authentication Error**:
  - Verify `newUser` credentials:
    ```bash
    mongo --host 172.16.0.102:27017 -u newUser -p securePassword123 --authenticationDatabase admin
    use admin
    db.getUser("newUser")
    ```
  - Ensure `newUser` has appropriate roles for the target databases.
- **Connection Failure**:
  - Test network connectivity:
    ```bash
    ping 172.16.0.102
    ```
  - Check MongoDB port:
    ```bash
    nc -zv 172.16.0.102 27017
    ```
  - Verify replica set status:
    ```javascript
    rs.status()
    ```
- **Permission Denied**:
  - Ensure `backup-user` has write access to `/home/backup-user/backups`.
