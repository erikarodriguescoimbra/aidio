# AzureAI Technical Article Translator — Playbook + Prototype

Este repositório contém os arquivos individuais do playbook (Bicep) e do protótipo em Python. Você pode copiar cada bloco de código e salvar nos arquivos correspondentes.

---

## 📁 Estrutura de diretórios
```
azureai-translator/
├─ bicep/
│  ├─ main.bicep
│  ├─ storage.bicep
│  ├─ translator.bicep
│  ├─ cognitivesearch.bicep
│  └─ appservice.bicep
├─ prototype/
│  ├─ requirements.txt
│  ├─ .env.example
│  ├─ scripts/
│  │  ├─ upload_to_blob.py
│  │  ├─ document_translation.py
│  │  ├─ post_edit_openai.py
│  │  └─ cognitive_search_index.py
│  └─ glossary/
│     └─ eng-por-glossary.tmx
└─ docs/
   └─ runbook.md
```

---

## 📌 Arquivos Bicep

### bicep/main.bicep
```bicep
param location string = resourceGroup().location
param prefix string = 'aztr'

module storage 'storage.bicep' = {
  name: 'storageModule'
  params: {
    location: location
    prefix: prefix
  }
}

module translator 'translator.bicep' = {
  name: 'translatorModule'
  params: {
    location: location
    prefix: prefix
  }
}

module search 'cognitivesearch.bicep' = {
  name: 'searchModule'
  params: {
    location: location
    prefix: prefix
  }
}

module app 'appservice.bicep' = {
  name: 'appModule'
  params: {
    location: location
    prefix: prefix
    storageAccountName: storage.outputs.storageAccountName
  }
}

output storageAccountName string = storage.outputs.storageAccountName
output translatorResourceName string = translator.outputs.translatorName
output searchServiceName string = search.outputs.searchName
output appServiceName string = app.outputs.appName
```

### bicep/storage.bicep
```bicep
param location string
param prefix string
var storageName = toLower('${prefix}storage${uniqueString(resourceGroup().id)}')
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}

resource blobService 'Microsoft.Storage/storageAccounts/blobServices@2021-02-01' = {
  parent: storageAccount
  name: 'default'
}

resource inputContainer 'Microsoft.Storage/storageAccounts/blobServices/containers@2021-02-01' = {
  parent: blobService
  name: 'input'
  properties: {}
}

resource outputContainer 'Microsoft.Storage/storageAccounts/blobServices/containers@2021-02-01' = {
  parent: blobService
  name: 'output'
  properties: {}
}

output storageAccountName string = storageAccount.name
```

### bicep/translator.bicep
```bicep
param location string
param prefix string
var translatorName = '${prefix}-translator'
resource cognitiveAccount 'Microsoft.CognitiveServices/accounts@2021-10-01' = {
  name: translatorName
  location: location
  sku: {
    name: 'S0'
  }
  kind: 'CognitiveServices'
  properties: {}
}

output translatorName string = cognitiveAccount.name
```

### bicep/cognitivesearch.bicep
```bicep
param location string
param prefix string
var searchName = '${prefix}-search'
resource search 'Microsoft.Search/searchServices@2020-08-01' = {
  name: searchName
  location: location
  sku: {
    name: 'basic'
  }
  properties: {
    replicaCount: 1
    partitionCount: 1
  }
}

output searchName string = search.name
```

### bicep/appservice.bicep
```bicep
param location string
param prefix string
param storageAccountName string
var appName = '${prefix}-translator-api'
resource plan 'Microsoft.Web/serverfarms@2021-02-01' = {
  name: '${appName}-plan'
  location: location
  sku: {
    name: 'B1'
    tier: 'Basic'
    size: 'B1'
  }
  kind: 'app'
}

resource webApp 'Microsoft.Web/sites@2021-02-01' = {
  name: appName
  location: location
  kind: 'app'
  properties: {
    serverFarmId: plan.id
    siteConfig: {
      appSettings: [
        {
          name: 'AZURE_STORAGE_ACCOUNT'
          value: storageAccountName
        }
      ]
    }
  }
}

output appName string = webApp.name
```

---

## 📌 Protótipo Python

### prototype/requirements.txt
```
azure-identity>=1.12.0
azure-storage-blob>=12.10.0
requests>=2.28.0
python-dotenv>=1.0.0
openai>=0.27.0
```

### prototype/.env.example
```
AZURE_STORAGE_ACCOUNT=<your-storage-account>
AZURE_STORAGE_SAS_TOKEN=<SAS_TOKEN>
TRANSLATOR_ENDPOINT=https://<your-translator-endpoint>.cognitiveservices.azure.com
TRANSLATOR_KEY=<translator-key>
AZURE_OPENAI_ENDPOINT=https://<your-openai-resource>.openai.azure.com
AZURE_OPENAI_KEY=<openai-key>
```

