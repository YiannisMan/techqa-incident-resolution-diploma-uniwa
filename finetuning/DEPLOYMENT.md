# Ανάπτυξη του εξειδικευμένου μοντέλου (adapter → Ollama)

## 1. Μετατροπή του LoRA adapter σε GGUF
 ```bash
python llama.cpp/convert_lora_to_gguf.py ./adapter --outtype f16 --outfile techqa-lora-f16.gguf
ls -lh techqa-lora-f16.gguf
```

 ```bash
python llama.cpp/convert_lora_to_gguf.py ./adapter \
    --base-model-id NousResearch/Meta-Llama-3-8B-Instruct \
    --outtype f16 --outfile techqa-lora-f16.gguf
```

 ```bash
cp ./adapter/adapter_config.json ./adapter/adapter_model.json
```

## 2. Δημιουργία του μοντέλου στο Ollama
 ```bash
ollama create techqa-llama3 -f Modelfile
```
