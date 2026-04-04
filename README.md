# Fine-tuning Gemma 3.1B com Unsloth para QA

Um projeto completo de fine-tuning de modelos LLM pequenos usando Unsloth, otimizado para tarefas de perguntas e respostas (QA).

## 📋 Visão Geral

Este projeto fornece uma pipeline profissional para:
- **Fine-tuning** de modelos Gemma 3.1B em dados SQuAD
- **Exportação** para vLLM com otimizações de inferência
- **Testes e validação** da qualidade do modelo treinado

Usando a biblioteca **Unsloth** para otimizações avançadas de treinamento que reduzem o uso de memória em até 80% e aceleram o treinamento em 2-5x.

## 🚀 Estrutura do Projeto

```
.
├── src/
│   ├── fine-tune-small-llm.ipynb    # Script principal de fine-tuning
│   ├── vllm-export.ipynb            # Exportação para vLLM
│   └── test-model.ipynb             # Testes e validação
├── results/
│   ├── checkpoint/                  # Checkpoints durante treinamento
│   ├── fine-tuned-model/            # Modelo fine-tuned salvo
│   ├── vllm-model/                  # Modelo exportado para vLLM
│   └── test_results/                # Resultados dos testes
├── README.md                        # Esta documentação
└── .venv/                          # Virtual environment
```

## 🛠️ Instalação e Setup

### Pré-requisitos
- Python 3.10+
- CUDA 12.1+ (recomendado para GPU)
- 8GB+ de vram ou 16GB+ de RAM

### 1. Clonar repositório

```bash
git clone https://github.com/wyctorfogos/Fine-tunning-Gemma3-1b-unsloth.git
cd Fine-tunning-Gemma3-1b-unsloth
```

### 2. Criar virtual environment

```bash
python -m venv .venv
source .venv/bin/activate  # No Windows: .venv\Scripts\activate
```

### 3. Instalar dependências

As dependências serão instaladas automaticamente em cada notebook quando executado. Alternativamente:

```bash
pip install -r requirements.txt
```

## 📚 Notebooks

### 1. Fine-tuning (`fine-tune-small-llm.ipynb`)

Script principal que realiza o treinamento do modelo.

**Pipeline:**
1. ✅ Instalação de dependências (Unsloth, Transformers, TRL, PEFT)
2. ✅ Carregamento do dataset SQuAD
3. ✅ Formatação de dados para QA
4. ✅ Carregamento do modelo Gemma 3.1B com quantização 4-bit
5. ✅ Configuração de LoRA
6. ✅ Tokenização dos dados
7. ✅ Configuração de hiperparâmetros de treinamento
8. ✅ Execução do fine-tuning com SFTTrainer
9. ✅ Salvamento do modelo treinado

**Configuração de Treinamento:**
- **Modelo Base:** unsloth/gemma-3-270m-it-unsloth-bnb-4bit
- **Quantização:** 4-bit (reduz memória em 75%)
- **LoRA Rank:** 16 (r=16)
- **LoRA Alpha:** 32
- **Batch Size:** 4 (per device)
- **Learning Rate:** 2e-4
- **Epochs:** 3
- **Otimizador:** adamw_8bit

**Tempo estimado de treinamento:**
- GPU (NVIDIA A100): ~2-3 horas
- GPU (RTX 3090): ~4-6 horas
- GPU (T4): ~8-12 horas
- CPU: ~24-48 horas

### 2. vLLM Export (`vllm-export.ipynb`)

Exporta o modelo fine-tuned para o formato vLLM, otimizado para inferência de alta performance.

**Pipeline:**
1. ✅ Instalação de vLLM
2. ✅ Carregamento do modelo fine-tuned
3. ✅ Conversão para formato vLLM
4. ✅ Criação de configuração de deployment
5. ✅ Validação com teste de inferência
6. ✅ Instruções para produção

**Recursos vLLM:**
- Servidor OpenAI-compatible API
- Batching automático de requisições
- Suporte a quantização (AWQ, GPTQ)
- Caching KV para melhor performance
- Suporte a múltiplas GPUs

### 3. Test Model (`test-model.ipynb`)

Valida o modelo com testes de inferência e gera relatórios de performance.

**Pipeline:**
1. ✅ Carregamento do modelo fine-tuned
2. ✅ Definição de 5 casos de teste padrão
3. ✅ Execução de inferências
4. ✅ Medição de tempo de inferência
5. ✅ Testes personalizados
6. ✅ Salvamento de resultados

