# 🚀 n8n Self-Hosted - Stack de Automação Completa

> **Solução simples, escalável e de baixa manutenção para side hustles**

Uma stack **pronta para produção** do n8n com PostgreSQL, baseada nas **melhores práticas oficiais** para máxima confiabilidade e mínimo esforço de manutenção.

## 📋 **O que está incluído**

- ✅ **n8n Latest (1.103.2)** - Versão estável mais recente
- ✅ **PostgreSQL 15** - Banco de dados robusto para produção
- ✅ **Configuração de segurança** - Criptografia e autenticação
- ✅ **Volumes persistentes** - Dados protegidos contra perda
- ✅ **Healthchecks** - Monitoramento automático
- ✅ **Escalabilidade preparada** - Worker mode e horizontal scaling
- ✅ **Limpeza automática** - Gerenciamento de dados antigos

---

## 🎯 **Quick Start (5 minutos)**

### **1. Clone e Configure**

```bash
# Clone este repositório
git clone https://github.com/Lmshimokawa/n8n_self_hosted_lms.git
cd n8n_self_hosted_lms

# Copie e configure as variáveis de ambiente
cp env.template .env

# IMPORTANTE: Edite o arquivo .env com suas configurações
# Mude pelo menos: POSTGRES_PASSWORD e N8N_ENCRYPTION_KEY
```

### **2. Gere Chave de Criptografia Segura**

```bash
# Opção 1: Com OpenSSL
openssl rand -hex 16

# Opção 2: Com Node.js
node -e "console.log(require('crypto').randomBytes(16).toString('hex'))"

# Copie o resultado para N8N_ENCRYPTION_KEY no arquivo .env
```

### **3. Execute a Stack**

```bash
# Inicie os containers
docker compose up -d

# Verifique se está funcionando
docker compose ps
docker compose logs -f n8n
```

### **4. Acesse o n8n**

Abra seu navegador em: **http://localhost:5678**

Na primeira vez, você será solicitado a criar uma conta de administrador.

---

## ⚙️ **Configurações Detalhadas**

### **Arquivo .env - Configurações Essenciais**

| Variável | Descrição | Valor Padrão |
|----------|-----------|--------------|
| `PROJECT_NAME` | Nome do projeto (usado nos volumes) | `meu_n8n_project` |
| `POSTGRES_PASSWORD` | **🔴 OBRIGATÓRIO:** Senha do banco | `change_me!` |
| `N8N_ENCRYPTION_KEY` | **🔴 OBRIGATÓRIO:** Chave de 32 chars | `GENERATE_KEY` |
| `N8N_HOST` | Host do n8n (localhost/domínio) | `localhost` |
| `WEBHOOK_URL` | URL completa para webhooks | `http://localhost:5678/` |
| `TIMEZONE` | Timezone para agendamentos | `America/Sao_Paulo` |

### **Configurações de Produção**

Para usar em produção, edite no `.env`:

```bash
# Produção
N8N_HOST=n8n.meudominio.com
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.meudominio.com/

# Segurança adicional
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=senha_muito_segura
```

---

## 🔧 **Comandos Essenciais**

### **Operações Básicas**

```bash
# Iniciar
docker compose up -d

# Parar
docker compose down

# Reiniciar
docker compose restart

# Ver logs
docker compose logs -f n8n
docker compose logs -f postgres

# Status dos containers
docker compose ps
```

### **Manutenção**

```bash
# Atualizar n8n para a versão mais recente
docker compose pull
docker compose up -d

# Backup dos dados
docker compose exec postgres pg_dump -U n8n_admin n8n_production > backup_$(date +%Y%m%d).sql

# Acessar container do n8n
docker compose exec n8n sh

# Acessar banco de dados
docker compose exec postgres psql -U n8n_admin -d n8n_production
```

---

## 💾 **Backup e Recuperação**

### **Backup Automático Recomendado**

Crie um script de backup simples:

```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="./backups"

# Criar diretório de backup
mkdir -p $BACKUP_DIR

# Backup do banco PostgreSQL
docker compose exec -T postgres pg_dump -U n8n_admin n8n_production > "$BACKUP_DIR/n8n_db_$DATE.sql"

# Backup dos volumes Docker (workflows e configurações)
docker run --rm -v meu_n8n_project_n8n_data:/source -v $(pwd)/backups:/backup ubuntu tar czf /backup/n8n_data_$DATE.tar.gz /source

echo "Backup completo: $DATE"
```

Execute com:
```bash
chmod +x backup.sh
./backup.sh
```

### **Recuperação**

```bash
# Restaurar banco
docker compose exec -T postgres psql -U n8n_admin -d n8n_production < backup_20250127_120000.sql

# Restaurar volumes
docker run --rm -v meu_n8n_project_n8n_data:/target -v $(pwd)/backups:/backup ubuntu tar xzf /backup/n8n_data_20250127_120000.tar.gz -C /target --strip-components=1
```

---

## 📈 **Escalabilidade e Performance**

### **Configurações de Performance**

No arquivo `.env`, ajuste conforme necessário:

```bash
# Para workflows pesados
EXECUTIONS_DATA_MAX_AGE=168  # 7 dias (ao invés de 30)
N8N_PAYLOAD_SIZE_MAX=32      # 32MB max payload

# Para alta concorrência
N8N_RUNNERS_ENABLED=true
```

