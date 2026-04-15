## Step 4 - Install Models (WSL2)

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Start Ollama service
ollama serve &

# Pull recommended models
ollama pull gemma4:26b-a4b-it-q5_K_M
ollama pull qwen3.5:27b-it-q5_K_M
```

## Step 5 - Verify Models

```bash
ollama list  # Should show both models
hermes model status  # Should show Ollama as provider
```