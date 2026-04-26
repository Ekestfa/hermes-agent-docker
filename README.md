# 🚀 Hermes Agent Docker Kurulum Guideline

## 📋 1. Docker Compose YAML Dosyası

docker-compose.yml (Production-Ready)
```yaml
version: '3.8'

services:
  # ──────────────────────────────────────────────────────────
  # Hermes Agent Gateway (Ana Servis)
  # ──────────────────────────────────────────────────────────
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes-agent
    restart: unless-stopped
    command: gateway run --host 0.0.0.0 --port 8642
    networks:
      - hermes-net
    ports:
      - "8642:8642"        # Gateway API
      - "9119:9119"        # Dashboard (Web Terminal)
    volumes:
      - hermes_data:/opt/data  # Kalıcı bellek, skills, config <-- HOST shared volume için path değişikliği
    environment:
      - GATEWAY_HEALTH_URL=http://hermes:8642
      - OLLAMA_BASE_URL=http://ollama:11434  # Ollama entegrasyonu
    deploy:
      resources:
        limits:
          memory: 8G
          cpus: "4.0"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8642/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ──────────────────────────────────────────────────────────
  # Redis (Bellek optimizasyonu)
  # ──────────────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: hermes-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --maxmemory 2gb --maxmemory-policy allkeys-lru
    networks:
      - hermes-net
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      retries: 3

networks:
  hermes-net: # RAG için ortak ağ değişikliği
    driver: bridge

volumes:
  hermes_data:
  ollama_data:
  redis_data:
```

.env Dosyası (API Anahtarları)
```yaml
# Ollama modeli (otomatik çekilir)
OLLAMA_MODEL=llama3.2:70b  # veya mistral-nemo:12b, gemma2:27b

# Dashboard giriş bilgileri
DASHBOARD_USERNAME=admin
DASHBOARD_PASSWORD=your_secure_password

# MCP için (opsiyonel)
MCP_GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
MCP_POSTGRES_URL=postgresql://user:pass@postgres:5432/hermes
```

## Ollama Provider Konfigürasyonu
```yaml
# /opt/data/config.yaml
provider:
  type: ollama
  base_url: "http://host.docker.internal:11434"  # 🎯 Host Ollama bağlantısı
  models:
    - name: "llama3.2:70b"
      context_window: 128000
      max_tokens: 4096
      temperature: 0.7
    - name: "mistral-nemo:12b"
      context_window: 128000
      max_tokens: 4096
      temperature: 0.3

# Fallback (opsiyonel - internet varsa)
fallback:
  type: openai
  api_key: ${OPENAI_API_KEY}
  model: gpt-4o-mini
```

## MCP Konfigürasyonu
```yaml
mcp:
  auto_discovery: true
  
  servers:
    # GitHub MCP Server
    - name: github
      type: stdio
      command: npx
      args:
        - @modelcontextprotocol/server-github
        - --token=${MCP_GITHUB_TOKEN}
    
    # Dosya Sistemi MCP Server
    - name: filesystem
      type: stdio
      command: npx
      args:
        - @modelcontextprotocol/server-filesystem
        - /opt/data/shared
    
    # Uzak MCP Server (aynı network'teki servisler)
    - name: agent-zero
      type: http
      url: http://agent-zero:8080/mcp
    
    - name: lightrag
      type: http
      url: http://lightrag:8000/mcp
```