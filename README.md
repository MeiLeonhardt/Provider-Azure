# Provider-Azure

In Terraform, il provider è un plugin che consente di interagire con un cloud provider per poter creare l’infrastruttura all’interno di quel cloud. 

Quando andiamo a creare un’infrastruttura, infatti, il primo blocco sarà provvisto del comando “provider” seguito dal nome del provider, es. ```azurerm```, ```features``` e ```subscription```:

```
provider "azurerm" {
  features {}
  subscription_Id = Id-della-sottoscrizione
}
```

## AzureRM
Azure Resource Manager è il servizio di distribuzione e di gestione di Azure: permette all'utente di creare, gestire, leggere, aggiornare e cancellare risorse in Azure, tra le quali VM, Storage Accounts, VNet etc...

![image](https://github.com/user-attachments/assets/624417a8-c725-4641-a1de-730713080cf4)

Questo servizio è disponibile sia da portale, che da Powershell, Azure CLI e API REST.
- Resource : le risorse in Azure sono tutti gli oggetti che possono essere configurati come VM, VNet, Stroage accounts etc...
- Resource groups : I gruppi di risorse sono contenitori logici all'interno dei quali inseriamo le risorse che abbiamo creato. Es. sto creando un'infrastruttura per lo sviluppo:
    - creo il gruppo risorse "Sviluppo"
    - quando configuro una nuova risorsa, es. VM-dev, la inserisco in quel gruppo risorse

 ## AzureAD
Azure AD è il servizio di identità e gestione degli accessi di Microsoft sul cloud. Serve per:
- Autenticare utenti e dispositivi (Single Sign-On, Multi-Factor Authentication)
- Gestire permessi e ruoli (RBAC)
- Integrazione con applicazioni (Microsoft 365, app custom)
- Autenticazione per servizi cloud come Azure, Microsoft 365, e migliaia di app SaaS

## Azure DevOps
Azure DevOps è una piattaforma di DevOps offerta da Microsoft per supportare lo sviluppo software end-to-end. Include:
- Repos – controllo di versione Git
- Pipelines – CI/CD per automatizzare build e deploy
- Boards – gestione dei task (tipo Jira)
- Test Plans – gestione e automazione dei test
- Artifacts – hosting di pacchetti (NuGet, npm, ecc.)

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
