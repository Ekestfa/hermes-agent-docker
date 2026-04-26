# 🚀 Hermes Agent Docker Kurulum Guideline

```bash
# Gateway ve dashboard'ı sürekli çalışır hale getir
docker compose up -d gateway dashboard

# Yeni bir terminalde interaktif setup
docker compose run --rm setup
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