### prototype/scripts/upload_to_blob.py
```python
import os
from azure.storage.blob import BlobClient
from dotenv import load_dotenv

load_dotenv()

STORAGE_ACCOUNT = os.getenv('AZURE_STORAGE_ACCOUNT')
SAS = os.getenv('AZURE_STORAGE_SAS_TOKEN')

def upload_file(container: str, local_path: str, blob_name: str):
    url = f"https://{STORAGE_ACCOUNT}.blob.core.windows.net/{container}/{blob_name}?{SAS}"
    blob = BlobClient.from_blob_url(url)
    with open(local_path, 'rb') as f:
        blob.upload_blob(f, overwrite=True)
    print(f"Uploaded {local_path} -> {container}/{blob_name}")

if __name__ == '__main__':
    upload_file('input', 'example/article1.pdf', 'article1.pdf')
    upload_file('glossaries', 'glossary/eng-por-glossary.tmx', 'eng-por-glossary.tmx')
```

### prototype/scripts/document_translation.py
```python
import os
import requests
import json
from dotenv import load_dotenv

load_dotenv()
TRANSLATOR_ENDPOINT = os.getenv('TRANSLATOR_ENDPOINT')
TRANSLATOR_KEY = os.getenv('TRANSLATOR_KEY')

SOURCE_URL = 'https://<account>.blob.core.windows.net/input/article1.pdf?<SAS>'
TARGET_URL = 'https://<account>.blob.core.windows.net/output/article1-pt.pdf?<SAS>'
GLOSSARY_URL = 'https://<account>.blob.core.windows.net/glossaries/eng-por-glossary.tmx?<SAS>'

def create_translation_job():
    url = TRANSLATOR_ENDPOINT.rstrip('/') + '/translator/text/batch/v1.1/batches'
    payload = {
        "inputs": [
            {
                "source": {"sourceUrl": SOURCE_URL},
                "targets": [
                    {
                        "targetUrl": TARGET_URL,
                        "language": "pt",
                        "glossaries": [
                            {"url": GLOSSARY_URL, "format": "tmx"}
                        ]
                    }
                ]
            }
        ]
    }
    headers = {
        'Ocp-Apim-Subscription-Key': TRANSLATOR_KEY,
        'Content-Type': 'application/json'
    }
    resp = requests.post(url, headers=headers, json=payload)
    resp.raise_for_status()
    print('Job created:', resp.json())

if __name__ == '__main__':
    create_translation_job()
```

### prototype/scripts/post_edit_openai.py
```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()
OA_ENDPOINT = os.getenv('AZURE_OPENAI_ENDPOINT')
OA_KEY = os.getenv('AZURE_OPENAI_KEY')
MODEL_DEPLOYMENT = os.getenv('AZURE_OPENAI_DEPLOYMENT', 'gpt-4o-mini')

mt_output_example = '''O inversor está conectado ao painel e o termopar detectou...'''

GLOSSARY = {
    'inverter': 'inversor',
    'thermocouple': 'termopar',
}

PROMPT_TEMPLATE = '''Você é um revisor técnico. Use este glossário.
Glossário: {glossary}
MT output:\n{mt}
Regras:
- Substitua termos conforme glossário.
- Primeira ocorrência: mantenha termo original entre parênteses.
- Não traduza siglas.
- Mantenha estilo técnico.
Retorne apenas o texto pós-editado.
'''

def call_openai_postedit(mt_text: str):
    prompt = PROMPT_TEMPLATE.format(glossary=', '.join([f"{k}->{v}" for k,v in GLOSSARY.items()]), mt=mt_text)
    url = OA_ENDPOINT.rstrip('/') + '/openai/deployments/' + MODEL_DEPLOYMENT + '/completions?api-version=2024-07-01'
    headers = {'api-key': OA_KEY, 'Content-Type': 'application/json'}
    payload = {'prompt': prompt, 'max_tokens': 800, 'temperature': 0.0}
    resp = requests.post(url, headers=headers, json=payload)
    resp.raise_for_status()
    return resp.json()['choices'][0]['text']

if __name__ == '__main__':
    edited = call_openai_postedit(mt_output_example)
    print('Pós-edição:')
    print(edited)
```

### prototype/scripts/cognitive_search_index.py
```python
# Esboço para indexar documentos no Cognitive Search
# Implementar com azure-search-documents + embeddings Azure OpenAI
```

### prototype/glossary/eng-por-glossary.tmx
```xml
<?xml version="1.0" encoding="UTF-8"?>
<tmx version="1.4">
  <body>
    <tu>
      <tuv xml:lang="en"><seg>thermocouple</seg></tuv>
      <tuv xml:lang="pt"><seg>termopar</seg></tuv>
    </tu>
    <tu>
      <tuv xml:lang="en"><seg>inverter</seg></tuv>
      <tuv xml:lang="pt"><seg>inversor</seg></tuv>
    </tu>
  </body>
</tmx>
```

---

## 📌 docs/runbook.md

```
1. Deploy Bicep: az deployment group create -g <rg> -f bicep/main.bicep --parameters prefix=<prefix>
2. Criar SAS tokens para input, glossaries, output.
3. Configurar .env no prototype.
4. pip install -r requirements.txt
5. python scripts/upload_to_blob.py
6. python scripts/document_translation.py
7. Após job, baixar saída e rodar python scripts/post_edit_openai.py
```

---

Agora os arquivos estão adicionados individualmente no canvas.

