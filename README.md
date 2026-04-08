# POC RAG GPT - Infraestructura Azure

> Infraestructura como Código (IaC) en Bicep para desplegar una solución GPT-RAG empresarial con Azure OpenAI.

Este repositorio contiene la carpeta `infra/` con toda la infraestructura necesaria para desplegar el patrón **Retrieval-Augmented Generation (RAG)** sobre Azure, derivada del proyecto [Azure/GPT-RAG v1.0.1](https://github.com/Azure/GPT-RAG/tree/release/1.0.1). La infraestructura está definida en **Bicep** y se despliega con **Azure Developer CLI (`azd`)**.

---

## 🏛️ Arquitectura

La solución despliega los siguientes componentes en Azure:

| Componente | Servicio Azure | Descripción |
|---|---|---|
| 🔄 **Ingestion** | Azure Functions | Procesa e indexa documentos en Azure AI Search |
| 🎯 **Orchestrator** | Azure Functions | Orquesta el flujo RAG: búsqueda + generación |
| 🖥️ **Frontend** | App Service | Interfaz React + backend Python (FastAPI) |
| 🔍 **AI Search** | Azure AI Search | Motor de búsqueda vectorial/híbrida |
| 🤖 **OpenAI** | Azure OpenAI | Modelos GPT-4o (chat) + text-embedding-3-large |
| 📄 **Doc Intelligence** | Azure AI Services | Extracción de contenido de documentos (multi-service) |
| 💾 **CosmosDB** | Azure Cosmos DB | Historial de conversaciones |
| 🔐 **Key Vault** | Azure Key Vault | Gestión de secretos y credenciales |
| 📦 **Storage** | Azure Blob Storage | Almacenamiento de documentos fuente |
| 📊 **Monitoring** | App Insights + Log Analytics | Observabilidad y trazas distribuidas |
| 🌐 **Red** | VNet + Subnets + Private Endpoints | Arquitectura Zero Trust con aislamiento de red |
| 🖥️ **Acceso seguro** | VM + Azure Bastion | Acceso a entornos con aislamiento de red |

---

## 📂 Estructura de Archivos

```
infra/
├── main.bicep                  # Plantilla principal de infraestructura
├── main.parameters.json        # Parámetros mapeados a variables de entorno azd
└── core/                       # Módulos Bicep reutilizables
    ├── ai/                     # Azure OpenAI y AI Services
    ├── db/                     # CosmosDB
    ├── host/                   # App Service, Functions, App Insights
    ├── loadtesting/            # Azure Load Testing
    ├── network/                # VNet, subnets, Private Endpoints, DNS privado
    ├── search/                 # Azure AI Search
    ├── security/               # Key Vault, RBAC
    ├── storage/                # Blob Storage
    ├── util/                   # Utilidades (delays, etc.)
    └── vm/                     # VM + Bastion (Zero Trust)
```

---

## ✅ Pre-requisitos

| Herramienta | Versión mínima | Verificación |
|---|---|---|
| [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd) | `>= 1.5.0` | `azd version` |
| [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) | Latest | `az version` |
| [Git](https://git-scm.com/) | Latest | `git --version` |
| [Python](https://www.python.org/) | `3.11` | `python --version` |
| [Node.js](https://nodejs.org/) | `>= 16` | `node --version` |
| PowerShell | `>= 7` (Windows) | `pwsh --version` |

---

## 🚀 Despliegue

Consulta la guía completa de aprovisionamiento en:

📖 **[docs/SETUP_GUIDE.md](docs/SETUP_GUIDE.md)** — Instrucciones detalladas paso a paso, incluyendo nomenclatura del cliente, configuración de red y variables de entorno.

Para una referencia rápida tipo cheat-sheet:

⚡ **[docs/QUICK_REFERENCE.md](docs/QUICK_REFERENCE.md)** — Scripts listos para copiar y pegar.

---

## 📜 Basado en

- **Repositorio original**: [Azure/GPT-RAG](https://github.com/Azure/GPT-RAG/tree/release/1.0.1) — rama `release/1.0.1`
- **Licencia**: [MIT](https://github.com/Azure/GPT-RAG/blob/release/1.0.1/LICENSE)
- **Mantenido por**: Microsoft Azure
