# Tradutor de Artigos Técnicos com Azure AI

Este projeto é uma solução completa para automatizar a tradução de documentos técnicos, utilizando um pipeline de serviços de Inteligência Artificial do Azure. A solução provisiona a infraestrutura necessária como código (Bicep), executa a tradução em lote de documentos, utiliza glossários para garantir a consistência de termos técnicos e aplica uma camada de pós-edição com Azure OpenAI para refinar o resultado.

---

## 🚀 Funcionalidades Principais

-   **Infraestrutura como Código (IaC)**: Provisionamento de todos os recursos do Azure de forma automatizada e replicável com arquivos Bicep.
-   **Tradução em Lote**: Capacidade de traduzir múltiplos documentos de uma só vez usando o serviço Azure AI Translator.
-   **Consistência Terminológica**: Suporte para glossários no formato TMX, garantindo que termos técnicos específicos (ex: *inverter* -> *inversor*) sejam traduzidos corretamente em todo o documento.
-   **Pós-edição com IA Generativa**: Utilização do Azure OpenAI para revisar e refinar o texto traduzido, aplicando regras específicas como manter siglas e ajustar o estilo técnico.
-   **Pronto para Busca Inteligente**: A arquitetura inclui o Azure AI Search, preparando o terreno para indexar os documentos traduzidos e torná-los pesquisáveis.

---

## 🛠️ Tecnologias Utilizadas

### Serviços do Azure
-   **Azure Storage Account**: Para armazenar os documentos de entrada, saída e os arquivos de glossário.
-   **Azure AI Translator**: Serviço principal para a tradução de documentos em lote.
-   **Azure AI Search**: Para futura indexação e busca nos documentos traduzidos.
-   **Azure App Service**: Para hospedar uma possível API ou interface web para a solução.
-   **Azure OpenAI** (Pré-requisito): Utilizado no script de pós-edição para refinamento do texto.

### Protótipo
-   **Python 3.8+**
-   **Bibliotecas**: `azure-storage-blob`, `requests`, `python-dotenv`, `openai`.

---

## ⚙️ Pré-requisitos

Antes de começar, garanta que você tenha as seguintes ferramentas instaladas:
1.  [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
2.  [Python 3.8 ou superior](https://www.python.org/downloads/)
3.  Um editor de código, como o [Visual Studio Code](https://code.visualstudio.com/).

---

## 🏁 Como Executar o Projeto

Siga os passos abaixo para configurar e executar o pipeline de tradução.

### 1. Clonar o Repositório
```sh
git clone [https://github.com/SEU-USUARIO/azureai-translator.git](https://github.com/SEU-USUARIO/azureai-translator.git)
cd azureai-translator
