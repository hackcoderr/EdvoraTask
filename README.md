# EdvoraTask
1. Create a Kubernetes cluster with 2 node pools; 2 nodes each.
2. Install cert-manager, ingress-nginx and configure them.

**Note: I am setting up this lab on Azure cloud.**
## 1. Create a Kubernetes cluster with 2 node pools; 2 nodes each.

**Note: I am unable to add one more node pool in this cluster due to resourse limit on my azure account so I have generated my request for increase resourse limit on my account but I added the command ``az aks nodepool add --resource-group GitHubActionsRunners --cluster-name GitHubActionsRunnersK8sCluster --name mynodepool --node-count 2`` in my installation script to add node pool. so as the resourse limit of my account will be increse, I will run this command to add one more node pool.**

**Rightnow My cluster is having 1 node pool with 2 nodes.**

:warning: *All the below assumes you are running `Bash`.*

### Install az cli

```bash
# Refresh packages
apt-get update
apt-get upgrade

# From:
# https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt
sudo apt-get update
sudo apt-get install ca-certificates curl apt-transport-https lsb-release gnupg

# Download the microsoft signing keys
curl -sL https://packages.microsoft.com/keys/microsoft.asc |
    gpg --dearmor |
    sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null

# Add the Azure CLI software repository:
AZ_REPO=$(lsb_release -cs)
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" |
    sudo tee /etc/apt/sources.list.d/azure-cli.list

# Update repository information and install the azure-cli package:
sudo apt-get update
sudo apt-get install azure-cli
```

### Install kubectl (latest stable version)

```bash
# Download the latest release 
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Download the kubectl checksum file:
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

# Validate the kubectl binary against the checksum file:
echo "$(<kubectl.sha256)  kubectl" | sha256sum --check

# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify
kubectl version --client

# Install auto-completion
# OPTIONAL
# -
sudo apt-get install bash-completion
source /usr/share/bash-completion/bash_completion
```

### Install Helm

```bash

# Add signing keys
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -

# Install dependencies
sudo apt-get install apt-transport-https --yes

# Add repository
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update

# Install helm
sudo apt-get install helm

# Verify
helm version

```

### Setup AKS via Azure CLI

```bash
# Authenticate with Azure CLI
az login

# list regions with az
az account list-locations

# Create a resource group for our AKS cluster
az group create --name GitHubActionsRunners --location westeurope

# Get list of resources in the resource group
az group show --resource-group GitHubActionsRunners

# Verify Microsoft.OperationsManagement and Microsoft.OperationalInsights 
# are registered on your subscription.
az provider show -n Microsoft.OperationsManagement -o table
az provider show -n Microsoft.OperationalInsights -o table

# Create AKS cluster in resource group
# --name cannot exceed 63 characters and can only contain letters, 
# numbers, or dashes (-).
az aks create \
  --resource-group GitHubActionsRunners \
  --name GitHubActionsRunnersK8sCluster \
  --enable-addons monitoring \
  --node-count 1 \
  --generate-ssh-keys

###############################################################################
# Access K8s cluster
###############################################################################

# Configure kubectl to connect to your Kubernetes cluster
# Downloads credentials and configures the Kubernetes CLI to use them.
  # Uses ~/.kube/config, the default location for the Kubernetes configuration 
  # file. Specify a different location for your Kubernetes configuration file 
  # using --file.
az aks get-credentials \
  --resource-group GitHubActionsRunners \
  --name GitHubActionsRunnersK8sCluster

# Verify
kubectl config get-contexts
# AND
kubectl get nodes

###############################################################################
# Manually scaling nodes
###############################################################################
# add node pool
az aks nodepool add \
    --resource-group GitHubActionsRunners \
    --cluster-name GitHubActionsRunnersK8sCluster \
    --name mynodepool \
    --node-count 2

# varify 
az aks nodepool list --resource-group GitHubActionsRunners --cluster-name GitHubActionsRunnersK8sCluster

# Scale down
# (OPTIONAL)
az aks scale \
  --resource-group GitHubActionsRunners \
  --name GitHubActionsRunnersK8sCluster \
  --node-count 1

# Check progress
watch -n 2 kubectl get nodes

```
