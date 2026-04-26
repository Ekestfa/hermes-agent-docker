# 🚀 Hermes Agent Docker Kurulum Guideline

```bash
# 1. Eski container'ları temizle
docker compose down

# 2. Tek seferlik setup (interactive)
docker compose run --rm setup

# Setup wizard'ı tamamla (LLM, API key, Telegram vs.)

# 3. Gateway ve dashboard'ı başlat
docker compose up -d gateway dashboard

# 4. Test
curl http://localhost:8642/health
curl http://localhost:9119/api/status
http://localhost:9119
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