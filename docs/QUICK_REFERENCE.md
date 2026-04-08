# ⚡ Referencia Rápida — POC RAG GPT

Cheat sheet con todos los comandos listos para copiar y pegar. Solo reemplaza `<nombre-rg-cliente>` y `<subscription-id>` con los valores reales.

---

## 📑 Tabla de Contenidos

1. [Script Bash: Todos los `azd env set`](#1-script-bash-todos-los-azd-env-set)
2. [Archivo `.env` Completo](#2-archivo-env-completo)
3. [Comandos de Verificación Post-Despliegue](#3-comandos-de-verificación-post-despliegue)
4. [Comandos Útiles de Troubleshooting](#4-comandos-útiles-de-troubleshooting)
5. [Flujo Completo Resumido](#5-flujo-completo-resumido)

---

## 1. Script Bash: Todos los `azd env set`

Copia y pega este bloque completo en tu terminal. Solo edita las líneas marcadas con `# ← EDITAR`.

```sh
#!/bin/bash
# ============================================================
# POC RAG GPT — Configuración de variables azd
# Cliente: ASIMED / GuiCli
# ============================================================

# ---- BASE ---- (← EDITAR estas dos líneas)
azd env set AZURE_RESOURCE_GROUP_NAME  "<nombre-rg-cliente>"   # ← EDITAR
azd env set AZURE_SUBSCRIPTION_ID      "<subscription-id>"     # ← EDITAR

azd env set AZURE_ENV_NAME             "asimed-guicli-poc"
azd env set AZURE_LOCATION             "eastus2"
azd env set AZURE_NETWORK_ISOLATION    "true"

# ---- COMPUTE / APP ----
azd env set AZURE_DATA_INGEST_FUNC_NAME          "func-asimed-guicli-ingest-poc-s"
azd env set AZURE_ORCHESTRATOR_FUNCTION_APP_NAME "func-asimed-guicli-orques-poc-s"
azd env set AZURE_APP_SERVICE_NAME               "app-serv-asimed-guicli-poc-s"
azd env set AZURE_APP_SERVICE_PLAN_NAME          "asp-asimed-guicli-manage-poc-s"

# ---- DATA / SECURITY ----
azd env set AZURE_DB_ACCOUNT_NAME       "cosdb-asimed-guicli-manage-poc-s"
azd env set AZURE_STORAGE_ACCOUNT_NAME "stasimedguiclipocs"
azd env set AZURE_KEY_VAULT_NAME       "kv-asimed-guicli-poc-s"

# ---- AI / SEARCH ----
azd env set AZURE_AI_SERVICES_NAME    "aif-asimed-guicli-poc-s"
azd env set AZURE_OPENAI_SERVICE_NAME "aif-asimed-guicli-poc-s"
azd env set AZURE_SEARCH_SERVICE_NAME "ais-asimed-guicli-poc-s"

# ---- MONITORING ----
azd env set AZURE_APP_INSIGHTS_NAME "appi-asimed-guicli-manage-poc-s"

# ---- NETWORKING ----
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

# ---- PRIVATE ENDPOINTS ----
azd env set AZURE_STORAGE_ACCOUNT_PE  "pe-storage-asimed-guicli-manage-poc-s"
azd env set AZURE_DB_ACCOUNT_PE       "pe-cosmos-asimed-guicli-manage-poc-s"
azd env set AZURE_KEYVAULT_PE         "pe-kv-asimed-guicli-manage-poc-s"
azd env set AZURE_ORCHESTRATOR_PE     "pe-orc-asimed-guicli-manage-poc-s"
azd env set AZURE_FRONTEND_PE         "pe-frontend-asimed-guicli-manage-poc-s"
azd env set AZURE_DATA_INGESTION_PE   "pe-ingest-asimed-guicli-manage-poc-s"
azd env set AZURE_AI_SERVICES_PE      "pe-ai-asimed-guicli-manage-poc-s"
azd env set AZURE_OPEN_AI_PE          "pe-oai-asimed-guicli-manage-poc-s"
azd env set AZURE_SEARCH_PE           "pe-search-asimed-guicli-manage-poc-s"

# ---- VM / BASTION ----
azd env set AZURE_VM_DEPLOY_VM    "true"
azd env set AZURE_VM_NAME         "vm-asimed-guicli-poc-s"
azd env set AZURE_VM_USER_NAME    "gptrag"
azd env set AZURE_BASTION_KV_NAME "kv-bastion-asimed-poc-s"
azd env set AZURE_VM_KV_SEC_NAME  "vmUserInitialPassword"

# ---- MODELOS AI ----
azd env set AZURE_CHAT_GPT_MODEL_NAME          "gpt-4o"
azd env set AZURE_CHAT_GPT_MODEL_VERSION       "2024-11-20"
azd env set AZURE_CHAT_GPT_DEPLOYMENT_NAME     "chat"
azd env set AZURE_CHAT_GPT_DEPLOYMENT_TYPE     "GlobalStandard"
azd env set AZURE_CHAT_GPT_DEPLOYMENT_CAPACITY "40"
azd env set AZURE_EMBEDDINGS_MODEL_NAME        "text-embedding-3-large"
azd env set AZURE_EMBEDDINGS_VERSION           "1"
azd env set AZURE_EMBEDDINGS_DEPLOYMENT_NAME   "text-embedding"
azd env set AZURE_EMBEDDINGS_VECTOR_SIZE       "3072"
azd env set AZURE_OPENAI_API_VERSION           "2024-10-21"

# ---- SEARCH / RAG / IDIOMA ----
azd env set AZURE_RETRIEVAL_APPROACH             "hybrid"
azd env set AZURE_SEARCH_ANALYZER_NAME           "es.microsoft"
azd env set AZURE_USE_SEMANTIC_RERANKING         "true"
azd env set AZURE_SEARCH_INDEX                   "ragindex"
azd env set AZURE_SEARCH_INDEX_INTERVAL          "PT1H"
azd env set AZURE_ORCHESTRATOR_MESSAGES_LANGUAGE "es"
azd env set AZURE_SPEECH_RECOGNITION_LANGUAGE    "es-CO"
azd env set AZURE_SPEECH_SYNTHESIS_LANGUAGE      "es-CO"
azd env set AZURE_SPEECH_SYNTHESIS_VOICE_NAME    "es-CO-SalomeNeural"

# ---- LOG ANALYTICS (si ya existe el workspace) ----
# azd env set LOG_ANALYTICS_WORKSPACE_REUSE "true"
# azd env set LOG_ANALYTICS_WORKSPACE_ID    "/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/LogA-AsistenteMedico-poc-s"

echo "✅ Variables configuradas correctamente."
echo "   Siguiente paso: azd provision"
```

---

## 2. Archivo `.env` Completo

Pega este contenido directamente en `.azure/asimed-guicli-poc/.env`:

```ini
AZURE_ENV_NAME="asimed-guicli-poc"
AZURE_LOCATION="eastus2"
AZURE_RESOURCE_GROUP_NAME="<nombre-rg-cliente>"
AZURE_SUBSCRIPTION_ID="<subscription-id>"
AZURE_NETWORK_ISOLATION="true"
AZURE_DATA_INGEST_FUNC_NAME="func-asimed-guicli-ingest-poc-s"
AZURE_ORCHESTRATOR_FUNCTION_APP_NAME="func-asimed-guicli-orques-poc-s"
AZURE_APP_SERVICE_NAME="app-serv-asimed-guicli-poc-s"
AZURE_APP_SERVICE_PLAN_NAME="asp-asimed-guicli-manage-poc-s"
AZURE_DB_ACCOUNT_NAME="cosdb-asimed-guicli-manage-poc-s"
AZURE_STORAGE_ACCOUNT_NAME="stasimedguiclipocs"
AZURE_AI_SERVICES_NAME="aif-asimed-guicli-poc-s"
AZURE_OPENAI_SERVICE_NAME="aif-asimed-guicli-poc-s"
AZURE_SEARCH_SERVICE_NAME="ais-asimed-guicli-poc-s"
AZURE_KEY_VAULT_NAME="kv-asimed-guicli-poc-s"
AZURE_APP_INSIGHTS_NAME="appi-asimed-guicli-manage-poc-s"
AZURE_VNET_NAME="vnet-asimed-guicli-main-poc-s"
AZURE_VNET_ADDRESS="10.0.0.0/23"
AZURE_AI_SUBNET_NAME="snet-asimed-guicli-ai-poc-s"
AZURE_AI_SUBNET_PREFIX="10.0.0.0/26"
AZURE_APP_INT_SUBNET_NAME="snet-asimed-guicli-appint-poc-s"
AZURE_APP_INT_SUBNET_PREFIX="10.0.0.128/26"
AZURE_APP_SERVICES_SUBNET_NAME="snet-asimed-guicli-app-poc-s"
AZURE_APP_SERVICES_SUBNET_PREFIX="10.0.0.192/26"
AZURE_DATABASE_SUBNET_NAME="snet-asimed-guicli-db-poc-s"
AZURE_DATABASE_SUBNET_PREFIX="10.0.1.0/26"
AZURE_BASTION_SUBNET_PREFIX="10.0.0.64/26"
AZURE_STORAGE_ACCOUNT_PE="pe-storage-asimed-guicli-manage-poc-s"
AZURE_DB_ACCOUNT_PE="pe-cosmos-asimed-guicli-manage-poc-s"
AZURE_KEYVAULT_PE="pe-kv-asimed-guicli-manage-poc-s"
AZURE_ORCHESTRATOR_PE="pe-orc-asimed-guicli-manage-poc-s"
AZURE_FRONTEND_PE="pe-frontend-asimed-guicli-manage-poc-s"
AZURE_DATA_INGESTION_PE="pe-ingest-asimed-guicli-manage-poc-s"
AZURE_AI_SERVICES_PE="pe-ai-asimed-guicli-manage-poc-s"
AZURE_OPEN_AI_PE="pe-oai-asimed-guicli-manage-poc-s"
AZURE_SEARCH_PE="pe-search-asimed-guicli-manage-poc-s"
AZURE_VM_DEPLOY_VM="true"
AZURE_VM_NAME="vm-asimed-guicli-poc-s"
AZURE_VM_USER_NAME="gptrag"
AZURE_BASTION_KV_NAME="kv-bastion-asimed-poc-s"
AZURE_VM_KV_SEC_NAME="vmUserInitialPassword"
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

---

## 3. Comandos de Verificación Post-Despliegue

```sh
# ── Variables azd actuales ──────────────────────────────────────────
azd env get-values

# ── Recursos en el RG ──────────────────────────────────────────────
az resource list \
  --resource-group "<nombre-rg-cliente>" \
  --output table

# ── Estado de Functions ────────────────────────────────────────────
az functionapp show \
  --name "func-asimed-guicli-ingest-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "{name:name, state:state, url:defaultHostName}" \
  --output table

az functionapp show \
  --name "func-asimed-guicli-orques-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "{name:name, state:state, url:defaultHostName}" \
  --output table

# ── Estado del App Service ─────────────────────────────────────────
az webapp show \
  --name "app-serv-asimed-guicli-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "{name:name, state:state, url:defaultHostName}" \
  --output table

# ── Verificar índice de AI Search ──────────────────────────────────
az search index list \
  --service-name "ais-asimed-guicli-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --output table

# ── Verificar deployments de OpenAI ────────────────────────────────
az cognitiveservices account deployment list \
  --name "aif-asimed-guicli-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --output table

# ── Verificar Private Endpoints ────────────────────────────────────
az network private-endpoint list \
  --resource-group "<nombre-rg-cliente>" \
  --query "[].{name:name, state:provisioningState}" \
  --output table

# ── Verificar VNet y subnets ───────────────────────────────────────
az network vnet show \
  --name "vnet-asimed-guicli-main-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "{name:name, addressSpace:addressSpace.addressPrefixes, subnets:subnets[].{name:name,prefix:addressPrefix}}" \
  --output json

# ── Verificar CosmosDB ─────────────────────────────────────────────
az cosmosdb show \
  --name "cosdb-asimed-guicli-manage-poc-s" \
  --resource-group "<nombre-rg-cliente>" \
  --query "{name:name, state:documentEndpoint}" \
  --output table

# ── Ver endpoints de azd ───────────────────────────────────────────
azd show
```

---

## 4. Comandos Útiles de Troubleshooting

```sh
# ── Ver logs de deployment de Bicep ────────────────────────────────
az deployment sub list \
  --query "[0:5].{name:name, state:properties.provisioningState, timestamp:properties.timestamp}" \
  --output table

# ── Inspeccionar errores de un deployment ──────────────────────────
az deployment sub show \
  --name "<deployment-name>" \
  --query "properties.error" \
  --output json

# ── Logs en tiempo real de la Function Ingestion ───────────────────
az functionapp log tail \
  --name "func-asimed-guicli-ingest-poc-s" \
  --resource-group "<nombre-rg-cliente>"

# ── Logs en tiempo real del Orchestrator ───────────────────────────
az functionapp log tail \
  --name "func-asimed-guicli-orques-poc-s" \
  --resource-group "<nombre-rg-cliente>"

# ── Reiniciar una Function App ─────────────────────────────────────
az functionapp restart \
  --name "func-asimed-guicli-orques-poc-s" \
  --resource-group "<nombre-rg-cliente>"

# ── Reiniciar el App Service ───────────────────────────────────────
az webapp restart \
  --name "app-serv-asimed-guicli-poc-s" \
  --resource-group "<nombre-rg-cliente>"

# ── Verificar disponibilidad del nombre de Storage ─────────────────
az storage account check-name --name "stasimedguiclipocs"

# ── Verificar cuota de OpenAI en la región ─────────────────────────
az cognitiveservices usage list \
  --location "eastus2" \
  --query "[?contains(name.value, 'gpt-4o')]" \
  --output table

# ── Ver secretos del Key Vault ─────────────────────────────────────
az keyvault secret list \
  --vault-name "kv-asimed-guicli-poc-s" \
  --output table

# ── Forzar re-trigger de ingesta (subir doc al storage) ────────────
az storage blob upload \
  --account-name "stasimedguiclipocs" \
  --container-name "documents" \
  --name "test.pdf" \
  --file "/ruta/local/test.pdf" \
  --auth-mode login

# ── Ver blobs en el storage ────────────────────────────────────────
az storage blob list \
  --account-name "stasimedguiclipocs" \
  --container-name "documents" \
  --auth-mode login \
  --output table

# ── Conectar a VM via Bastion (CLI) ────────────────────────────────
az network bastion ssh \
  --name "AzureBastion" \
  --resource-group "<nombre-rg-cliente>" \
  --target-resource-id "/subscriptions/<sub-id>/resourceGroups/<nombre-rg-cliente>/providers/Microsoft.Compute/virtualMachines/vm-asimed-guicli-poc-s" \
  --auth-type "password" \
  --username "gptrag"
```

---

## 5. Flujo Completo Resumido

```sh
# 1. Clonar e inicializar
azd init -t azure/gpt-rag -b release/1.0.1
# → Nombre del entorno: asimed-guicli-poc

# 2. Autenticar
azd auth login
az login
az account set --subscription "<subscription-id>"

# 3. Configurar variables
#    OPCIÓN A: script bash de la Sección 1 arriba
#    OPCIÓN B: pegar .env en .azure/asimed-guicli-poc/.env

# 4. Pre-crear Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group "<nombre-rg-cliente>" \
  --workspace-name "LogA-AsistenteMedico-poc-s" \
  --location "eastus2"

# 5. Configurar reutilización de Log Analytics
LOG_ID=$(az monitor log-analytics workspace show \
  --resource-group "<nombre-rg-cliente>" \
  --workspace-name "LogA-AsistenteMedico-poc-s" \
  --query id --output tsv)

azd env set LOG_ANALYTICS_WORKSPACE_REUSE "true"
azd env set LOG_ANALYTICS_WORKSPACE_ID    "$LOG_ID"

# 6. Provisionar (~25-45 minutos)
azd provision

# 7. Desplegar código
#    Zero Trust → conectar a VM Bastion primero, luego:
azd deploy

# 8. Verificar
az resource list --resource-group "<nombre-rg-cliente>" --output table
azd show
```

---

> 📖 Para instrucciones detalladas consulta [SETUP_GUIDE.md](SETUP_GUIDE.md)
