# 📘 Guía de Aprovisionamiento — POC RAG GPT

Guía completa y granular para desplegar la infraestructura GPT-RAG empresarial en Azure con la nomenclatura de recursos definida por el cliente **ASIMED / GuiCli**.

---

## 📑 Tabla de Contenidos

1. [Tabla de Mapeo de Nomenclatura](#1-tabla-de-mapeo-de-nomenclatura)
2. [Direccionamiento IP](#2-direccionamiento-ip)
3. [Pre-requisitos](#3-pre-requisitos)
4. [Paso a Paso de Aprovisionamiento](#4-paso-a-paso-de-aprovisionamiento)
   - [Fase 1 — Clonar e Inicializar](#fase-1--clonar-e-inicializar)
   - [Fase 2 — Autenticación](#fase-2--autenticación)
   - [Fase 3 — Configurar Variables (`azd env set`)](#fase-3--configurar-variables-azd-env-set)
   - [Fase 4 — Alternativa: Archivo `.env` Directo](#fase-4--alternativa-archivo-env-directo)
   - [Fase 5 — Log Analytics (Pre-creación Manual)](#fase-5--log-analytics-pre-creación-manual)
   - [Fase 6 — Provisionar Infraestructura](#fase-6--provisionar-infraestructura)
   - [Fase 7 — Despliegue de Componentes](#fase-7--despliegue-de-componentes)
   - [Fase 8 — Subir Documentos de Prueba](#fase-8--subir-documentos-de-prueba)
   - [Fase 9 — Verificación Post-Despliegue](#fase-9--verificación-post-despliegue)
5. [Notas y Consideraciones Importantes](#5-notas-y-consideraciones-importantes)
6. [Troubleshooting](#6-troubleshooting)

---

## 1. Tabla de Mapeo de Nomenclatura

Mapeo completo entre el nombre de recurso definido por el cliente, la variable de entorno `azd` que lo controla y el parámetro Bicep correspondiente en `infra/main.bicep`.

| Recurso | Nombre del Cliente | Variable `azd` | Parámetro Bicep |
|---|---|---|---|
| Function Ingestion | `func-asimed-guicli-ingest-poc-s` | `AZURE_DATA_INGEST_FUNC_NAME` | `dataIngestionFunctionAppName` |
| Function Orchestrator | `func-asimed-guicli-orques-poc-s` | `AZURE_ORCHESTRATOR_FUNCTION_APP_NAME` | `orchestratorFunctionAppName` |
| App Service | `app-serv-asimed-guicli-poc-s` | `AZURE_APP_SERVICE_NAME` | `appServiceName` |
| App Service Plan | `asp-asimed-guicli-manage-poc-s` | `AZURE_APP_SERVICE_PLAN_NAME` | `appServicePlanName` |
| CosmosDB | `cosdb-asimed-guicli-manage-poc-s` | `AZURE_DB_ACCOUNT_NAME` | `azureDbConfig.dbAccountName` |
| Key Vault | `kv-asimed-guicli-poc-s` | `AZURE_KEY_VAULT_NAME` | `keyVaultName` |
| AI Foundry / AI Services | `aif-asimed-guicli-poc-s` | `AZURE_AI_SERVICES_NAME` | `aiServicesName` |
| AI Search | `ais-asimed-guicli-poc-s` | `AZURE_SEARCH_SERVICE_NAME` | `searchServiceName` |
| Storage Account | `stasimedguiclipocs` | `AZURE_STORAGE_ACCOUNT_NAME` | `storageAccountName` |
| App Insights | `appi-asimed-guicli-manage-poc-s` | `AZURE_APP_INSIGHTS_NAME` | `appInsightsName` |
| Log Analytics | `LogA-AsistenteMedico-poc-s` | `LOG_ANALYTICS_WORKSPACE_ID` (Resource ID completo) | — (pre-creado, se referencia por ID) |
| VNet | `vnet-asimed-guicli-main-poc-s` | `AZURE_VNET_NAME` | `vnetName` |
| Subnet AI | `snet-asimed-guicli-ai-poc-s` | `AZURE_AI_SUBNET_NAME` | `aiSubnetName` |
| Subnet App Int | `snet-asimed-guicli-appint-poc-s` | `AZURE_APP_INT_SUBNET_NAME` | `appIntSubnetName` |
| Subnet App Services | `snet-asimed-guicli-app-poc-s` | `AZURE_APP_SERVICES_SUBNET_NAME` | `appServicesSubnetName` |
| Subnet Database | `snet-asimed-guicli-db-poc-s` | `AZURE_DATABASE_SUBNET_NAME` | `databaseSubnetName` |
| PE Storage | `pe-storage-asimed-guicli-manage-poc-s` | `AZURE_STORAGE_ACCOUNT_PE` | `azureStorageAccountPe` |
| PE CosmosDB | `pe-cosmos-asimed-guicli-manage-poc-s` | `AZURE_DB_ACCOUNT_PE` | `azureDbAccountPe` |
| PE KeyVault | `pe-kv-asimed-guicli-manage-poc-s` | `AZURE_KEYVAULT_PE` | `azureKeyvaultPe` |
| PE Orchestrator | `pe-orc-asimed-guicli-manage-poc-s` | `AZURE_ORCHESTRATOR_PE` | `azureOrchestratorPe` |
| PE Frontend | `pe-frontend-asimed-guicli-manage-poc-s` | `AZURE_FRONTEND_PE` | `azureFrontendPe` |
| PE Ingestion | `pe-ingest-asimed-guicli-manage-poc-s` | `AZURE_DATA_INGESTION_PE` | `azureDataIngestionPe` |
| PE AI Services | `pe-ai-asimed-guicli-manage-poc-s` | `AZURE_AI_SERVICES_PE` | `azureAiServicesPe` |
| PE OpenAI | `pe-oai-asimed-guicli-manage-poc-s` | `AZURE_OPEN_AI_PE` | `azureOpenAiPe` |
| PE AI Search | `pe-search-asimed-guicli-manage-poc-s` | `AZURE_SEARCH_PE` | `azureSearchPe` |
| VM | `vm-asimed-guicli-poc-s` | `AZURE_VM_NAME` | `ztVmName` |
| Key Vault Bastion | `kv-bastion-asimed-poc-s` | `AZURE_BASTION_KV_NAME` | `bastionKvName` |

> **Nota sobre Document Intelligence**: El servicio de Document Intelligence (di-asimed-guicli-manage-poc-s) se provisiona automáticamente **dentro** del recurso multi-service de Azure AI Services (`aif-asimed-guicli-poc-s`). No requiere un recurso separado en esta arquitectura.

---

## 2. Direccionamiento IP

Configuración de red para la VNet `vnet-asimed-guicli-main-poc-s`:

| Subnet | Nombre | CIDR | Variable `azd` |
|---|---|---|---|
| **Address Space** | — | `10.0.0.0/23` | `AZURE_VNET_ADDRESS` |
| **AI Subnet** | `snet-asimed-guicli-ai-poc-s` | `10.0.0.0/26` | `AZURE_AI_SUBNET_PREFIX` |
| **Bastion Subnet** | `AzureBastionSubnet` ⚠️ | `10.0.0.64/26` | `AZURE_BASTION_SUBNET_PREFIX` |
| **App Int Subnet** | `snet-asimed-guicli-appint-poc-s` | `10.0.0.128/26` | `AZURE_APP_INT_SUBNET_PREFIX` |
| **App Services Subnet** | `snet-asimed-guicli-app-poc-s` | `10.0.0.192/26` | `AZURE_APP_SERVICES_SUBNET_PREFIX` |
| **Database Subnet** | `snet-asimed-guicli-db-poc-s` | `10.0.1.0/26` | `AZURE_DATABASE_SUBNET_PREFIX` |

> ⚠️ **AzureBastionSubnet**: Azure exige que la subnet de Bastion se llame **exactamente** `AzureBastionSubnet`. No se puede cambiar este nombre. El parámetro `AZURE_BASTION_SUBNET_PREFIX` controla únicamente el CIDR.

---

## 3. Pre-requisitos

### Herramientas necesarias

```sh
# 1. Verificar Azure Developer CLI (>= 1.5.0)
azd version

# 2. Verificar Azure CLI
az version

# 3. Verificar Git
git --version

# 4. Verificar Python (debe ser 3.11)
python --version

# 5. Verificar Node.js (>= 16)
node --version
npm --version

# 6. Verificar PowerShell (>= 7, solo Windows)
pwsh --version
```

### Instalación rápida (si falta alguna herramienta)

```sh
# Azure Developer CLI
curl -fsSL https://aka.ms/install-azd.sh | bash

# Azure CLI (Linux/Mac)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Azure CLI (Windows)
winget install Microsoft.AzureCLI
```

### Permisos necesarios en Azure

- **Contributor** o **Owner** en la suscripción (para crear recursos)
- **User Access Administrator** (para asignar roles RBAC a identidades administradas)
- El Resource Group del cliente debe existir previamente o tener permisos para crearlo

---

## 4. Paso a Paso de Aprovisionamiento

### Fase 1 — Clonar e Inicializar

```sh
# Crear un directorio de trabajo
mkdir poc-rag-gpt && cd poc-rag-gpt

# Inicializar el proyecto azd desde el template oficial de Azure GPT-RAG
azd init -t azure/gpt-rag -b release/1.0.1
```

> `azd init` crea la carpeta `.azure/<env-name>/` con los archivos de entorno. Se te pedirá el nombre del entorno — usa `asimed-guicli-poc`.

```sh
# Cuando pregunte el nombre del entorno:
# ? Enter a new environment name: asimed-guicli-poc
```

---

### Fase 2 — Autenticación

```sh
# Login con Azure Developer CLI
azd auth login

# Login con Azure CLI (necesario para scripts de pre/post provision)
az login

# Verificar la suscripción activa
az account show

# Si necesitas cambiar la suscripción activa:
az account set --subscription "<subscription-id-del-cliente>"

# Confirmación
azd env set AZURE_SUBSCRIPTION_ID "$(az account show --query id -o tsv)"
```

---

### Fase 3 — Configurar Variables (`azd env set`)

Ejecuta los siguientes comandos para configurar todas las variables del entorno `azd`. Los comandos están organizados por categoría para facilidad de lectura.

#### 3.1 — Configuración Base

```sh
# Nombre del entorno azd
azd env set AZURE_ENV_NAME "asimed-guicli-poc"

# Región de despliegue
azd env set AZURE_LOCATION "eastus2"

# Nombre del Resource Group pre-existente del cliente
azd env set AZURE_RESOURCE_GROUP_NAME "<nombre-rg-cliente>"

# Habilitar aislamiento de red Zero Trust
azd env set AZURE_NETWORK_ISOLATION "true"
```

#### 3.2 — Nombres de Recursos: Cómputo y App

```sh
azd env set AZURE_DATA_INGEST_FUNC_NAME          "func-asimed-guicli-ingest-poc-s"
azd env set AZURE_ORCHESTRATOR_FUNCTION_APP_NAME "func-asimed-guicli-orques-poc-s"
azd env set AZURE_APP_SERVICE_NAME               "app-serv-asimed-guicli-poc-s"
azd env set AZURE_APP_SERVICE_PLAN_NAME          "asp-asimed-guicli-manage-poc-s"
```

#### 3.3 — Nombres de Recursos: Datos y Seguridad

```sh
azd env set AZURE_DB_ACCOUNT_NAME        "cosdb-asimed-guicli-manage-poc-s"
azd env set AZURE_STORAGE_ACCOUNT_NAME  "stasimedguiclipocs"
azd env set AZURE_KEY_VAULT_NAME        "kv-asimed-guicli-poc-s"
```

#### 3.4 — Nombres de Recursos: AI y Búsqueda

```sh
azd env set AZURE_AI_SERVICES_NAME     "aif-asimed-guicli-poc-s"
azd env set AZURE_OPENAI_SERVICE_NAME  "aif-asimed-guicli-poc-s"
azd env set AZURE_SEARCH_SERVICE_NAME  "ais-asimed-guicli-poc-s"
```

#### 3.5 — Nombres de Recursos: Monitoreo

```sh
azd env set AZURE_APP_INSIGHTS_NAME "appi-asimed-guicli-manage-poc-s"
```

#### 3.6 — Networking: VNet y Subnets

```sh
azd env set AZURE_VNET_NAME               "vnet-asimed-guicli-main-poc-s"
azd env set AZURE_VNET_ADDRESS            "10.0.0.0/23"

azd env set AZURE_AI_SUBNET_NAME          "snet-asimed-guicli-ai-poc-s"
azd env set AZURE_AI_SUBNET_PREFIX        "10.0.0.0/26"

azd env set AZURE_BASTION_SUBNET_PREFIX   "10.0.0.64/26"

azd env set AZURE_APP_INT_SUBNET_NAME     "snet-asimed-guicli-appint-poc-s"
azd env set AZURE_APP_INT_SUBNET_PREFIX   "10.0.0.128/26"

azd env set AZURE_APP_SERVICES_SUBNET_NAME   "snet-asimed-guicli-app-poc-s"
azd env set AZURE_APP_SERVICES_SUBNET_PREFIX "10.0.0.192/26"

azd env set AZURE_DATABASE_SUBNET_NAME   "snet-asimed-guicli-db-poc-s"
azd env set AZURE_DATABASE_SUBNET_PREFIX "10.0.1.0/26"
```

#### 3.7 — Private Endpoints

```sh
azd env set AZURE_STORAGE_ACCOUNT_PE  "pe-storage-asimed-guicli-manage-poc-s"
azd env set AZURE_DB_ACCOUNT_PE       "pe-cosmos-asimed-guicli-manage-poc-s"
azd env set AZURE_KEYVAULT_PE         "pe-kv-asimed-guicli-manage-poc-s"
azd env set AZURE_ORCHESTRATOR_PE     "pe-orc-asimed-guicli-manage-poc-s"
azd env set AZURE_FRONTEND_PE         "pe-frontend-asimed-guicli-manage-poc-s"
azd env set AZURE_DATA_INGESTION_PE   "pe-ingest-asimed-guicli-manage-poc-s"
azd env set AZURE_AI_SERVICES_PE      "pe-ai-asimed-guicli-manage-poc-s"
azd env set AZURE_OPEN_AI_PE          "pe-oai-asimed-guicli-manage-poc-s"
azd env set AZURE_SEARCH_PE           "pe-search-asimed-guicli-manage-poc-s"
```

#### 3.8 — VM y Bastion (Zero Trust)

```sh
azd env set AZURE_VM_DEPLOY_VM    "true"
azd env set AZURE_VM_NAME         "vm-asimed-guicli-poc-s"
azd env set AZURE_VM_USER_NAME    "gptrag"
azd env set AZURE_BASTION_KV_NAME "kv-bastion-asimed-poc-s"
azd env set AZURE_VM_KV_SEC_NAME  "vmUserInitialPassword"
```

#### 3.9 — Modelos AI (GPT-4o + Embeddings)

```sh
# Modelo de chat GPT-4o
azd env set AZURE_CHAT_GPT_MODEL_NAME          "gpt-4o"
azd env set AZURE_CHAT_GPT_MODEL_VERSION       "2024-11-20"
azd env set AZURE_CHAT_GPT_DEPLOYMENT_NAME     "chat"
azd env set AZURE_CHAT_GPT_DEPLOYMENT_TYPE     "GlobalStandard"
azd env set AZURE_CHAT_GPT_DEPLOYMENT_CAPACITY "40"

# Modelo de embeddings
azd env set AZURE_EMBEDDINGS_MODEL_NAME        "text-embedding-3-large"
azd env set AZURE_EMBEDDINGS_VERSION           "1"
azd env set AZURE_EMBEDDINGS_DEPLOYMENT_NAME   "text-embedding"
azd env set AZURE_EMBEDDINGS_VECTOR_SIZE       "3072"

# Versión de API de OpenAI
azd env set AZURE_OPENAI_API_VERSION "2024-10-21"
```

#### 3.10 — Search, Idioma y Configuración RAG

```sh
azd env set AZURE_RETRIEVAL_APPROACH           "hybrid"
azd env set AZURE_SEARCH_ANALYZER_NAME         "es.microsoft"
azd env set AZURE_USE_SEMANTIC_RERANKING       "true"
azd env set AZURE_SEARCH_INDEX                 "ragindex"
azd env set AZURE_SEARCH_INDEX_INTERVAL        "PT1H"
azd env set AZURE_ORCHESTRATOR_MESSAGES_LANGUAGE "es"

# Configuración de voz (español Colombia)
azd env set AZURE_SPEECH_RECOGNITION_LANGUAGE  "es-CO"
azd env set AZURE_SPEECH_SYNTHESIS_LANGUAGE    "es-CO"
azd env set AZURE_SPEECH_SYNTHESIS_VOICE_NAME  "es-CO-SalomeNeural"
```

---

### Fase 4 — Alternativa: Archivo `.env` Directo

En lugar de ejecutar todos los comandos `azd env set` uno por uno, puedes pegar directamente el contenido del archivo `.env` en `.azure/asimed-guicli-poc/.env`. Esta es la forma más rápida.

1. Inicializa el entorno primero (si no lo hiciste):
   ```sh
   azd env new asimed-guicli-poc
   ```

2. Abre o crea el archivo `.azure/asimed-guicli-poc/.env` y pega el siguiente contenido:

```ini
AZURE_ENV_NAME="asimed-guicli-poc"
AZURE_LOCATION="eastus2"
AZURE_RESOURCE_GROUP_NAME="<nombre-rg-cliente>"
AZURE_SUBSCRIPTION_ID="<subscription-id>"
AZURE_NETWORK_ISOLATION="true"

# Compute / App
AZURE_DATA_INGEST_FUNC_NAME="func-asimed-guicli-ingest-poc-s"
AZURE_ORCHESTRATOR_FUNCTION_APP_NAME="func-asimed-guicli-orques-poc-s"
AZURE_APP_SERVICE_NAME="app-serv-asimed-guicli-poc-s"
AZURE_APP_SERVICE_PLAN_NAME="asp-asimed-guicli-manage-poc-s"

# Data / Security
AZURE_DB_ACCOUNT_NAME="cosdb-asimed-guicli-manage-poc-s"
AZURE_STORAGE_ACCOUNT_NAME="stasimedguiclipocs"
AZURE_KEY_VAULT_NAME="kv-asimed-guicli-poc-s"

# AI / Search
AZURE_AI_SERVICES_NAME="aif-asimed-guicli-poc-s"
AZURE_OPENAI_SERVICE_NAME="aif-asimed-guicli-poc-s"
AZURE_SEARCH_SERVICE_NAME="ais-asimed-guicli-poc-s"

# Monitoring
AZURE_APP_INSIGHTS_NAME="appi-asimed-guicli-manage-poc-s"

# Networking
AZURE_VNET_NAME="vnet-asimed-guicli-main-poc-s"
AZURE_VNET_ADDRESS="10.0.0.0/23"
AZURE_AI_SUBNET_NAME="snet-asimed-guicli-ai-poc-s"
AZURE_AI_SUBNET_PREFIX="10.0.0.0/26"
AZURE_BASTION_SUBNET_PREFIX="10.0.0.64/26"
AZURE_APP_INT_SUBNET_NAME="snet-asimed-guicli-appint-poc-s"
AZURE_APP_INT_SUBNET_PREFIX="10.0.0.128/26"
AZURE_APP_SERVICES_SUBNET_NAME="snet-asimed-guicli-app-poc-s"
AZURE_APP_SERVICES_SUBNET_PREFIX="10.0.0.192/26"
AZURE_DATABASE_SUBNET_NAME="snet-asimed-guicli-db-poc-s"
AZURE_DATABASE_SUBNET_PREFIX="10.0.1.0/26"

# Private Endpoints
AZURE_STORAGE_ACCOUNT_PE="pe-storage-asimed-guicli-manage-poc-s"
AZURE_DB_ACCOUNT_PE="pe-cosmos-asimed-guicli-manage-poc-s"
AZURE_KEYVAULT_PE="pe-kv-asimed-guicli-manage-poc-s"
AZURE_ORCHESTRATOR_PE="pe-orc-asimed-guicli-manage-poc-s"
AZURE_FRONTEND_PE="pe-frontend-asimed-guicli-manage-poc-s"
AZURE_DATA_INGESTION_PE="pe-ingest-asimed-guicli-manage-poc-s"
AZURE_AI_SERVICES_PE="pe-ai-asimed-guicli-manage-poc-s"
AZURE_OPEN_AI_PE="pe-oai-asimed-guicli-manage-poc-s"
AZURE_SEARCH_PE="pe-search-asimed-guicli-manage-poc-s"

# VM / Bastion
AZURE_VM_DEPLOY_VM="true"
AZURE_VM_NAME="vm-asimed-guicli-poc-s"
AZURE_VM_USER_NAME="gptrag"
AZURE_BASTION_KV_NAME="kv-bastion-asimed-poc-s"
AZURE_VM_KV_SEC_NAME="vmUserInitialPassword"

# Modelos AI
AZURE_CHAT_GPT_MODEL_NAME="gpt-4o"
AZURE_CHAT_GPT_MODEL_VERSION="2024-11-20"
AZURE_CHAT_GPT_DEPLOYMENT_NAME="chat"
AZURE_CHAT_GPT_DEPLOYMENT_TYPE="GlobalStandard"
AZURE_CHAT_GPT_DEPLOYMENT_CAPACITY="40"
AZURE_EMBEDDINGS_MODEL_NAME="text-embedding-3-large"
AZURE_EMBEDDINGS_VERSION="1"
AZURE_EMBEDDINGS_DEPLOYMENT_NAME="text-embedding"
AZURE_EMBEDDINGS_VECTOR_SIZE="3072"
AZURE_OPENAI_API_VERSION="2024-10-21"

# Search / RAG / Idioma
AZURE_RETRIEVAL_APPROACH="hybrid"
AZURE_SEARCH_ANALYZER_NAME="es.microsoft"
AZURE_USE_SEMANTIC_RERANKING="true"
AZURE_SEARCH_INDEX="ragindex"
AZURE_SEARCH_INDEX_INTERVAL="PT1H"
AZURE_ORCHESTRATOR_MESSAGES_LANGUAGE="es"
AZURE_SPEECH_RECOGNITION_LANGUAGE="es-CO"
AZURE_SPEECH_SYNTHESIS_LANGUAGE="es-CO"
AZURE_SPEECH_SYNTHESIS_VOICE_NAME="es-CO-SalomeNeural"
```

> ⚠️ **Reemplaza** `<nombre-rg-cliente>` y `<subscription-id>` con los valores reales antes de continuar.

---

### Fase 5 — Log Analytics (Pre-creación Manual)

El Log Analytics Workspace **debe crearse manualmente** antes de ejecutar `azd provision`, ya que el cliente ya tiene el nombre definido (`LogA-AsistenteMedico-poc-s`) y se reutiliza el recurso existente.

#### 5.1 — Crear el workspace (si no existe)

```sh
# Crear Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group "<nombre-rg-cliente>" \
  --workspace-name "LogA-AsistenteMedico-poc-s" \
  --location "eastus2" \
  --sku "PerGB2018"
```

#### 5.2 — Obtener el Resource ID del workspace

```sh
# Obtener el Resource ID completo del workspace
az monitor log-analytics workspace show \
  --resource-group "<nombre-rg-cliente>" \
  --workspace-name "LogA-AsistenteMedico-poc-s" \
  --query id \
  --output tsv
```

El comando anterior devuelve algo como:
```
/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/LogA-AsistenteMedico-poc-s
```

#### 5.3 — Configurar reutilización del workspace en azd

```sh
# Indicar a Bicep que reutilice el workspace existente
azd env set LOG_ANALYTICS_WORKSPACE_REUSE "true"
azd env set LOG_ANALYTICS_WORKSPACE_ID "/subscriptions/<sub-id>/resourceGroups/<nombre-rg-cliente>/providers/Microsoft.OperationalInsights/workspaces/LogA-AsistenteMedico-poc-s"

# Reutilizar también App Insights si ya existe
azd env set APP_INSIGHTS_REUSE "true"
azd env set APP_INSIGHTS_RESOURCE_GROUP_NAME "<nombre-rg-cliente>"
azd env set APP_INSIGHTS_NAME "appi-asimed-guicli-manage-poc-s"
```

---

### Fase 6 — Provisionar Infraestructura

```sh
azd provision
```

#### ¿Qué ocurre internamente?

```
azd provision
  │
  ├── [hook] preprovision.sh/.ps1
  │     └── Muestra advertencia si AZURE_NETWORK_ISOLATION=true
  │         (recordatorio de que el deploy posterior requiere acceso vía VM+Bastion)
  │
  ├── [Bicep deployment] infra/main.bicep
  │     ├── Crea o usa el Resource Group del cliente
  │     ├── Despliega VNet con 5 subnets
  │     ├── Despliega Azure AI Services (OpenAI + Doc Intelligence)
  │     ├── Despliega Azure AI Search
  │     ├── Despliega CosmosDB
  │     ├── Despliega Key Vault + secretos
  │     ├── Despliega Storage Account (docs)
  │     ├── Despliega Storage Accounts adicionales (para Functions: orc/ingest)
  │     ├── Despliega App Service Plan + App Service (frontend)
  │     ├── Despliega Azure Functions (orchestrator + ingestion)
  │     ├── Despliega App Insights (o reutiliza existente)
  │     ├── Despliega VM + Azure Bastion (si AZURE_VM_DEPLOY_VM=true)
  │     ├── Crea Private Endpoints para todos los servicios
  │     ├── Crea DNS Zones privadas y los vincula a la VNet
  │     └── Asigna roles RBAC (identidades administradas de cada servicio)
  │
  └── [hook] postprovision.sh/.ps1
        └── Configuraciones adicionales post-infraestructura
```

> ⏱️ El aprovisionamiento completo tarda entre **25 y 45 minutos** dependiendo de la región.

#### Monitorear el progreso

```sh
# Ver el estado del deployment en Azure Portal
az deployment sub show \
  --name "<deployment-name>" \
  --query "properties.provisioningState"

# Ver todos los recursos creados en el RG
az resource list \
  --resource-group "<nombre-rg-cliente>" \
  --output table
```

---

### Fase 7 — Despliegue de Componentes

El despliegue del código (Functions + App Service) se realiza con `azd deploy`. Hay dos variantes según la arquitectura:

#### Opción A — Zero Trust (con VM + Bastion) ⭐ Recomendado para producción

En arquitectura Zero Trust, los endpoints de las Functions y App Service **no son accesibles desde internet**. El deploy debe ejecutarse **desde dentro de la VNet**, es decir, desde la VM creada en la Fase 6.

**Pasos:**

1. **Conectar a la VM via Azure Bastion**:
   ```sh
   # Desde Azure Portal: ir a la VM → Connect → Bastion
   # O desde Azure CLI:
   az network bastion ssh \
     --name "AzureBastion" \
     --resource-group "<nombre-rg-cliente>" \
     --target-resource-id "/subscriptions/<sub-id>/resourceGroups/<nombre-rg-cliente>/providers/Microsoft.Compute/virtualMachines/vm-asimed-guicli-poc-s" \
     --auth-type "password" \
     --username "gptrag"
   ```

2. **Dentro de la VM**, instalar herramientas si no están:
   ```sh
   # Azure Developer CLI
   curl -fsSL https://aka.ms/install-azd.sh | bash

   # Azure CLI
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

3. **Clonar el repositorio** dentro de la VM:
   ```sh
   git clone https://github.com/jmaya30/poc_rag_gpt.git
   cd poc_rag_gpt
   ```

4. **Autenticar** dentro de la VM:
   ```sh
   azd auth login --use-device-code
   az login --use-device-code
   ```

5. **Copiar el archivo `.env`** con todas las variables configuradas a `.azure/asimed-guicli-poc/.env`

6. **Ejecutar el deploy**:
   ```sh
   azd deploy
   ```

#### Opción B — Despliegue Básico (sin aislamiento de red)

Si `AZURE_NETWORK_ISOLATION` es `false`, el deploy puede ejecutarse directamente desde tu máquina local:

```sh
azd deploy
```

---

### Fase 8 — Subir Documentos de Prueba

Una vez desplegada la infraestructura, sube documentos al Storage Account para probar el pipeline de ingesta.

```sh
# Subir un documento de prueba al contenedor de documentos
az storage blob upload \
  --account-name "stasimedguiclipocs" \
  --container-name "documents" \
  --name "documento-prueba.pdf" \
  --file "./ruta/local/documento-prueba.pdf" \
  --auth-mode login

# Listar blobs para verificar
az storage blob list \
  --account-name "stasimedguiclipocs" \
  --container-name "documents" \
  --auth-mode login \
  --output table
```

La función de ingesta (`func-asimed-guicli-ingest-poc-s`) se activará automáticamente con un trigger de Storage y procesará el documento, indexándolo en Azure AI Search.

---

### Fase 9 — Verificación Post-Despliegue

```sh
# 1. Verificar que todos los recursos fueron creados
az resource list \
  --resource-group "<nombre-rg-cliente>" \
  --output table

# 2. Verificar el estado de las Functions
az functionapp show \
  --name "func-asimed-guicli-ingest-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "state" --output tsv

az functionapp show \
  --name "func-asimed-guicli-orques-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "state" --output tsv

# 3. Verificar el App Service
az webapp show \
  --name "app-serv-asimed-guicli-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "state" --output tsv

# 4. Verificar el índice de AI Search
az search index list \
  --service-name "ais-asimed-guicli-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --output table

# 5. Ver todas las variables del entorno azd
azd env get-values

# 6. Ver los endpoints desplegados
azd show
```

---

## 5. Notas y Consideraciones Importantes

### 🔒 AzureBastionSubnet — Nombre Forzado por Azure

Azure **exige** que la subnet del servicio Bastion se llame exactamente `AzureBastionSubnet`. No existe forma de personalizar este nombre. El template Bicep incluye esta restricción internamente. Solo puedes controlar el CIDR (`AZURE_BASTION_SUBNET_PREFIX`).

---

### 📦 Storage Account — Restricciones de Nombre

El nombre del Storage Account (`stasimedguiclipocs`) debe cumplir:
- **Solo minúsculas y números** (sin guiones ni caracteres especiales)
- **3 a 24 caracteres**
- **Globalmente único en Azure**

El nombre `stasimedguiclipocs` cumple todas estas restricciones. ✅

---

### 🔑 Key Vault — Restricciones de Nombre

El nombre del Key Vault debe cumplir:
- **3 a 24 caracteres**
- Solo letras, números y guiones
- Empezar con letra, terminar con letra o número
- **Globalmente único en Azure** (el nombre queda "reservado" 90 días después de eliminar)

El nombre `kv-asimed-guicli-poc-s` cumple con 22 caracteres. ✅

---

### 💾 Storage Accounts Adicionales para Functions

El template Bicep crea automáticamente **Storage Accounts adicionales** para las Azure Functions con sufijos:
- `orc` → Para la Function del Orchestrator
- `ingest` → Para la Function de Ingestion

Estos nombres se generan automáticamente basados en el `environmentName` y `resourceToken`. No necesitas configurarlos manualmente.

---

### 🗂️ Resource Group Pre-existente

El template Bicep maneja el Resource Group con una declaración `resource resourceGroup ... existing`. Si el RG ya existe, Bicep lo referencia sin intentar recrearlo (**idempotente**). Si no existe, lo crea. En ambos casos, el comportamiento es correcto.

---

### 🤖 Document Intelligence dentro de AI Services

El servicio de Document Intelligence **no se despliega como recurso separado** en esta versión. Se incluye como parte del recurso **Azure AI Services multi-service** (`aif-asimed-guicli-poc-s`), que provee acceso a todos los servicios cognitivos de Azure (incluyendo Document Intelligence/Form Recognizer) a través de un único endpoint y key.

---

### 🌍 Disponibilidad Regional de GPT-4o

Antes de desplegar, verifica que la región `eastus2` tenga cuota disponible para:
- `gpt-4o` (versión `2024-11-20`) con capacidad `40K TPM`
- `text-embedding-3-large` con capacidad suficiente

```sh
# Verificar disponibilidad de modelos en la región
az cognitiveservices model list \
  --location eastus2 \
  --query "[?name=='gpt-4o'].{name:name, version:version, capacity:maxCapacity}" \
  --output table
```

---

## 6. Troubleshooting

### ❌ Error: `InvalidTemplateDeployment` — Resource Group no encontrado

**Síntoma**: `azd provision` falla porque no puede encontrar el Resource Group.

**Solución**:
```sh
# Verificar que el RG existe
az group show --name "<nombre-rg-cliente>"

# Si no existe, crearlo
az group create \
  --name "<nombre-rg-cliente>" \
  --location "eastus2"
```

---

### ❌ Error: `StorageAccountAlreadyTaken` — Nombre de Storage en uso

**Síntoma**: El nombre `stasimedguiclipocs` ya está en uso por otra cuenta.

**Solución**: El nombre de Storage Account es globalmente único en Azure. Coordina con el cliente para usar un nombre diferente o verifica si el recurso ya existe en la suscripción del cliente.

```sh
# Verificar si el nombre está disponible
az storage account check-name --name "stasimedguiclipocs"
```

---

### ❌ Error: `QuotaExceeded` — Sin cuota para modelos OpenAI

**Síntoma**: El deployment de modelos GPT-4o o text-embedding-3-large falla por cuota.

**Solución**:
1. Revisar la cuota actual: Azure Portal → Azure OpenAI → Quotas
2. Solicitar aumento de cuota en: https://aka.ms/oai/quotaincrease
3. Reducir `AZURE_CHAT_GPT_DEPLOYMENT_CAPACITY` si la cuota actual lo requiere

---

### ❌ Error: `VnetAlreadyExists` durante re-despliegue

**Síntoma**: Al re-ejecutar `azd provision`, falla por conflictos en la VNet.

**Solución**: Habilitar la reutilización de la VNet existente:
```sh
azd env set VNET_REUSE "true"
azd env set VNET_RESOURCE_GROUP_NAME "<nombre-rg-cliente>"
azd env set VNET_NAME "vnet-asimed-guicli-main-poc-s"
```

---

### ❌ Deploy falla desde máquina local con `AZURE_NETWORK_ISOLATION=true`

**Síntoma**: `azd deploy` falla con errores de conexión rechazada (403/connection refused).

**Causa**: Los Private Endpoints bloquean el acceso desde internet. El deploy debe ejecutarse desde dentro de la VNet.

**Solución**: Seguir el proceso descrito en la [Fase 7 — Opción A (Zero Trust)](#opción-a--zero-trust-con-vm--bastion-️-recomendado-para-producción).

---

### 🔄 Reintentar un aprovisionamiento fallido

Si `azd provision` falla a mitad del proceso:

```sh
# Verificar el estado del último deployment en Azure
az deployment sub list \
  --query "[0].{name:name, state:properties.provisioningState}" \
  --output table

# Reintentar (Bicep es idempotente)
azd provision
```

---

### 📋 Ver logs de las Functions

```sh
# Logs en tiempo real de la Function de Ingestion
az functionapp log tail \
  --name "func-asimed-guicli-ingest-poc-s" \
  --resource-group "<nombre-rg-cliente>"

# Logs del Orchestrator
az functionapp log tail \
  --name "func-asimed-guicli-orques-poc-s" \
  --resource-group "<nombre-rg-cliente>"
```
