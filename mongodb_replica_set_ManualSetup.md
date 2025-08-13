# MongoDB Installation and Replica Set Setup Guide

## Part 1: Install MongoDB on Ubuntu 22.04 / 23.04 / Jammy

### Installation Script

```bash
#!/bin/bash
set -e

echo "[Step 1] Importing GPG key..."
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

echo "[Step 2] Adding MongoDB repo..."
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

echo "[Step 3] Updating package index..."
sudo apt-get update

echo "[Step 4] Installing MongoDB..."
sudo apt-get install -y mongodb-org

echo "[Step 5] Starting and enabling MongoDB service..."
sudo systemctl start mongod
sudo systemctl enable mongod

echo "[Step 6] MongoDB status:"
sudo systemctl status mongod --no-pager

echo "[Step 7] MongoDB version:"
mongod --version
```

### Start and Verify MongoDB

- Start MongoDB:

  ```bash
  sudo systemctl start mongod
  ```

- Enable MongoDB to start on reboot:

  ```bash
  sudo systemctl enable mongod
  ```

- Verify status:

  ```bash
  sudo systemctl status mongod
  ```

- Restart MongoDB:

  ```bash
  sudo systemctl restart mongod
  ```

- Monitor logs:

  ```bash
  tail -f /var/log/mongodb/mongod.log
  ```

### Configure User Authentication (Recommended)

1. Start MongoDB shell:

   ```bash
   mongosh
   ```

2. Switch to admin database:

   ```js
   use admin
   ```

3. Create admin user:

   ```js
   db.createUser({
     user: "admin",
     pwd: "admin@123",
     roles: [{ role: "root", db: "admin" }]
   });
   ```

4. Verify user creation:

   ```js
   db.getUsers()
   ```

5. Enable authorization in `/etc/mongod.conf`:

   ```yaml
   security:
     authorization: "enabled"
   ```

6. Restart MongoDB:

   ```bash
   sudo systemctl restart mongod
   ```

7. Connect with authentication:

   ```bash
   mongosh -u admin -p admin@123 --authenticationDatabase admin
   ```

---

## Part 2: Setting up MongoDB Replica Set

### 1. Configure Hosts File

On **all nodes** (`server1`, `server2`, `server3`), edit `/etc/hosts`:

```
127.0.0.1 localhost
192.168.224.134 server1
192.168.224.135 server2
192.168.224.147 server3
```

### 2. Setup Replica Set Keyfile

On **primary node (server1)**:

```bash
openssl rand -base64 756 > /var/lib/mongodb/keyfile
sudo chown mongodb:mongodb /var/lib/mongodb/keyfile
sudo chmod 400 /var/lib/mongodb/keyfile
```

Copy `/var/lib/mongodb/keyfile` to the same location on **server2** and **server3** with the same ownership and permissions:

```bash
sudo chown mongodb:mongodb /var/lib/mongodb/keyfile
sudo chmod 400 /var/lib/mongodb/keyfile
```

### 3. Configure MongoDB on All Nodes

Edit `/etc/mongod.conf`:

- Enable security and keyFile:

```yaml
security:
  authorization: enabled
  keyFile: /var/lib/mongodb/keyfile
```

- Configure network interfaces (bindIp):

On `server1`:

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1,192.168.224.134
```

On `server2`:

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1,192.168.224.135
```

On `server3`:

```yaml
net:
  port: 27017
  bindIp: 127.0.0.1,192.168.224.147
```

- Configure replication section:

```yaml
replication:
  replSetName: "rs0"
```

Restart MongoDB on **all nodes**:

```bash
sudo systemctl restart mongod
```

### 4. Initialize the Replica Set

On **primary node**, login to MongoDB shell with admin user:

```bash
mongosh -u admin -p admin@123 --authenticationDatabase admin
```

Run the following command to initiate the replica set:

```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "192.168.224.134:27017" },
    { _id: 1, host: "192.168.224.135:27017" },
    { _id: 2, host: "192.168.224.147:27017" }
  ]
})
```

Verify replica set status:

```js
rs.status()
```

### 5. Use the Replica Set

- Create a sample database and insert data:

```js
use company_db
db.employees.insertOne({
  employee_id: 1,
  employee_name: "SUSHMITHA",
  email: "sushma38@gmail.com",
  mobile: "9833998090"
})
```

- On **secondary nodes**, allow reads:

```js
db.getMongo().setReadPref('primaryPreferred')
db.employees.find()
```

- Attempting to write on a secondary node will fail:

```js
db.employees.insertOne({ employee_id: 2, employee_name: "ARCHANA", email: "acchu38@gmail.com", mobile: "9546572810" })
// Will return MongoServerError: not primary
```

### 6. Failover Testing

Stop the primary node:

```bash
sudo systemctl stop mongod
```

Connect to any node and check data consistency:

```bash
mongosh -u admin -p admin@123 --authenticationDatabase admin
use company_db
db.employees.find()
```

### 7. Manual Election and Reconfiguration

If needed, force re-election:

```js
rs.stepDown()
```

Check replica config:

```js
rs.conf()
```

Set priority to influence primary election:

```js
cfg = rs.conf()
cfg.members[0].priority = 2
rs.reconfig(cfg)
```

### 8. Check Replica Set Health

```js
rs.status()
```

---

## Notes

- Always secure your MongoDB instances before exposing to public networks.
- Use DNS names in your replica set configuration for easier management.
- Maintain consistent keyfiles and permissions across all nodes.
- Monitor logs at `/var/log/mongodb/mongod.log` for errors.

---

## References

- [MongoDB Official Docs](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/)
- [Replica Set Tutorial](https://www.mongodb.com/docs/manual/replication/)

---

------------------------------------------------------------------------------------------------------------------------------

cat docker-compose.yaml 
version: '3.8'

services:
  mongodb:
    image: mongo:6.0
    entrypoint: ["/usr/bin/mongod"]
    command: >
      --bind_ip_all
      --port 27117
      --auth
      --wiredTigerCacheSizeGB 2
      --logpath /var/log/mongodb/mongod.log
      --logappend
      --tlsMode requireTLS
      --tlsCertificateKeyFile /certs/tls.pem
      --tlsCAFile /certs/ca.pem
      --tlsAllowConnectionsWithoutCertificates
    volumes:
      - ./mongodb/data:/data/db
      - ./mongodb/logs:/var/log/mongodb
      - ./mongodb/certs:/certs
      - ./mongodb/dumps01:/dumps01
    ports:
      - "27117:27117"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=smtcMONG0
    mem_limit: 8g
    mem_reservation: 4g
    cpus: 4.0
    ulimits:
      nofile:
        soft: 64000
        hard: 64000
      nproc:
        soft: 64000
        hard: 64000
    restart: unless-stopped
    healthcheck:
      disable: true