**Métricas Coletadas:**
- Tempo de inferência por query
- Qualidade das respostas (análise manual)
- Tempo médio de respostas
- Estatísticas gerais

## 🔧 Hiperparâmetros Recomendados

### Para SQuAD (Padrão)
```
- batch_size: 4
- learning_rate: 2e-4
- num_epochs: 3
- warmup_steps: 100
- max_seq_length: 2048
```

### Para dados menores (< 10k samples)
```
- batch_size: 8
- learning_rate: 5e-4
- num_epochs: 5
- warmup_steps: 50
- max_seq_length: 1024
```

### Para dados maiores (> 100k samples)
```
- batch_size: 16
- learning_rate: 1e-4
- num_epochs: 1-2
- warmup_steps: 200
- max_seq_length: 2048
```

## 📊 Resultados Esperados

Após fine-tuning no SQuAD:
- **Perplexidade:** ~15-20 (dependendo dos dados)
- **BLEU Score:** ~25-35
- **Tempo de inferência (single GPU):** ~0.5-1s por query
- **Tamanho do modelo:** ~540MB (base) + ~30MB (LoRA)

## 🚀 Deployment em Produção

### Opção 1: vLLM Server (Recomendado)

```bash
python -m vllm.entrypoints.openai.api_server \
    --model ./results/vllm-model \
    --dtype half \
    --gpu-memory-utilization 0.9 \
    --port 8000
```

### Opção 2: Docker

```dockerfile
FROM nvidia/cuda:12.1.0-runtime-ubuntu22.04
WORKDIR /app
COPY results/vllm-model /app/model
RUN pip install vllm transformers
CMD ["python", "-m", "vllm.entrypoints.openai.api_server", \
     "--model", "/app/model"]
```

### Opção 3: HuggingFace Transformers

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained("./results/fine-tuned-model")
tokenizer = AutoTokenizer.from_pretrained("./results/fine-tuned-model")

# Inference
inputs = tokenizer("Your prompt here", return_tensors="pt")
outputs = model.generate(**inputs, max_length=256)
print(tokenizer.decode(outputs[0]))
```

## 📈 Monitoramento e Otimização

### Durante Treinamento
- Monitore `trainer_stats.training_loss`
- Verifique uso de VRAM com `nvidia-smi`
- Valide em dataset de eval a cada 100 passos

### Pós-Treinamento
- Teste com dados não vistos
- Compare com baseline
- Meça latência e throughput
- Colha feedback dos usuários

## 🔍 Troubleshooting

### Erro: "CUDA out of memory"
- Reduza `batch_size` para 2
- Reduza `max_seq_length` para 1024
- Use `gradient_checkpointing=True`

### Erro: "Model not found"
- Verifique caminhos dos modelos
- Verifique conexão com HuggingFace Hub
- Tente fazer download manual antes

### Inferência lenta
- Use vLLM em vez de Transformers
- Aumente `gpu_memory_utilization` em vLLM
- Use quantização AWQ ou GPTQ
- Considere multi-GPU setup

## 📚 Referências

- [Unsloth Documentation](https://github.com/unslothai/unsloth)
- [vLLM Documentation](https://docs.vllm.ai/)
- [SQuAD Dataset](https://rajpurkar.github.io/SQuAD-explorer/)
- [Transformers](https://huggingface.co/transformers/)
- [TRL (Transformer Reinforcement Learning)](https://github.com/huggingface/trl)

## 📝 Licença

Este projeto está sob licença MIT. Veja LICENSE para mais detalhes.

## 🤝 Contribuições

Contribuições são bem-vindas! Por favor:
1. Faça um Fork
2. Crie uma branch (`git checkout -b feature/amazing-feature`)
3. Commit suas mudanças (`git commit -m 'Add amazing feature'`)
4. Push para a branch (`git push origin feature/amazing-feature`)
5. Abra um Pull Request

## 📧 Contato

- GitHub: [@wyctorfogos](https://github.com/wyctorfogos)
- Issues: [GitHub Issues](https://github.com/wyctorfogos/Fine-tunning-Gemma3-1b-unsloth/issues)

## ⭐ Se este projeto ajudou você, considere dar uma estrela!

---

**Última atualização:** Abril 2026
**Status:** ✅ Pronto para produção
