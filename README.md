# Azure Medallion Data Pipeline Project

This markdown file combines all the instructions, scripts, and code necessary to set up an Azure Medallion Data Pipeline for processing daily employee roster CSV files through Bronze, Silver, and Gold layers using Azure services.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Overview](#project-overview)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
  - [1. Clone the Repository](#1-clone-the-repository)
  - [2. Install Required Tools](#2-install-required-tools)
  - [3. Configure Azure Authentication](#3-configure-azure-authentication)
  - [4. Set Up Azure Infrastructure](#4-set-up-azure-infrastructure)
    - [Option 1: Using the Jupyter Notebook](#option-1-using-the-jupyter-notebook)
    - [Option 2: Using the Bash Script](#option-2-using-the-bash-script)
  - [5. Configure Azure Data Factory](#5-configure-azure-data-factory)
  - [6. Develop Databricks Notebooks](#6-develop-databricks-notebooks)
  - [7. Create Data Factory Pipelines](#7-create-data-factory-pipelines)
  - [8. Schedule Pipeline Execution](#8-schedule-pipeline-execution)
- [Additional Considerations](#additional-considerations)
- [Cleaning Up Resources](#cleaning-up-resources)
- [Complete Project Files](#complete-project-files)
  - [README.md](#readmemd)
  - [Azure_Medallion_Pipeline_Setup.ipynb](#azure_medallion_pipeline_setupipynb)
  - [setup_medallion_pipeline.sh](#setup_medallion_pipelinesh)
  - [push_notebook_to_github.sh](#push_notebook_to_githubsh)
- [Conclusion](#conclusion)

---

## Prerequisites

- **Azure Subscription**: An active Azure subscription with sufficient permissions.
- **Azure CLI**: Installed and configured on your local machine.
  - [Install Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- **Git**: Installed on your local machine.
  - [Download Git](https://git-scm.com/downloads)
- **Python 3.x**: Installed on your local machine.
  - [Download Python](https://www.python.org/downloads/)
- **Visual Studio Code**: Installed with the following extensions:
  - **Python Extension**
  - **Jupyter Extension**
- **GitHub CLI (`gh`)**: Installed for GitHub repository management.
  - [Install GitHub CLI](https://cli.github.com/manual/installation)

---

## Project Overview

The pipeline uses the following Azure services:

- **Azure Storage Account**: Stores data in Bronze, Silver, and Gold layers.
- **Azure Data Factory (ADF)**: Orchestrates data movement and transformations.
- **Azure Databricks**: Performs data processing and transformations.
- **Azure Key Vault** (Optional): Stores secrets and credentials securely.

---

## Project Structure

```
AzureMedallionPipeline/
├── README.md
├── Azure_Medallion_Pipeline_Setup.ipynb
├── setup_medallion_pipeline.sh
└── push_notebook_to_github.sh
```

- **`README.md`**: Contains detailed instructions and documentation.
- **`Azure_Medallion_Pipeline_Setup.ipynb`**: Jupyter notebook to set up Azure infrastructure.
- **`setup_medallion_pipeline.sh`**: Bash script to create Azure resources.
- **`push_notebook_to_github.sh`**: Script to push the notebook to your GitHub repository.

---

## Setup Instructions

### 1. Clone the Repository

Clone the repository to your local machine:

```bash
git clone https://github.com/jaybenne2001/copilot-use-case.git
```

Navigate to the project directory:

```bash
cd copilot-use-case
```

### 2. Install Required Tools

Ensure that you have the following tools installed:

- **Azure CLI**
- **Git**
- **Python 3.x**
- **GitHub CLI (`gh`)**

### 3. Configure Azure Authentication

Login to your Azure account using Azure CLI:

```bash
az login
```

Verify that you are logged in:

```bash
az account show
```

### 4. Set Up Azure Infrastructure

#### Option 1: Using the Jupyter Notebook

Open the `Azure_Medallion_Pipeline_Setup.ipynb` notebook in Visual Studio Code:

1. Open VS Code.
2. Open the project folder (`copilot-use-case`).
3. Locate and open `Azure_Medallion_Pipeline_Setup.ipynb`.
4. Run each cell sequentially:
   - Provide the required inputs when prompted (resource names, locations, etc.).
   - The notebook will execute Azure CLI commands to create resources.

#### Option 2: Using the Bash Script

1. Open a terminal in the project directory.
2. Make the script executable:

   ```bash
   chmod +x setup_medallion_pipeline.sh
   ```

3. Run the script:

   ```bash
   ./setup_medallion_pipeline.sh
   ```

4. Follow the prompts to provide the required information.

### 5. Configure Azure Data Factory

#### Set Up Linked Services

- **Azure Storage Linked Service**:
  - Use your Storage Account name and key.
  - Optionally, store the Storage Account key in Azure Key Vault.

- **Azure Databricks Linked Service**:
  - Obtain the Databricks workspace URL (FQDN).
    - Format: `https://<databricks-instance>.azuredatabricks.net`
  - Generate a Personal Access Token (PAT) in Databricks.
  - Configure the linked service using the workspace URL and PAT.

#### Create Datasets

- **Bronze Layer Dataset**:
  - Point to the `bronze` container in your Storage Account.
  - Define the dataset schema if necessary.

- **Silver and Gold Layer Datasets**:
  - Similarly, define datasets for `silver` and `gold` containers.

### 6. Develop Databricks Notebooks

#### Bronze to Silver Transformation

Create a notebook named `BronzeToSilver` with the following sample code:

```python
# Replace placeholders with your actual storage account name and key
storage_account_name = '<your_storage_account_name>'
storage_account_key = '<your_storage_account_key>'
container_bronze = 'bronze'
container_silver = 'silver'

# Set up the Spark configuration to access Azure Storage
spark.conf.set(
    f"fs.azure.account.key.{storage_account_name}.dfs.core.windows.net",
    storage_account_key
)

# Read data from the Bronze layer
bronze_df = spark.read.csv(
    f"abfss://{container_bronze}@{storage_account_name}.dfs.core.windows.net/employee_roster/*.csv",
    header=True
)

# Data cleaning and transformation
silver_df = bronze_df.dropDuplicates().filter("employee_id IS NOT NULL")

# Write data to the Silver layer
silver_df.write.mode("overwrite").parquet(
    f"abfss://{container_silver}@{storage_account_name}.dfs.core.windows.net/employee_roster/"
)
```

#### Silver to Gold Transformation

Create a notebook named `SilverToGold` with your specific business logic for transforming data from Silver to Gold layers.

### 7. Create Data Factory Pipelines

#### Define Activities

- **Copy Activity**:
  - To ingest data into the Bronze layer.
  - Configure the source (e.g., your daily CSV files) and sink (Bronze dataset).

- **Databricks Notebook Activity**:
  - To run the `BronzeToSilver` and `SilverToGold` notebooks.
  - Set up parameters and link to the appropriate notebooks.

#### Configure the Pipeline

1. In Azure Data Factory, create a new pipeline named `MedallionPipeline`.
2. Add the activities defined above in the correct sequence.
3. Set up any necessary parameters and variables.

### 8. Schedule Pipeline Execution

#### Create a Trigger

- **Daily Schedule Trigger**:
  - Set up a trigger to execute the pipeline daily at a specified time.
  - Configure time zone and recurrence settings.

#### Activate the Trigger

- Enable the trigger to start scheduling pipeline runs.

---

## Additional Considerations

- **Security**:
  - Use Azure Key Vault to store secrets like Storage Account keys and Databricks tokens.
  - Implement role-based access control (RBAC) for resources.

- **Monitoring and Logging**:
  - Enable diagnostic settings for Azure Data Factory and Databricks.
  - Set up alerts for pipeline failures or critical events.

- **Error Handling**:
  - Implement retry policies and error handling in your pipelines and notebooks.

- **Compliance**:
  - Ensure compliance with any industry-specific regulations (e.g., GDPR, HIPAA).

---

## Cleaning Up Resources

To avoid incurring unnecessary charges, you can delete the resource group when it's no longer needed:

```bash
az group delete --name <YourResourceGroupName> --yes --no-wait
```

---

## Complete Project Files

### README.md

*(This file is the README.md content.)*

### Azure_Medallion_Pipeline_Setup.ipynb

Create a file named `Azure_Medallion_Pipeline_Setup.ipynb` with the following content:

<details>
<summary>Click to expand the notebook content</summary>

```json
{
 "cells": [
  {
   "cell_type": "markdown",
   "id": "introduction",
   "metadata": {},
   "source": [
    "# Azure Medallion Data Pipeline Setup\n",
    "\n",
    "This notebook will guide you through setting up the necessary Azure infrastructure for a medallion-style data pipeline. We'll use Azure CLI commands within this notebook to create resources such as:\n",
    "\n",
    "- **Resource Group**\n",
    "- **Storage Account** with containers for **Bronze**, **Silver**, and **Gold** layers\n",
    "- **Azure Data Factory**\n",
    "- **Azure Databricks Workspace**\n",
    "- *(Optional)* **Azure Key Vault**\n",
    "\n",
    "**Prerequisites:**\n",
    "\n",
    "- Azure CLI installed and configured\n",
    "- Azure account with sufficient permissions\n",
    "- Jupyter extension installed in VS Code\n",
    "\n",
    "**Note:** This notebook assumes you're running it in a Unix-like environment (e.g., Linux, macOS, or Windows Subsystem for Linux)."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "check_azure_cli",
   "metadata": {},
   "outputs": [],
   "source": [
    "import os\n",
    "import subprocess\n",
    "import json\n",
    "\n",
    "# Function to check if Azure CLI is installed\n",
    "def check_azure_cli():\n",
    "    try:\n",
    "        subprocess.run(['az', '--version'], check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)\n",
    "        print(\"Azure CLI is installed.\")\n",
    "    except subprocess.CalledProcessError:\n",
    "        print(\"Azure CLI is not installed. Please install it before running this notebook.\")\n",
    "        raise\n",
    "\n",
    "# Check if Azure CLI is installed\n",
    "check_azure_cli()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "user_inputs",
   "metadata": {},
   "source": [
    "## User Inputs\n",
    "\n",
    "Please provide the required information in the following cell."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "user_input_variables",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Replace the placeholder values with your actual Azure resource details.\n",
    "\n",
    "# Resource Group Name\n",
    "RESOURCE_GROUP = 'MedallionDataPipelineRG'  # e.g., 'MyResourceGroup'\n",
    "\n",
    "# Azure Region\n",
    "LOCATION = 'eastus'  # e.g., 'eastus', 'westus2'\n",
    "\n",
    "# Storage Account Name (must be globally unique, 3-24 lowercase letters and numbers)\n",
    "STORAGE_ACCOUNT_NAME = 'medallionstorage123'  # e.g., 'mystorageaccount'\n",
    "\n",
    "# Data Factory Name\n",
    "DATA_FACTORY_NAME = 'MedallionADF'  # e.g., 'MyDataFactory'\n",
    "\n",
    "# Databricks Workspace Name\n",
    "DATABRICKS_WORKSPACE = 'MedallionDatabricksWS'  # e.g., 'MyDatabricksWorkspace'\n",
    "\n",
    "# Create Azure Key Vault?\n",
    "CREATE_KEY_VAULT = True  # Set to False if you don't want to create a Key Vault\n",
    "\n",
    "# Key Vault Name (if CREATE_KEY_VAULT is True)\n",
    "KEY_VAULT_NAME = 'MedallionKeyVault123'  # e.g., 'MyKeyVault'\n",
    "\n",
    "# Ensure storage account name is lowercase and between 3-24 characters\n",
    "STORAGE_ACCOUNT_NAME = STORAGE_ACCOUNT_NAME.lower()\n",
    "if len(STORAGE_ACCOUNT_NAME) < 3 or len(STORAGE_ACCOUNT_NAME) > 24:\n",
    "    raise ValueError(\"Storage account name must be between 3 and 24 characters.\")\n",
    "\n",
    "print(\"You have provided the following information:\")\n",
    "print(\"--------------------------------------------\")\n",
    "print(f\"Resource Group Name       : {RESOURCE_GROUP}\")\n",
    "print(f\"Location                  : {LOCATION}\")\n",
    "print(f\"Storage Account Name      : {STORAGE_ACCOUNT_NAME}\")\n",
    "print(f\"Data Factory Name         : {DATA_FACTORY_NAME}\")\n",
    "print(f\"Databricks Workspace Name : {DATABRICKS_WORKSPACE}\")\n",
    "if CREATE_KEY_VAULT:\n",
    "    print(f\"Key Vault Name            : {KEY_VAULT_NAME}\")\n",
    "print(\"--------------------------------------------\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "confirm_inputs",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Confirm the inputs\n",
    "confirm = input('Is this information correct? (yes/no): ')\n",
    "if confirm.lower() != 'yes':\n",
    "    raise Exception('Setup aborted by the user.')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "azure_login",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Check Azure login status\n",
    "print(\"Checking Azure login status...\")\n",
    "try:\n",
    "    subprocess.run(['az', 'account', 'show'], check=True, stdout=subprocess.PIPE)\n",
    "    print(\"Already logged in to Azure.\")\n",
    "except subprocess.CalledProcessError:\n",
    "    print(\"You are not logged in to Azure CLI. Please log in.\")\n",
    "    subprocess.run(['az', 'login'], check=True)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_resource_group",
   "metadata": {},
   "source": [
    "## Create Resource Group"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "create_rg",
   "metadata": {},
   "outputs": [],
   "source": [
    "print(\"Creating Resource Group...\")\n",
    "subprocess.run([\n",
    "    'az', 'group', 'create',\n",
    "    '--name', RESOURCE_GROUP,\n",
    "    '--location', LOCATION\n",
    "], check=True)\n",
    "print(\"Resource Group created.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_storage_account",
   "metadata": {},
   "source": [
    "## Create Storage Account"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "create_storage",
   "metadata": {},
   "outputs": [],
   "source": [
    "print(\"Creating Storage Account...\")\n",
    "subprocess.run([\n",
    "    'az', 'storage', 'account', 'create',\n",
    "    '--name', STORAGE_ACCOUNT_NAME,\n",
    "    '--resource-group', RESOURCE_GROUP,\n",
    "    '--location', LOCATION,\n",
    "    '--sku', 'Standard_LRS',\n",
    "    '--kind', 'StorageV2',\n",
    "    '--hierarchical-namespace', 'true'\n",
    "], check=True)\n",
    "print(\"Storage Account created.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_blob_containers",
   "metadata": {},
   "source": [
    "## Create Blob Containers for Bronze, Silver, and Gold Layers"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "create_containers",
   "metadata": {},
   "outputs": [],
   "source": [
    "print(\"Creating Blob Containers...\")\n",
    "\n",
    "# Get storage account key\n",
    "result = subprocess.run([\n",
    "    'az', 'storage', 'account', 'keys', 'list',\n",
    "    '--resource-group', RESOURCE_GROUP,\n",
    "    '--account-name', STORAGE_ACCOUNT_NAME,\n",
    "    '--query', '[0].value',\n",
    "    '-o', 'tsv'\n",
    "], check=True, stdout=subprocess.PIPE)\n",
    "ACCOUNT_KEY = result.stdout.decode('utf-8').strip()\n",
    "\n",
    "for container in ['bronze', 'silver', 'gold']:\n",
    "    print(f\"Creating container '{container}'...\")\n",
    "    subprocess.run([\n",
    "        'az', 'storage', 'container', 'create',\n",
    "        '--name', container,\n",
    "        '--account-name', STORAGE_ACCOUNT_NAME,\n",
    "        '--account-key', ACCOUNT_KEY\n",
    "    ], check=True)\n",
    "print(\"Blob Containers created.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_data_factory",
   "metadata": {},
   "source": [
    "## Create Azure Data Factory"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "create_adf",
   "metadata": {},
   "outputs": [],
   "source": [
    "print(\"Creating Azure Data Factory...\")\n",
    "subprocess.run([\n",
    "    'az', 'datafactory', 'create',\n",
    "    '--resource-group', RESOURCE_GROUP,\n",
    "    '--factory-name', DATA_FACTORY_NAME,\n",
    "    '--location', LOCATION\n",
    "], check=True)\n",
    "print(\"Azure Data Factory created.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_databricks_workspace",
   "metadata": {},
   "source": [
    "## Create Azure Databricks Workspace"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "create_databricks",
   "metadata": {},
   "outputs": [],
   "source": [
    "print(\"Creating Azure Databricks Workspace...\")\n",
    "subprocess.run([\n",
    "    'az', 'databricks', 'workspace', 'create',\n",
    "    '--resource-group', RESOURCE_GROUP,\n",
    "    '--name', DATABRICKS_WORKSPACE,\n",
    "    '--location', LOCATION,\n",
    "    '--sku', 'standard'\n",
    "], check=True)\n",
    "print(\"Azure Databricks Workspace created.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_key_vault",
   "metadata": {},
   "source": [
    "## (Optional) Create Azure Key Vault"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "create_keyvault",
   "metadata": {},
   "outputs": [],
   "source": [
    "if CREATE_KEY_VAULT:\n",
    "    print(\"Creating Azure Key Vault...\")\n",
    "    subprocess.run([\n",
    "        'az', 'keyvault', 'create',\n",
    "        '--name', KEY_VAULT_NAME,\n",
    "        '--resource-group', RESOURCE_GROUP,\n",
    "        '--location', LOCATION\n",
    "    ], check=True)\n",
    "    print(\"Azure Key Vault created.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "next_steps",
   "metadata": {},
   "source": [
    "## Next Steps\n",
    "\n",
    "The basic Azure infrastructure for your medallion-style data pipeline has been set up. Here are the next steps to complete your pipeline configuration:"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "databricks_configuration",
   "metadata": {},
   "source": [
    "### 1. Configure Databricks Workspace\n",
    "\n",
    "- Obtain the fully qualified domain name (FQDN) of your Databricks workspace from the Azure Portal.\n",
    "- Generate a Personal Access Token (PAT) in the Databricks UI under **User Settings** > **Access Tokens**.\n",
    "- Store the PAT securely, preferably in Azure Key Vault."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "data_factory_linked_services",
   "metadata": {},
   "source": [
    "### 2. Set Up Linked Services in Azure Data Factory\n",
    "\n",
    "- Create linked services for your Storage Account and Databricks workspace.\n",
    "- Use the Storage Account key or reference it from Key Vault.\n",
    "- Configure the Databricks linked service using the workspace URL and PAT."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_datasets",
   "metadata": {},
   "source": [
    "### 3. Create Datasets in Azure Data Factory\n",
    "\n",
    "- Define datasets for the Bronze, Silver, and Gold layers.\n",
    "- Specify the container names and paths."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "develop_databricks_notebooks",
   "metadata": {},
   "source": [
    "### 4. Develop Databricks Notebooks\n",
    "\n",
    "- Create notebooks for data transformations:\n",
    "  - **BronzeToSilver**: Read from Bronze, transform, write to Silver.\n",
    "  - **SilverToGold**: Read from Silver, transform, write to Gold.\n",
    "- Use Spark to read and write data, applying necessary transformations."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "create_pipelines",
   "metadata": {},
   "source": [
    "### 5. Create Pipelines in Azure Data Factory\n",
    "\n",
    "- Define activities to orchestrate data movement and transformation.\n",
    "- Configure parameters and variables as needed."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "set_up_scheduling",
   "metadata": {},
   "source": [
    "### 6. Set Up Scheduling and Triggers\n",
    "\n",
    "- Determine the frequency and timing of pipeline runs.\n",
    "- Create triggers in Azure Data Factory."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "security_and_compliance",
   "metadata": {},
   "source": [
    "### 7. Security and Compliance\n",
    "\n",
    "- Use Azure Key Vault to store secrets.\n",
    "- Implement role-based access control (RBAC).\n",
    "- Ensure compliance with any regulatory requirements."
   ]
  },
  {
   "cell_type": "markdown",
   "id": "cleanup_resources",
   "metadata": {},
   "source": [
    "## Clean Up Resources\n",
    "\n",
    "To avoid incurring unnecessary charges, you can delete the resource group when it's no longer needed."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "cleanup",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Uncomment the following lines to delete the resource group\n",
    "# print(\"Deleting Resource Group...\")\n",
    "# subprocess.run([\n",
    "#     'az', 'group', 'delete',\n",
    "#     '--name', RESOURCE_GROUP,\n",
    "#     '--yes', '--no-wait'\n",
    "# ], check=True)\n",
    "# print(\"Resource Group deleted.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "conclusion",
   "metadata": {},
   "source": [
    "## Conclusion\n",
    "\n",
    "You have successfully set up the Azure infrastructure for your medallion-style data pipeline. Proceed with configuring the remaining components as outlined in the next steps."
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "name": "python"
  },
  "orig_nbformat": 4
 },
 "nbformat": 4,
 "nbformat_minor": 5
}
```

</details>

---

### setup_medallion_pipeline.sh

Create a file named `setup_medallion_pipeline.sh` with the following content:

```bash
#!/bin/bash

# Azure Medallion Data Pipeline Setup Script

# Function to check if Azure CLI is installed
function check_azure_cli() {
    if ! command -v az &> /dev/null
    then
        echo "Azure CLI could not be found. Please install it before running this script."
        exit 1
    fi
}

# Check if Azure CLI is installed
check_azure_cli

# Prompt for required information
echo "=== Azure Medallion Data Pipeline Setup ==="

read -p "Enter the name for the Resource Group: " RESOURCE_GROUP
read -p "Enter the Azure region (e.g., eastus): " LOCATION

# Storage Account names must be globally unique and use only lowercase letters and numbers
while true; do
    read -p "Enter a globally unique name for the Storage Account (3-24 lowercase letters and numbers): " STORAGE_ACCOUNT_NAME
    if [[ "$STORAGE_ACCOUNT_NAME" =~ ^[a-z0-9]{3,24}$ ]]; then
        break
    else
        echo "Invalid Storage Account name. Please use 3-24 lowercase letters and numbers."
    fi
done

read -p "Enter the name for Azure Data Factory: " DATA_FACTORY_NAME
read -p "Enter the name for Azure Databricks Workspace: " DATABRICKS_WORKSPACE
read -p "Do you want to create a new Azure Key Vault? (yes/no): " CREATE_KEY_VAULT

if [[ "$CREATE_KEY_VAULT" == "yes" ]]; then
    read -p "Enter the name for Azure Key Vault: " KEY_VAULT_NAME
fi

# Confirm the inputs
echo ""
echo "You have provided the following information:"
echo "--------------------------------------------"
echo "Resource Group Name       : $RESOURCE_GROUP"
echo "Location                  : $LOCATION"
echo "Storage Account Name      : $STORAGE_ACCOUNT_NAME"
echo "Data Factory Name         : $DATA_FACTORY_NAME"
echo "Databricks Workspace Name : $DATABRICKS_WORKSPACE"
if [[ "$CREATE_KEY_VAULT" == "yes" ]]; then
    echo "Key Vault Name            : $KEY_VAULT_NAME"
fi
echo "--------------------------------------------"

read -p "Is this information correct? (yes/no): " CONFIRM

if [[ "$CONFIRM" != "yes" ]]; then
    echo "Setup aborted by the user."
    exit 1
fi

# Login to Azure if not already logged in
echo "Checking Azure login status..."
az account show &> /dev/null
if [ $? != 0 ]; then
    echo "You are not logged in to Azure CLI. Please log in."
    az login
fi

# Begin resource creation
echo ""
echo "Creating Resource Group..."
az group create --name "$RESOURCE_GROUP" --location "$LOCATION"

echo ""
echo "Creating Storage Account..."
az storage account create \
    --name "$STORAGE_ACCOUNT_NAME" \
    --resource-group "$RESOURCE_GROUP" \
    --location "$LOCATION" \
    --sku Standard_LRS \
    --kind StorageV2 \
    --hierarchical-namespace true

echo ""
echo "Creating Blob Containers for Bronze, Silver, and Gold layers..."
ACCOUNT_KEY=$(az storage account keys list \
    --resource-group "$RESOURCE_GROUP" \
    --account-name "$STORAGE_ACCOUNT_NAME" \
    --query [0].value -o tsv)

az storage container create --name "bronze" --account-name "$STORAGE_ACCOUNT_NAME" --account-key "$ACCOUNT_KEY"
az storage container create --name "silver" --account-name "$STORAGE_ACCOUNT_NAME" --account-key "$ACCOUNT_KEY"
az storage container create --name "gold" --account-name "$STORAGE_ACCOUNT_NAME" --account-key "$ACCOUNT_KEY"

echo ""
echo "Creating Azure Data Factory..."
az datafactory create \
    --resource-group "$RESOURCE_GROUP" \
    --factory-name "$DATA_FACTORY_NAME" \
    --location "$LOCATION"

echo ""
echo "Creating Azure Databricks Workspace..."
az databricks workspace create \
    --resource-group "$RESOURCE_GROUP" \
    --name "$DATABRICKS_WORKSPACE" \
    --location "$LOCATION" \
    --sku standard

if [[ "$CREATE_KEY_VAULT" == "yes" ]]; then
    echo ""
    echo "Creating Azure Key Vault..."
    az keyvault create \
        --name "$KEY_VAULT_NAME" \
        --resource-group "$RESOURCE_GROUP" \
        --location "$LOCATION"
fi

echo ""
echo "=== Resource setup is complete. ==="
echo ""
echo "Next Steps:"
echo "1. Provide the following information to proceed with setting up linked services and datasets:"
echo "   a. The fully qualified domain name (FQDN) of your Databricks workspace."
echo "      - You can find it in the Azure portal under the Databricks workspace properties."
echo "   b. The personal access token (PAT) for your Databricks workspace."
echo "      - Generate it in the Databricks UI under 'User Settings' > 'Access Tokens'."
echo "2. Provide details about your CSV files:"
echo "   - Schema (column names and data types)."
echo "   - Any specific parsing requirements (e.g., delimiter, encoding)."
echo "3. Specify data transformation requirements for your Databricks notebooks."
echo "   - Data cleaning steps, business logic, and preferred output formats."
echo "4. Determine the scheduling details for your pipeline:"
echo "   - Exact timing and frequency."
echo "   - Any dependencies or triggers."
echo ""
echo "With this information, you can update the linked services, datasets, and pipeline scripts accordingly."
echo ""
echo "If you need assistance with these steps, please refer to the Azure documentation or reach out for support."
```

---

### push_notebook_to_github.sh

Create a file named `push_notebook_to_github.sh` with the following content:

```bash
#!/bin/bash

# Function to check if a command exists
function command_exists() {
    command -v "$1" &> /dev/null
}

# Check if Git is installed
if ! command_exists git; then
    echo "Git is not installed. Please install Git before running this script."
    exit 1
fi

# Check if GitHub CLI is installed
if ! command_exists gh; then
    echo "GitHub CLI (gh) is not installed. Please install GitHub CLI before running this script."
    exit 1
fi

# Authenticate with GitHub
echo "Authenticating with GitHub CLI..."
gh auth status &> /dev/null
if [ $? -ne 0 ]; then
    gh auth login
fi

# Get GitHub username
USERNAME=$(gh api user --jq '.login')

# Prompt for repository information
read -p "Enter the name for your new GitHub repository: " REPO_NAME
read -p "Enter a description for your repository (optional): " REPO_DESC
read -p "Is the repository private? (yes/no): " REPO_PRIVATE

if [[ "$REPO_PRIVATE" == "yes" ]]; then
    PRIVATE_FLAG="--private"
else
    PRIVATE_FLAG="--public"
fi

# Create a new GitHub repository
echo "Creating repository '$REPO_NAME' on GitHub..."
gh repo create "$USERNAME/$REPO_NAME" $PRIVATE_FLAG --description "$REPO_DESC" --confirm

# Initialize a local Git repository
echo "Initializing local Git repository..."
mkdir "$REPO_NAME"
cd "$REPO_NAME"
git init

# Copy the Jupyter notebook into the repository
echo "Copying Jupyter notebook into the repository..."
NOTEBOOK_PATH="../Azure_Medallion_Pipeline_Setup.ipynb"  # Adjust the path if necessary
if [ ! -f "$NOTEBOOK_PATH" ]; then
    echo "Notebook not found at '$NOTEBOOK_PATH'. Please update the script with the correct path."
    exit 1
fi
cp "$NOTEBOOK_PATH" .

# Add the notebook to Git
echo "Adding files to Git..."
git add Azure_Medallion_Pipeline_Setup.ipynb

# Commit the changes
echo "Committing files..."
git commit -m "Add Azure Medallion Pipeline Setup notebook"

# Set the remote repository URL
echo "Setting remote repository..."
REMOTE_URL="https://github.com/$USERNAME/$REPO_NAME.git"
git remote add origin "$REMOTE_URL"

# Push the changes to GitHub
echo "Pushing changes to GitHub..."
git branch -M main
git push -u origin main

echo "Notebook has been pushed to your GitHub repository successfully."
```

---

## Conclusion

By combining all the instructions, scripts, and files into this single markdown file, you now have a comprehensive guide and all the necessary code to set up your Azure Medallion Data Pipeline. You can copy the contents of each file into separate files on your local machine, follow the setup instructions, and run the project.

If you have any questions or need further assistance, feel free to reach out!

---

**Happy Coding!**
