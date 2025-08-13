
#  Create a Linux Virtual Machine in Azure

This guide provides instructions to create a Linux virtual machine (VM) running **Ubuntu Server 22.04 LTS** in Azure using either the **Azure portal** or the **Azure CLI**. It covers connecting to the VM via SSH, installing an NGINX web server, and managing resources.

**Applies to**: Linux VMs  
**Last updated**: May 2, 2025

---

## Prerequisites

- An Azure subscription. [Create a free account](https://azure.microsoft.com/free/).
- For Azure CLI:
  - [Azure CLI installed](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).
  - SSH client installed (e.g., OpenSSH on Linux/Mac or PowerShell on Windows).

---

## Option 1: Create a VM Using the Azure Portal

### Step 1: Sign in to Azure
- Open the [Azure portal](https://portal.azure.com) and sign in.

### Step 2: Create a Virtual Machine

1. Search for **Virtual machines** in the search bar.
2. Click **Create > Virtual machine**.
3. On the **Create a virtual machine** page:

#### Project Details
- **Subscription**: Use your subscription.
- **Resource group**: Click *Create new* and enter `myResourceGroup`.

#### Instance Details
- **Virtual machine name**: `myVM`
- **Image**: Ubuntu Server 22.04 LTS - Gen2
- **Size**: Use default or choose available size for your region.

#### Administrator Account
- **Authentication type**: SSH public key
- **Username**: `azureuser`
- **SSH public key source**: Generate new key pair
- **Key pair name**: `myKey`

#### Inbound Port Rules
- **Public inbound ports**: Allow selected ports
- **Ports**: SSH (22) and HTTP (80)

4. Click **Review + create**, then **Create**.

5. Download the private key (`myKey.pem`) and save it securely.

6. Once deployed, click **Go to resource** and copy the **public IP address**.

### Step 3: Connect to the Virtual Machine

```bash
chmod 400 ~/Downloads/myKey.pem
ssh -i ~/Downloads/myKey.pem azureuser@<Public-IP>
```

### Step 4: Install NGINX Web Server

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
exit
```

### Step 5: View the Web Server

- Open your browser and go to `http://<Public-IP>`.
- You should see the default NGINX welcome page.

---

## Option 2: Create a VM Using the Azure CLI

### Step 1: Sign in to Azure

```bash
az login
```

### Step 2: Create a Virtual Machine

Set environment variables:

```bash
RESOURCE_GROUP="myResourceGroup"
LOCATION="<location>" # e.g., eastus
VM_NAME="myVM"
USERNAME="azureuser"
IMAGE="Canonical:0001-com-ubuntu-server-jammy:22_04-lts-gen2:latest"
KEY_NAME="myKey"
```

Create resource group:

```bash
az group create --name $RESOURCE_GROUP --location $LOCATION
```

Generate SSH key:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/$KEY_NAME -N ""
```

Create VM:

```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --location $LOCATION \
  --image $IMAGE \
  --admin-username $USERNAME \
  --ssh-key-values ~/.ssh/$KEY_NAME.pub \
  --size Standard_D2s_v3 \
  --public-ip-sku Standard \
  --nsg-rule SSH,HTTP
```

Get public IP:

```bash
IP_ADDRESS=$(az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --show-details \
  --query publicIps \
  --output tsv)
echo $IP_ADDRESS
```

### Step 3: Connect to the Virtual Machine

```bash
chmod 400 ~/.ssh/$KEY_NAME
ssh -i ~/.ssh/$KEY_NAME $USERNAME@$IP_ADDRESS
```

### Step 4: Install NGINX Web Server

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
exit
```

### Step 5: View the Web Server

- Visit `http://$IP_ADDRESS` in your browser to see the NGINX welcome page.

---

## Step 6: Clean Up Resources (Optional)

### Using Azure Portal
1. On the VM's page, click **Resource group**.
2. Click **Delete resource group**, confirm the name, and delete.

### Using Azure CLI

```bash
az group delete --name $RESOURCE_GROUP --yes --no-wait
```

---

## Auto-Shutdown (Alternative)

### Using Azure Portal
1. Go to VM > **Operations** > **Auto-shutdown**
2. Enable, set time, and click **Save**

### Using Azure CLI

Enable auto-shutdown:

```bash
az vm auto-shutdown \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --time 2200
```

Disable auto-shutdown:

```bash
az vm auto-shutdown \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --off
```

---

## Additional Notes

- VM size and pricing vary by region. Use:
  ```bash
  az vm list-sizes --location $LOCATION
  ```
- SSH keys are stored locally (e.g., `~/.ssh/myKey`). Keep them secure.
- For advanced configurations, refer to:
  - [Azure Virtual Machines documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/)
  - [Azure CLI VM documentation](https://learn.microsoft.com/en-us/cli/azure/vm)

---
