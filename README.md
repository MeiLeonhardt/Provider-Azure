# Provider-Azure

In Terraform, il provider è un plugin che consente di interagire con un provider in cloud per poter creare l’infrastruttura. 

Quando andiamo a creare un’infrastruttura, infatti, il primo blocco sarà provvisto del comando ```provider``` seguito dal nome del provider, es. ```azurerm```, ```features``` e ```subscription```:

```
provider "azurerm" {
  features {} #features è obbligatorio ma può essere lasciato vuoto
  subscription_id = Id-della-sottoscrizione
}
```
Ci sono casi in cui è necessario specificare la versione del provider che si vuole utilizzare. Questo può avvenire nel caso di infrastrutture configurate in una certa versione, o quando si sta modificando un'infrastruttura che è stata condivisa da un cliente e di cui ci ha fornito i file terraform.

```
# 1. Specify the version of the AzureRM Provider to use
terraform {
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "=3.0.1"
    }
  }
}
```
## AzureRM
Azure Resource Manager è il servizio di **distribuzione e di gestione delle risorse** in Azure: permette all'utente di creare, gestire, leggere, aggiornare e cancellare risorse in Azure, tra le quali VM, Storage Accounts, VNet etc...

Con questo provider facciamo riferimento a delle **risorse cloud**, mentre con Azure AD faremo riferimento alla gestione delle **identità**.

