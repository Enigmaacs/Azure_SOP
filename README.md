
# Standard Operating Procedure: Deploying a Linux VM with Azure Bastion and Sample App

This document provides step-by-step instructions to:
- Create a Virtual Network with subnet, NAT Gateway, and Route Table
- Connect to the VM using Azure Bastion
- Deploy a sample Node.js application

## Pre requisites

- Azure Subscription
- Azure CLI installed and logged in (`az login`)
- SSH key pair (generated or use `az vm create` to generate)

## 1. Create Resource Group

Create a new resource group in your Azure subscription.

```bash
az group create \
  --name myResourceGroup \
  --location eastus
````

---

## 2. Create Virtual Network and Subnet

Create a Virtual Network (`myVNet`) and a subnet (`mySubnet`) within it.

```bash
az network vnet create \
  --resource-group myResourceGroup \
  --location eastus \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.1.0/24
```

---

## 3. Create Public IP and NAT Gateway

Create a Public IP and NAT Gateway for outbound traffic.

```bash
az network public-ip create \
  --resource-group myResourceGroup \
  --name myNatPublicIP \
  --sku Standard \
  --allocation-method Static

az network nat gateway create \
  --resource-group myResourceGroup \
  --name myNatGateway \
  --public-ip-addresses myNatPublicIP \
  --idle-timeout 10 \
  --sku Standard
```

---

## 4. Associate NAT Gateway with Subnet

Associate the NAT Gateway with the subnet to manage outbound traffic.

```bash
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --nat-gateway myNatGateway
```

---

## 5. Create Route Table and Add Route

Create a route table and add a default route to the internet.

```bash
az network route-table create \
  --resource-group myResourceGroup \
  --name myRouteTable \
  --location eastus

az network route-table route create \
  --resource-group myResourceGroup \
  --route-table-name myRouteTable \
  --name DefaultRoute \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type Internet

az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --route-table myRouteTable
```

---

## 6. Create Azure Bastion

### Create Bastion Subnet

Create a subnet specifically for Azure Bastion.

```bash
az network vnet subnet create \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name AzureBastionSubnet \
  --address-prefix 10.0.2.0/24
```

### Create Public IP and Bastion Host

Create a Public IP and the Bastion host to connect securely to your VM.

```bash
az network public-ip create \
  --resource-group myResourceGroup \
  --name BastionPublicIP \
  --sku Standard \
  --allocation-method Static

az network bastion create \
  --resource-group myResourceGroup \
  --name MyBastionHost \
  --public-ip-address BastionPublicIP \
  --vnet-name myVNet \
  --location eastus
```

---

## 7. Create Linux VM (Without Public IP)

Create a Linux VM without a public IP for enhanced security.

```bash
az vm create \
  --resource-group myResourceGroup \
  --name myAppVM \
  --image UbuntuLTS \
  --vnet-name myVNet \
  --subnet mySubnet \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address "" \
  --nsg-rule SSH
```

---

## 8. Connect to VM Using Bastion

1. Go to the Azure Portal, navigate to `myAppVM`, and click on **Connect** > **Bastion**.
2. Enter username `azureuser` and use the SSH private key when prompted.

---

## 9. Install Sample Node.js Application

Once connected to your VM, install the required packages and deploy a simple Node.js application.

```bash
sudo apt-get update
sudo apt-get install -y nodejs npm git
git clone https://github.com/heroku/node-js-sample.git
cd node-js-sample
npm install
PORT=80 sudo node index.js
```

---

## 10. Access the Application

To access your Node.js application:

1. Configure your Network Security Group (NSG) to allow inbound traffic on port 80.
2. (Optional) You can expose the app publicly using an Azure Load Balancer or Application Gateway.

---

## 11. Cleanup (Optional)

To delete the resource group and clean up all resources:

```bash
az group delete --name myResourceGroup --yes --no-wait
```

---

```

This `README.md` document contains all the steps for setting up the environment, connecting to the VM using Bastion, installing and running the sample application, and cleaning up resources afterward. Let me know if you need any further adjustments!
```

