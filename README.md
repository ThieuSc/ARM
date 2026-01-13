
# Azure Wazuh Infrastructuur (ARM Templates)

Dit repository bevat **Azure ARM-templates** voor het automatisch uitrollen van een complete infrastructuur voor een **Wazuh-omgeving**. De deployment is modulair opgezet en bestaat uit netwerken, subnetten, security (NSG/ASG), virtual machines, VNet peering en een Azure Firewall.

## Architectuuroverzicht

De infrastructuur bestaat uit:

* **Twee Virtual Networks**
    
    * `Wazuh` – voor dashboard, server en indexer
    * `ingest` – voor ingest en Azure Firewall
        
* **Subnetten**
    
    * Dashboard, Server, Indexer
    * Ingest, AzureFirewallSubnet, AzureFirewallManagementSubnet
        
* **Security**
    
    * Network Security Groups (NSG)
    * Application Security Groups (ASG)
        
* **Compute**
    
    * Linux Virtual Machines (Ubuntu 24.04 LTS)
        
* **Netwerk**
    
    * VNet Peering tussen `Wazuh` en `ingest`
        
* **Firewall**
    
    * Azure Firewall (Basic) met publieke IP’s
        

Alles wordt centraal aangestuurd via `main.json`.

## Repository structuur

```text
.
├── main.json
├── main.parameters.json
├── deployment_network.json
├── deployment_subnet_security.json
├── deployment_network_peering.json
├── deployment_virtualmachines.json
├── deployment_firewall.json
└── README.md
```

### Bestanden

| Bestand | Omschrijving |
| --- | --- |
| `main.json` | Hoofdtemplate dat alle nested deployments uitvoert |
| `main.parameters.json` | Parameters zoals admin-gebruiker en SSH key |
| `deployment_network.json` | VNet- en subnetdefinities |
| `deployment_subnet_security.json` | NSG’s en ASG’s per subnet |
| `deployment_network_peering.json` | VNet peering tussen Wazuh en ingest |
| `deployment_virtualmachines.json` | Virtual Machines + NICs |
| `deployment_firewall.json` | Azure Firewall + publieke IP’s |

## Deployment

### Vereisten

* Azure powershell CLI
* Een bestaande Resource Group
* Een SSH public key
    

### Deploy via Azure CLI

Twéé resource groups aanmaken, bijvoorbeeld LabSec en LabSec-Stack
``` poweshell
New-AzResourceGroup -Name LabSec -Location francecentral
New-AzResourceGroup -Name LabSec-Stack -Location francecentral
```

Template toevoegen aan azure template spec
``` powershell
New-AzTemplateSpec -ResourceGroupName "LabStack" -Name "LabSec-Stack" -Version "1.0.0" -Location "francecentral" -TemplateFile ".\main.json"
```

Template ID ophalen uit azure template spec
``` powershell
$id = (Get-AzTemplateSpec -Name LabSec-Stack -ResourceGroupName LabStack -Version 1.0.0).Versions.Id
```

Public-key inlezen
``` powershell
$keypub = (Get-Content ".\.ssh\serverkey.pem.pub" -Raw).Trim()
```
Deployment naam maken
``` powershell
$today=Get-Date -Format "MM-dd-yyyy"
$deploymentName="wazuh-infra-"+"$today"
```

Deployen
``` powershell
New-azresourceGroupDeployment -Name $deploymentName -TemplateSpecId $id -TemplateParameterFile ".\main.parameters.json" -adminPublicKey $keypub
```
Cleanup
``` powershell
Get-azresource -ResourceGroupName LabSec | Remove-AzResource -Force      
```

## Parameters

### `main.parameters.json`

| Parameter | Beschrijving |
| --- | --- |
| `adminUsername` | Linux admin gebruiker voor VM’s |
| `adminPublicKey` | SSH public key voor login |

## Virtual Machines

Standaard worden de volgende VM’s aangemaakt:

| Naam | Subnet | IP |
| --- | --- | --- |
| wazuh-dashboard | dashboard | 10.10.10.10 |
| wazuh-server | server | 10.10.20.10 |
| wazuh-indexer | indexer | 10.10.30.10 |

VM’s kunnen eenvoudig **aan/uit** gezet worden via de `enabled` property in `main.json`.

## 🔐 Security

* Inbound SSH (TCP/22) toegestaan per ASG 
* Outbound internet toegestaan
* NSG’s zijn dynamisch gekoppeld aan subnetten 
* ASG’s worden gebruikt voor gerichte security rules
    

## 🧩 Modulariteit

Dit project is bewust modulair opgezet:

* Templates zijn herbruikbaar  
* Netwerk en security zijn los te beheren
* VM’s kunnen onafhankelijk worden aangepast
    

## 📌 Toekomstige uitbreidingen

* Firewall rules & policies
* Azure Bastion 
* Log Analytics / Azure Monitor
* Private Endpoints
    

## 📄 Licentie

Vrij te gebruiken voor interne en educatieve doeleinden.
