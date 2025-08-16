# GPT-OSSRFU (One-Click Self-Hosted LLM)

Este projeto oferece uma solução simplificada para hospedar modelos de linguagem de grande porte (LLMs) em sua própria infraestrutura com apenas um comando. O objetivo é fornecer uma experiência semelhante à da API da OpenAI, mas executada em seu servidor local, com uma interface de usuário web e endpoints de API compatíveis.

A solução é conteinerizada usando Docker e oferece duas versões distintas para máxima flexibilidade: uma otimizada para execução em **CPU** e outra para execução em **GPU**.

## Stack de Tecnologias

- **Orquestração:** Docker e Docker Compose
- **Interface Web:** Open WebUI
- **Backend (CPU):** Llama.cpp
- **Backend (GPU):** vLLM

---

## Arquitetura

O projeto utiliza uma arquitetura baseada em serviços, com configurações diferentes para os ambientes de CPU e GPU.

### Versão CPU

A versão para CPU, gerenciada pelo `docker-compose.cpu.yml`, levanta dois serviços principais baseados no **Llama.cpp** para otimizar o uso de recursos:

1.  **Serviço de Chat/Completions (Porta `8000`)**
    -   **Função:** Processa pedidos de geração de texto, como chat e conclusões de prompts.
    -   **Endpoint Base:** `http://localhost:8000/v1`
    -   **Recursos:** `/chat/completions`, `/completions`

2.  **Serviço de Embeddings (Porta `8001`)**
    -   **Função:** Gera representações vetoriais (embeddings) de textos.
    -   **Endpoint Base:** `http://localhost:8001/v1`
    -   **Recursos:** `/embeddings`

Essa separação permite o uso de modelos diferentes e otimizados para cada tarefa.

### Versão GPU

A versão para GPU, gerenciada pelo `docker-compose.gpu.yml`, utiliza o **vLLM** para um backend de inferência de alta performance e o **Open WebUI** como interface de usuário.

1.  **Serviço de API (vLLM) (Porta `8000`)**
    -   **Função:** Fornece uma API de inferência de alta velocidade, compatível com o formato da API da OpenAI.
    -   **Endpoint:** `http://localhost:8000/v1`

2.  **Serviço de Web UI (Open WebUI) (Porta `3000`)**
    -   **Função:** Oferece uma interface de usuário gráfica para interagir com o modelo.
    -   **Acesso:** `http://localhost:3000`

---

## Pré-requisitos

- **Docker** e **Docker Compose** instalados.
- Para a **Versão GPU**:
    - Uma GPU NVIDIA compatível.
    - Drivers NVIDIA e NVIDIA Container Toolkit devidamente instalados.

---

## Como Executar

### 1. Preparação do Modelo

Antes de iniciar, você precisa baixar os modelos de linguagem que deseja usar. Renomeie os arquivos dos modelos e coloque-os no diretório `./models`.

### 2. Configuração do Ambiente

Edite os arquivos `.env` para ajustar as configurações do seu ambiente.

#### Para a Versão CPU (`cpu.env`)

```env
# Nome do arquivo do modelo para Chat/Completions (deve estar em ./models)
MODEL_FILENAME=seu-modelo-de-chat.gguf

# Configurações de performance
CPU_THREADS=8
N_CTX=4096
N_GPU_LAYERS=0 # Mantenha em 0 para modo CPU

# Portas
HOST_LLAMA_PORT=8000
LLAMA_PORT=8000
HOST_WEBUI_PORT=3000

# Nome do arquivo do modelo para Embeddings (deve estar em ./models)
MODEL_EMB_FILENAME=seu-modelo-de-embeddings.gguf
CPU_THREADS_EMB=6
HOST_LLAMA_EMB_PORT=8001
LLAMA_EMB_PORT=8001
```

#### Para a Versão GPU (`gpu.env`)

```env
# ===== Autenticação =====
# Defina o mesmo token para a API do vLLM e para a WebUI
VLLM_API_KEY=seu-token-secreto-e-forte
WEBUI_OPENAI_KEY=seu-token-secreto-e-forte

# ===== Modelo / Execução =====
# ID do modelo do Hugging Face (ex: openai/gpt-oss-20b)
MODEL_ID=openai/gpt-oss-20b

# Quantização (opcional, ex: awq, gptq)
QUANTIZATION=

# Ajustes de memória e contexto
MAX_MODEL_LEN=131072
GPU_MEMORY_UTILIZATION=0.9
TENSOR_PARALLEL_SIZE=1 # Número de GPUs a serem usadas

# ===== Portas =====
HOST_VLLM_PORT=8000
VLLM_PORT=8000
HOST_WEBUI_PORT=3000
```

### 3. Iniciando os Serviços

Execute o comando correspondente à versão desejada no terminal, a partir da raiz do projeto.

**Para iniciar a Versão CPU:**

```bash
docker-compose -f docker-compose.cpu.yml --env-file cpu.env up -d
```

**Para iniciar a Versão GPU:**

```bash
docker-compose -f docker-compose.gpu.yml --env-file gpu.env up -d
```

---

## Como Usar a API (Exemplos `curl`)

A API é compatível com o formato da OpenAI, facilitando a integração com ferramentas existentes.

### Versão CPU e GPU: Chat/Completions

**Endpoint:** `http://localhost:8000/v1/chat/completions`

```bash
curl -s http://localhost:8000/v1/chat/completions \
-H "Authorization: Bearer seu-token-secreto-e-forte" \
-H "Content-Type: application/json" \
-d 
{
  "model": "default",
  "messages": [{"role": "user", "content": "Explique o que é RAG em 2 linhas."}]
}
```
*Nota: Na versão CPU, o token de autorização é ignorado por padrão, mas o cabeçalho pode ser necessário para compatibilidade com alguns clientes.*

### Versão CPU: Embeddings

**Endpoint:** `http://localhost:8001/v1/embeddings`

```bash
curl -s http://localhost:8001/v1/embeddings \
-H "Authorization: Bearer dummy" \
-H "Content-Type: application/json" \
-d 
{
  "model": "default",
  "input": ["texto 1", "texto 2"]
}
```