### **Horizontal Scaling (Futuro)**

Quando sua operação crescer, descomente no `docker-compose.yml`:

```yaml
# Descomentar para worker mode
N8N_WORKER_ENABLED: true
```

E adicione mais containers worker conforme necessário.

### **Monitoramento de Recursos**

```bash
# Ver uso de CPU/RAM dos containers
docker stats

# Ver tamanho dos volumes
docker system df -v
```

---

## 🛡️ **Segurança**

### **Checklist de Segurança**

- [ ] **Senha do PostgreSQL alterada** (`POSTGRES_PASSWORD`)
- [ ] **Chave de criptografia única** (`N8N_ENCRYPTION_KEY`)
- [ ] **Autenticação básica ativada** para produção (`N8N_BASIC_AUTH_ACTIVE=true`)
- [ ] **Firewall configurado** (apenas portas necessárias)
- [ ] **HTTPS configurado** em produção
- [ ] **Backups regulares** configurados

### **Para Produção: SSL/HTTPS**

Para produção, recomenda-se usar um reverse proxy. Descomente no `docker-compose.yml`:

```yaml
# nginx:
#   image: nginx:alpine
#   # ... configuração SSL
```

Ou use serviços como Cloudflare, Caddy, ou Traefik.

---

## 🔍 **Troubleshooting**

### **Problemas Comuns**

#### **n8n não inicia**
```bash
# Verificar logs
docker compose logs n8n

# Verificar se PostgreSQL está saudável
docker compose ps postgres
```

#### **Erro de conexão com banco**
```bash
# Verificar credenciais no .env
cat .env | grep POSTGRES

# Testar conexão manual
docker compose exec postgres psql -U n8n_admin -d n8n_production -c "\dt"
```

#### **Workflows não salvam**
```bash
# Verificar chave de criptografia
cat .env | grep N8N_ENCRYPTION_KEY

# Verificar volumes
docker volume ls | grep n8n
```

#### **Performance lenta**
```bash
# Verificar recursos
docker stats

# Limpar dados antigos
docker compose exec postgres psql -U n8n_admin -d n8n_production -c "DELETE FROM execution_entity WHERE startedAt < NOW() - INTERVAL '7 days';"
```

---

## 📂 **Estrutura do Projeto**

```
n8n-selfhosted/
├── docker-compose.yml      # Configuração principal
├── env.template           # Template de configuração
├── .env                   # Suas configurações (não commitado)
├── .gitignore            # Arquivos ignorados
├── shared-files/         # Arquivos compartilhados com n8n
│   └── .gitkeep         # Preserva diretório no Git
├── README.md            # Esta documentação
└── scripts/             # Scripts úteis (opcional)
    ├── backup.sh        # Script de backup
    └── update.sh        # Script de atualização
```

---

## 🔄 **Atualizações**

### **Atualizações Regulares**

```bash
# 1. Fazer backup antes de atualizar
./backup.sh

# 2. Baixar imagens mais recentes
docker compose pull

# 3. Reiniciar com novas imagens
docker compose up -d

# 4. Verificar se tudo está funcionando
docker compose ps
docker compose logs -f n8n
```

### **Versões Específicas**

Para usar uma versão específica do n8n, edite o `docker-compose.yml`:

```yaml
# Ao invés de 'latest', use versão específica
image: docker.n8n.io/n8nio/n8n:1.103.2
```

---

## 📖 **Recursos Adicionais**

### **Documentação Oficial**
- [n8n Documentation](https://docs.n8n.io/)
- [Docker Installation Guide](https://docs.n8n.io/hosting/installation/docker/)
- [Environment Variables](https://docs.n8n.io/hosting/configuration/environment-variables/)

### **Comunidade e Suporte**
- [n8n Community Forum](https://community.n8n.io/)
- [GitHub Issues](https://github.com/n8n-io/n8n/issues)
- [Discord Server](https://discord.gg/n8n)

### **Templates e Workflows**
- [n8n Workflow Templates](https://n8n.io/workflows)
- [GitHub Examples](https://github.com/n8n-io/n8n/tree/master/packages/cli/templates)

---

## 🆘 **Suporte**

Se você encontrar problemas:

1. **Verifique os logs**: `docker compose logs -f n8n`
2. **Consulte o troubleshooting** acima
3. **Procure no [Forum da Comunidade](https://community.n8n.io/)**
4. **Abra uma issue** no repositório

---

## 📄 **Licença**

Este projeto é baseado no n8n, que usa a [Sustainable Use License](https://github.com/n8n-io/n8n/blob/master/LICENSE.md).

**Para uso comercial extensivo, considere a [licença Enterprise](https://n8n.io/pricing/) do n8n.**

---

## 🎉 **Próximos Passos**

1. **Configure seu primeiro workflow** - Explore os templates
2. **Conecte suas ferramentas** - APIs, bancos de dados, serviços
3. **Configure backups automáticos** - Use cron jobs ou GitHub Actions
4. **Monitore performance** - Configure alertas para recursos
5. **Scale conforme necessário** - Worker mode para alta demanda

**Happy Automating! 🚀**