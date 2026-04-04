# 🚀 Guia Rápido - Fine-tuning Gemma 3.1B

## Comece em 5 minutos!

### ⚡ Setup Rápido

```bash
# 1. Ativar virtual environment
source .venv/bin/activate

# 2. Abrir os notebooks
jupyter notebook src/
```

## 📖 Workflow Completo

### Passo 1: Fine-tuning (1-24 horas dependendo do hardware)
1. Abra `src/fine-tune-small-llm.ipynb`
2. Execute todas as células (Shift + Enter)
3. Espere o treinamento completar
4. Modelo salvo em `results/fine-tuned-model/`

### Passo 2: Exportar para vLLM (5-10 minutos)
1. Abra `src/vllm-export.ipynb`
2. Execute todas as células
3. Modelo vLLM salvo em `results/vllm-model/`

### Passo 3: Testar o Modelo (5-15 minutos)
1. Abra `src/test-model.ipynb`
2. Execute todas as células
3. Resultados salvos em `results/test_results/`

## 🎯 Casos de Uso

### ✅ Para Desenvolvimento/Prototipagem
- Use o notebook de fine-tuning padrão
- Teste localmente com test-model.ipynb
- Tempo: ~1-2 horas em GPU

### ✅ Para Produção
- Complete todo o pipeline
- Exporte com vLLM para melhor performance
- Deploy com Docker ou servidor vLLM
- Tempo: ~2-24 horas dependendo do dataset

### ✅ Para Pesquisa
- Experimente com diferentes hiperparâmetros
- Modifique o config.yaml
- Compare resultados em test-results/

## 📊 Monitorar o Treinamento

### Durante Execução
```bash
# Em outro terminal, monitorar GPU
nvidia-smi -l 1  # Atualizar a cada 1 segundo
```

### Logs de Treinamento
- Você verá `loss` diminuindo ao longo do treinamento
- Ideal: training_loss deve cair para < 0.5

### Checkpoints
- Salvos a cada 100 passos em `results/checkpoint/`
- Melhor checkpoint carregado automaticamente ao fim

## 🔧 Customizar o Projeto

### 1. Usar Seu Próprio Dataset

No notebook de fine-tuning, substitua:
```python
# Original:
dataset = load_dataset('squad', split="train")

# Seu dataset:
dataset = load_dataset('seu_dataset', split="train")
```

### 2. Mudar Hiperparâmetros

Edite `config.yaml`:
```yaml
training:
  num_train_epochs: 5  # Mais epochs
  learning_rate: 0.0001  # Learning rate menor
  batch_size: 8  # Batch maior
```

### 3. Usar Modelo Diferente

No notebook:
```python
model_name = "unsloth/Llama-3.2-1B"  # Outro modelo
```

## ⚠️ Troubleshooting Comum

| Problema | Solução |
|----------|---------|
| CUDA out of memory | Reduza batch_size para 2 |
| Modelo não carrega | Verifique caminho em `results/` |
| Inferência lenta | Use vLLM em vez de Transformers |
| Dependências não instalam | Use `pip install --no-deps` |
| Training muito lento | Verifique se você está em GPU com `nvidia-smi` |

## 📚 Próximos Passos

1. **Deploy Local:** Use vLLM server com `python -m vllm.entrypoints.openai.api_server`
2. **Deploy Cloud:** Push para HuggingFace Hub ou deploy em Azure/AWS
3. **Integração:** Use OpenAI-compatible API do vLLM
4. **Melhorias:** Fine-tune com seus dados reais para melhores resultados

## 📞 Precisa de Ajuda?

- 📖 Veja documentação completa em `README.md`
- 🐛 Reporte issues no GitHub
- 💬 Confira as perguntas comuns na seção Troubleshooting

## ✨ Tips Profissionais

1. **Economize tempo:** Comece com um dataset menor para validar pipeline
2. **Economize memória:** Use quantização 4-bit (já ativada)
3. **Economize custo:** Use spot instances em cloud providers
4. **Melhore resultados:** Fine-tune com dados de domínio específico
5. **Ganhe speed:** Use vLLM para serving em produção

---

**Pronto para começar? Execute a célula 1 de `fine-tune-small-llm.ipynb`!** 🚀
