
# Azure VNet Setup: Public & Private Subnets, VM Access via Jump Host and Bastion

## 🧾 Standard Operating Procedure (SOP)

### Title: Azure Network and VM Deployment with Access Methods  

---

## 📌 Objective
To set up a secure Azure Virtual Network (VNet) with isolated public and private subnets, configure route tables, deploy a public VM, and access a private VM using either a jump host or Azure Bastion.

---

## 🔧 Part 1: VNet and Public VM Setup

### 1. Create a Resource Group
1. Go to **Azure Portal**
2. Navigate to **Resource Groups** → **Create**
3. Enter:
   - **Subscription**: Choose your subscription
   - **Resource Group Name**: `demo-rg`
   - **Region**: Select your desired region
   - **Tags**: Optional
4. Click **Review + Create** → **Create**

### 2. Create a Virtual Network (VNet)
1. Go to **Virtual Networks** → **Create**
2. Enter:
   - **Subscription**: Same as above
   - **Resource Group**: `demo-rg`
   - **VNet Name**: `demo-vnet`
   - **Region**: Same as resource group
3. (Optional) Enable **Virtual Network Encryption**
4. Under **IP Addresses**:
   - **Address space**: `10.0.0.0/16`
   - Delete the default subnet
5. Click **Review + Create** → **Create**

### 3. Add Subnets to VNet
1. Go to **Virtual Networks** → `demo-vnet` → **Subnets**
2. Add Subnet:
   - **Name**: `public-subnet`
   - **Address range**: `10.0.1.0/24`
3. Add another Subnet:
   - **Name**: `private-subnet`
   - **Address range**: `10.0.2.0/24`

### 4. Create Route Tables

#### Public Route Table
- Name: `demo-pub-rt`
- Associate with `public-subnet`

#### Private Route Table
- Name: `demo-pvt-rt`
- Associate with `private-subnet`

### 5. Add Internet Route to Public Route Table
1. Go to **Route Tables** → `demo-pub-rt` → **Routes** → **+ Add**
2. Configure:
   - **Route Name**: `internet`
   - **Address Prefix**: `0.0.0.0/0`
   - **Next Hop Type**: `Internet`
3. Click **Add**

### 6. Deploy a Public VM
1. Go to **Virtual Machines** → **Create**
2. Configure:
   - Name: `ubuntu-pub-vm`
   - Region: Same as VNet
   - Authentication: SSH Public Key
   - Username: `azureuser`
   - SSH Key: Generate new or use existing
   - Inbound Ports: SSH (22)
   - VNet: `demo-vnet`, Subnet: `public-subnet`
   - Public IP: Enabled
3. Click **Create**

### 7. SSH into Public VM
```bash
cd ~/Downloads
mv ubuntu-pub-vm_key.pem ubuntu-pub-vm_key
chmod 400 ubuntu-pub-vm_key
ssh -i ubuntu-pub-vm_key azureuser@<PUBLIC-IP>
```

---

## 🔧 Part 2: Access Private VM

### 1. Deploy a Private VM (No Public IP)
1. Go to **Virtual Machines** → **Create**
2. Configure:
   - Name: `demo-private-vm`
   - SSH Key: `demo-pvt-vm-ssh-key-pair`
   - VNet: `demo-vnet`
   - Subnet: `private-subnet`
   - Public IP: **None**
3. Click **Create**

### 2. Access Private VM via Public VM (Jump Host)
1. SSH into the public VM
2. Create private key file:
```bash
touch demo-pvt-vm-ssh-key-pair.pem
vi demo-pvt-vm-ssh-key-pair.pem  # Paste key content here
chmod 400 demo-pvt-vm-ssh-key-pair.pem
```
3. SSH into private VM:
```bash
ssh -i demo-pvt-vm-ssh-key-pair.pem azureuser@<PRIVATE-IP>
```

### 3. Access Private VM via Azure Bastion
1. Go to **demo-private-vm** → **Connect** → **Bastion**
2. Choose:
   - **Authentication**: SSH Private Key
   - **Username**: `azureuser`
   - **Browse Private Key File**
3. Click **Connect**

---

## 💡 Notes
- Use Azure Bastion for secure, scalable access without managing jump hosts.
- Always secure your `.pem` keys with proper file permissions (`chmod 400`).
