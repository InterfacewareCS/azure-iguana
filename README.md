# Automated deployment of FHIR Iguana on Azure VM

**THIS REPOSITORY IS IN DEVELOPMENT - CONFIGURATION NOT COMPLETE**

This repository contains an Azure ARM Template that is able to deploy a virtual machine containing Iguana and FHIR related channels. The Iguana instance within the template will be configured to make a connection to a specific instance of the [Azure API for FHIR](https://azure.microsoft.com/en-us/services/azure-api-for-fhir/). 

# Architecture Diagram and Description

#### Components
- **Azure Template on Github**: Azure template assists the user with a click of the <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Finterfacewarecs%2Figuana-azure-fhir%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a> button or via powershell to auto provision the Azure VM and Iguana installation 
- **Azure**: Cloud infrastructure that hosts the HL7 to FHIR POC (proof of concept)
- **Azure AD**: Azure Active Directory that provides secure access to Azure API services
- **Azure API for FHIR**: FHIR server with data storage that provides API access
- **FHIR Dashboard**: Web dashboard that displays patient demographics information from FHIR server
- **Azure VM for Iguana**: Azure virtual machine that hosts the Iguana application for converting HL7 v2 (ADT) messages to FHIR patient resources. The VM OS will be Windows Server 2016.

<center><img src="https://raw.githubusercontent.com/InterfacewareCS/iguana-azure-fhir/master/DesignDiagram.png" width="512"></center>

#### Workflow
1. Iguana sends client id and client secret to Azure AD to be authenticated
2. Azure AD sends back token to Iguana
3. Iguana sends FHIR patient resource to Azure API for FHIR
4. Clinician accesses FHIR dashboard to review Patient demographic

# Prerequisites

This template requires a FHIR Server to complete the overall workflow. To set up a sandbox environment with Azure API for FHIR, web client, etc., deploy the [Microsoft/fhir-server-samples](https://github.com/Microsoft/fhir-server-samples) scenario and collect the following information from the script and Azure keyvault secrets:

- **Aad Service Client Id:** The service id of the FHIR Server resource.
- **Aad Service Client Secret:** The service secret of the FHIR Server resource.
- **Aad Authority:** The authenticating link used for applications to connect to your domain. ex `https://login.microsoftonline.com/interfaceware.com` or `https://login.microsoftonline.com/interfaceware.onmicrosoft.com`
- **Aad Audience:** Unless this differs, this will typically be the FHIR server URL in the format: `https://<AZURE API FOR FHIR NAME>.azurehealthcareapis.com`
- **FHIR Server URL:** The FHIR Server URL. ex `[https://<AZURE API FOR FHIR NAME>.azurehealthcareapis.com]`

This information will be needed to be passed into the deployment template.

# Deployment

#### Option 1: Via Powershell script 

To deploy the template:

1. Create a parameter file `azuredeploy.parameters.json`:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "MY-VM-NAME"
        },
        "adminUsername": {
            "value": "msiguanauser"
        },
        "adminPassword": {
            "value": "<MY SUPER SECRET PASSWORD>"
        },
        "aadAuthority": {
            "value": "https://login.microsoftonline.com/<TENANT-ID>"
        },
        "aadClientId": {
            "value": "<CLIENT-ID>"
        },
        "aadClientSecret": {
            "value": "<CLIENT-SECRET>"
        },
        "aadAudience": {
            "value": "https://<AZURE API FOR FHIR NAME>.azurehealthcareapis.com"
        },
        "fhirServerUrl": {
            "value": "https://<AZURE API FOR FHIR NAME>.azurehealthcareapis.com"
        }
    }
}
```
2. Create Resource Group and Deploy the VM:
```PowerShell
# Create resource group
$rg = New-AzureRmResourceGroup -Name "resource-group-name" -Location "westus2"

# Deploy VM
New-AzureRmResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/InterfacewareCS/iguana-azure-fhir/master/azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json -ResourceGroupName $rg.ResourceGroupName
```

#### Option 2: Use the Portal:

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Finterfacewarecs%2Figuana-azure-fhir%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

#### Deployment considerations

An important consideration to note is that depending on the type of Azure resource group chosen, **deploying the ARM template may take 5-10 minutes to fully complete.** Accessing the virtual machine before completion may cancel the Iguana provisioning process.

<center><img src="https://raw.githubusercontent.com/InterfacewareCS/iguana-azure-fhir/master/images/Deploying.png" width="384"></center>

It is important to **make sure that the deployment is complete** before accessing the virtual machine.

<center><img src="https://raw.githubusercontent.com/InterfacewareCS/iguana-azure-fhir/master/images/Success.png" width="384"></center>

# After Deployment

1. After deployment, use remote desktop to connect to the deployed VM and navigate to: `http://localhost:6543`
2. [Register Iguana](https://help.interfaceware.com/v6/register-iguana) using an existing account, or by using a trial license
3. Once at the dashboard, turn on either or both CHN 1: Random HL7 ADT Message and CHN1: HL7 From File channels.
4. Navigate to the FHIR dashboard `https://<AZURE API FOR FHIR NAME>.azurewebsites.net` to see the uploaded resources.

# More Information 
- [Iguana Help Documentation on the Iguana Azure ARM Template:](https://help.interfaceware.com/v6/automated-deployment-of-fhir-iguana-on-azure-vm) 
- [Microsoft FHIR Servers Sample Documentation:](https://github.com/Microsoft/fhir-server-samples)