![image](https://github.com/user-attachments/assets/624417a8-c725-4641-a1de-730713080cf4)

Questo servizio è disponibile sia da portale, che da Powershell, Azure CLI e API REST.
- **Resource **: le risorse in Azure sono tutti gli oggetti che possono essere configurati come VM, VNet, Stroage accounts etc...
- **Resource groups** : I gruppi di risorse sono contenitori logici all'interno dei quali inseriamo le risorse che abbiamo creato.

  Es. sto creando un'infrastruttura per lo sviluppo:
    - creo il gruppo risorse "Dev"
``` 
  resource "azurerm_resource_group" "Dev" {
  name     = "Dev"
  location = "West Europe"
}
```
    - quando configuro una nuova risorsa, es. VNet-dev, la inserisco in quel gruppo risorse
    
```
resource "azurerm_virtual_network" "VNet_Dev" {
  name                = "VNet-Dev"
  resource_group_name = azurerm_resource_group.Dev.name
  location            = azurerm_resource_group.Dev.location
  address_space       = ["10.0.0.0/16"]
}
```
 ## AzureAD
Azure AD è il **servizio di identità** e **gestione degli accessi** di Microsoft sul cloud. Serve per:
- **Autenticare** utenti e dispositivi (Single Sign-On, Multi-Factor Authentication)
- **Gestire** permessi e ruoli (RBAC)
- **Integrazione** con applicazioni (Microsoft 365, app custom)
- Autenticazione per servizi cloud come Azure, Microsoft 365, e altre app SaaS

Azure Provider può essere utilizzato per configuare un'infrastruttura in Azure Active Directory (Azure AD provider), utilizzando l'API Microsoft Graph.
```
# Configure Terraform
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.15.0"
    }
  }
}

# Configure the Azure Active Directory Provider
provider "azuread" {
  tenant_id = "00000000-0000-0000-0000-000000000000"
}

# Retrieve domain information
data "azuread_domains" "example" {
  only_initial = true
}

# Create an application
resource "azuread_application_registration" "example" {
  display_name = "ExampleApp"
}

# Create a service principal
resource "azuread_service_principal" "example" {
  client_id = azuread_application_registration.example.client_id
}

# Create a user
resource "azuread_user" "example" {
  user_principal_name = "ExampleUser@${data.azuread_domains.example.domains.0.domain_name}"
  display_name        = "Example User"
  password            = "..." #in questo caso si possono usare sia le variabili, che Azure Key Vault per la gestione sicura della password
}
```

## Azure DevOps
Azure DevOps è una piattaforma di DevOps offerta da Microsoft per supportare lo **sviluppo software end-to-end** (dallo sviluppo al monitoraggio). 
Include:
- **Repos** – controllo di versione Git
- **Pipelines** – CI/CD per automatizzare build e deploy
- **Boards** – gestione dei task (tipo Jira)
- **Test Plans** – gestione e automazione dei test
- **Artifacts** – hosting di pacchetti (NuGet, npm, ecc.)

Il ****provider** ```azuredevops``` consente di gestire e configurare risorse di Azure DevOps tramite Terraform, sfruttando le REST API di Azure DevOps. Può essere usato per creare **progetti, pipeline, repository, gruppi di sicurezza, permessi**, e molto altro.
```
terraform {
  required_providers {
    azuredevops = {
      source  = "microsoft/azuredevops"
      version = ">= 0.1.0"
    }
  }
}

resource "azuredevops_project" "project" {
  name        = "Project Name"
  description = "Project Description"
}
```

Per il processo di autenticazione è possibile utilizzare il servizio principale di Azure AD, attraverso EntraID, oppure tramite accesso personale con token.
**Metodi di autenticazione:**
- Personal Access Token > per ambienti locali o pipeline. Si genera da Azure DevOps con scadenza e permessi specifici;
- OpenID Connect per l'integrazione con sistemi automatizzati come GitHub Actions e Terraform Cloud
    - ```use_oidc = true``` : per usare direttamente un token OIDC
    - ```oidc_token_file_path```: percorso a un file con il token
    - ```oidc_token_request_url```: URL per ottenere un token dinamicamente
- Client Certificate Path : ```client_certificate_path```
-  ```client_id ```,  ```client_secret ```,  ```tenant_id ``` : autenticazione via app registrata
- ```use_msi = true```: : supporta Managed Identity (es. in ambienti Azure come VM, App Services)



## Azure API Provider
AzAPI è un provider Terraform che ti permette di gestire risorse Azure che non sono ancora supportate ufficialmente dai provider classici (azurerm). Serve per usare le REST API di Azure direttamente da Terraform, utile per:
- Risorse in anteprima (preview)
- Feature sperimentali
- Personalizzazioni particolari

## Azure Stack
Azure Stack è una estensione di Azure per ambienti on-premises o ibridi. Consente di eseguire i servizi Azure in un proprio datacenter. Esistono diverse versioni:
- Azure Stack Hub – piattaforma completa per eseguire servizi Azure localmente
- Azure Stack HCI – soluzioni ibride per infrastruttura iperconvergente
- Azure Stack Edge – appliance per elaborazione locale ed edge computing

_________________________________________________

# Azure DevOps : creare una Pipeline con Terraform
Azure DevOps: creare una Pipeline con Terraform
Per creare una pipeline Terraform in Azure DevOps, questi sono i passaggi base:

1. Requisiti
Codice Terraform in un repository Git (es. main.tf)

Service Connection verso Azure (Azure Resource Manager con SPN)

2. Pipeline YAML di base
file.yaml
```
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  TF_VERSION: '1.5.7'
  AZURE_SUBSCRIPTION: 'Nome-Connessione-Azure'

steps:
- task: UseTerraform@0
  inputs:
    terraformVersion: $(TF_VERSION)

- script: |
    terraform init
    terraform validate
  workingDirectory: $(System.DefaultWorkingDirectory)
  displayName: 'Init & Validate'

- script: |
    terraform plan -out=tfplan
  workingDirectory: $(System.DefaultWorkingDirectory)
  displayName: 'Terraform Plan'

- script: |
    terraform apply -auto-approve tfplan
  workingDirectory: $(System.DefaultWorkingDirectory)
  displayName: 'Terraform Apply'
```

3. Service Connection
- Vai su Project Settings > Service connections > New connection
- Scegli Azure Resource Manager, e configura con un Service Principal

4. Terraform State
Idealmente salva lo stato remoto in un Azure Storage Account (backend configurato in main.tf)
