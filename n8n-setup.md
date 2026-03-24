# Guia de Instalação: n8n-mcp + Skills no Claude Code

Tudo que foi configurado nesta sessão, documentado para replicar do zero.

---

## Instalação automática via Claude Code

> Mande isso pro Claude Code e ele faz tudo sozinho:
> _"Leia este arquivo e execute a seção de instalação automática"_

Execute os passos abaixo em ordem. Pergunte ao usuário as informações necessárias antes de prosseguir.

**1. Coletar credenciais — pergunte ao usuário:**
- "Qual é a URL da sua instância n8n? (ex: https://meu-n8n.com)"
- "Qual é a sua API Key do n8n? (gerada em Settings → n8n API)"

**2. Salvar credenciais** — use a URL e Key coletadas no passo anterior. Escreva diretamente no arquivo `~/.claude/.env` adicionando estas linhas (com os valores reais):
```
export N8N_API_URL="<URL fornecida pelo usuário>"
export N8N_API_KEY="<Key fornecida pelo usuário>"
```

**3. Registrar o MCP:**
```bash
source ~/.claude/.env
claude mcp add n8n-mcp \
  -e MCP_MODE=stdio \
  -e LOG_LEVEL=error \
  -e DISABLE_CONSOLE_OUTPUT=true \
  -e N8N_API_URL="$N8N_API_URL" \
  -e N8N_API_KEY="$N8N_API_KEY" \
  -- npx n8n-mcp
```

**4. Instalar as skills:**
```bash
git clone --depth=1 https://github.com/czlonkowski/n8n-skills.git /tmp/n8n-skills-tmp
mkdir -p ~/.claude/skills
cp -r /tmp/n8n-skills-tmp/skills/* ~/.claude/skills/
rm -rf /tmp/n8n-skills-tmp
```

**5. Criar o CLAUDE.md do projeto** — pergunte ao usuário: "Qual é a pasta do seu projeto n8n?" Depois crie o arquivo `CLAUDE.md` nessa pasta usando o Write tool, com o conteúdo abaixo. **Substitua `https://sua-instancia.n8n.cloud` pela URL real coletada no Passo 1.**

```
# n8n Workflow Builder

## Instância n8n
- URL: https://sua-instancia.n8n.cloud
- Versão: 2.x.x

## Regras obrigatórias

1. **Sempre buscar documentação do node** antes de configurá-lo (`get_node`)
2. **Sempre validar** o workflow antes de deployar (`validate_workflow`)
3. **Deploy direto** via `n8n_create_workflow` — não gerar JSON para copiar manualmente
4. **Update = deletar + recriar** — usar `n8n_delete_workflow` + `n8n_create_workflow` em vez de `n8n_update_workflow` para evitar conflitos
5. **OAuth primeiro** — usar credenciais OAuth já configuradas no n8n em vez de pedir API keys
6. **Community nodes**: verificar se está instalado antes de usar (`search_nodes` + filtrar por community)

## Processo de criação (4 fases)

### Fase 1 — Planning
- Entender o caso de uso completo
- Salvar resumo em `usecase.md`
- Não buscar nodes ainda

### Fase 2 — Research
- Buscar todos os nodes necessários via MCP
- Documentar parâmetros obrigatórios
- Salvar em `node-research.md`

### Fase 3 — Build
- Ler `usecase.md` + `node-research.md`
- Construir o workflow JSON
- Validar via MCP antes de deployar

### Fase 4 — Deploy & Test
- Deploy via `n8n_create_workflow`
- Verificar com `n8n_health_check`
- Testar com `n8n_test_workflow` se aplicável

## Gestão de contexto
- Usar `/clear` entre fases para não explodir o contexto
- Para workflows complexos (10+ nodes), dividir em sub-workflows

## MCP Tools disponíveis
- `get_node` / `search_nodes` — documentação de nodes
- `list_templates` / `search_templates` — templates prontos
- `validate_workflow` — validação antes do deploy
- `n8n_create_workflow` — criar workflow na instância
- `n8n_list_workflows` — listar workflows existentes
- `n8n_activate_workflow` / `n8n_deactivate_workflow` — lifecycle
- `n8n_executions` — histórico de execuções
- `n8n_health_check` — status da instância
```

**6. Verificar:**
```bash
claude mcp list
# Deve mostrar: n8n-mcp: npx n8n-mcp - ✓ Connected
```

---

## O que é isso?

O **n8n-mcp** é um servidor MCP que permite ao Claude Code criar, editar, validar e deployar workflows n8n diretamente via chat — sem copiar JSON manualmente.

As **skills do n8n** são instruções especializadas que ensinam o Claude a usar o n8n corretamente: sintaxe de expressões, padrões de workflow, configuração de nodes, etc.

Juntos, eles transformam o Claude Code em um builder de automações n8n.

---

## Pré-requisitos

- [Claude Code](https://claude.ai/code) instalado
- [Node.js](https://nodejs.org) v18+ (para rodar o `npx n8n-mcp`)
- Instância n8n rodando (self-hosted ou cloud)
- API Key da instância n8n

---

## Passo 1 — Gerar a API Key do n8n

1. Acesse sua instância n8n
2. Vá em **Settings → n8n API**
3. Clique em **Create an API key**
4. Copie o token gerado (formato JWT)

---

## Passo 2 — Salvar as credenciais

Adicione as credenciais com um único comando (substitua os valores pelos seus):

```bash
cat >> ~/.claude/.env << 'EOF'

# n8n credentials
export N8N_API_URL="https://sua-instancia.n8n.cloud"
export N8N_API_KEY="SEU_TOKEN_AQUI"
EOF
```

> **Importante:** Nunca cole o token diretamente em comandos shell expostos — use variáveis de ambiente. O Claude Code tem hooks de segurança que bloqueiam tokens hardcoded.

---

## Passo 3 — Instalar o n8n-mcp

Com as variáveis exportadas, registre o servidor MCP:

```bash
# Carrega as variáveis primeiro
source ~/.claude/.env

# Registra o servidor MCP
claude mcp add n8n-mcp \
  -e MCP_MODE=stdio \
  -e LOG_LEVEL=error \
  -e DISABLE_CONSOLE_OUTPUT=true \
  -e N8N_API_URL="$N8N_API_URL" \
  -e N8N_API_KEY="$N8N_API_KEY" \
  -- npx n8n-mcp
```

**Verificar instalação:**

```bash
claude mcp list
# Deve mostrar: n8n-mcp: npx n8n-mcp - ✓ Connected
```

> O MCP usa `npx n8n-mcp` — não precisa instalar globalmente. Na primeira execução, o npx faz o download automático.

---

## Passo 4 — Instalar as Skills do n8n

As skills são instruções contextuais que o Claude carrega automaticamente ao trabalhar com n8n. Repositório: [czlonkowski/n8n-skills](https://github.com/czlonkowski/n8n-skills)

**Método 1: Plugin direto (recomendado)**

Execute dentro do Claude Code (não é comando shell):
```
/plugin install czlonkowski/n8n-skills
```

**Método 2: Via marketplace**

Execute dentro do Claude Code:
```
/plugin marketplace add czlonkowski/n8n-skills
/plugin install
```
Selecione "n8n-mcp-skills" da lista.

**Método 3: Manual (shell)**
```bash
git clone https://github.com/czlonkowski/n8n-skills.git
cp -r n8n-skills/skills/* ~/.claude/skills/
```

As 7 skills instaladas e suas funções:

| Skill | O que faz |
|---|---|
| `n8n-expression-syntax` | Valida e corrige expressões `{{ $json.campo }}` |
| `n8n-mcp-tools-expert` | Guia de uso das ferramentas MCP do n8n |
| `n8n-workflow-patterns` | Padrões arquiteturais para workflows complexos |
| `n8n-validation-expert` | Interpreta erros de validação e como corrigir |
| `n8n-node-configuration` | Configuração correta de cada tipo de node |
| `n8n-code-javascript` | Escrever JavaScript em Code nodes |
| `n8n-code-python` | Escrever Python em Code nodes (com limitações) |

> As skills ficam em `~/.claude/skills/` e são carregadas automaticamente pelo Claude Code quando relevantes.

---

## Passo 5 — Criar o CLAUDE.md do projeto

Em cada projeto n8n, crie um `CLAUDE.md` com as instruções do contexto. Exemplo:

```bash
mkdir -p ~/code/projetos/meu-projeto-n8n
cat > ~/code/projetos/meu-projeto-n8n/CLAUDE.md << 'EOF'
# n8n Workflow Builder

## Instância n8n
- URL: https://sua-instancia.n8n.cloud
- Versão: 2.x.x

## Regras obrigatórias

1. **Sempre buscar documentação do node** antes de configurá-lo (`get_node`)
2. **Sempre validar** o workflow antes de deployar (`validate_workflow`)
3. **Deploy direto** via `n8n_create_workflow` — não gerar JSON para copiar manualmente
4. **Update = deletar + recriar** — usar `n8n_delete_workflow` + `n8n_create_workflow` em vez de `n8n_update_workflow` para evitar conflitos
5. **OAuth primeiro** — usar credenciais OAuth já configuradas no n8n em vez de pedir API keys
6. **Community nodes**: verificar se está instalado antes de usar (`search_nodes` + filtrar por community)

## Processo de criação (4 fases)

### Fase 1 — Planning
- Entender o caso de uso completo
- Salvar resumo em `usecase.md`
- Não buscar nodes ainda

### Fase 2 — Research
- Buscar todos os nodes necessários via MCP
- Documentar parâmetros obrigatórios
- Salvar em `node-research.md`

### Fase 3 — Build
- Ler `usecase.md` + `node-research.md`
- Construir o workflow JSON
- Validar via MCP antes de deployar

### Fase 4 — Deploy & Test
- Deploy via `n8n_create_workflow`
- Verificar com `n8n_health_check`
- Testar com `n8n_test_workflow` se aplicável

## Gestão de contexto
- Usar `/clear` entre fases para não explodir o contexto
- Para workflows complexos (10+ nodes), dividir em sub-workflows

## MCP Tools disponíveis
- `get_node` / `search_nodes` — documentação de nodes
- `list_templates` / `search_templates` — templates prontos
- `validate_workflow` — validação antes do deploy
- `n8n_create_workflow` — criar workflow na instância
- `n8n_list_workflows` — listar workflows existentes
- `n8n_activate_workflow` / `n8n_deactivate_workflow` — lifecycle
- `n8n_executions` — histórico de execuções
- `n8n_health_check` — status da instância
EOF
```

---

## Passo 6 — Testar a instalação

Abra o Claude Code **dentro da pasta do projeto n8n** (onde está o `CLAUDE.md`) e mande uma das frases abaixo no chat.

### Teste rápido — verificar conexão

```
Verifique se o n8n está funcionando
```

O Claude vai chamar `n8n_health_check`. Se responder com status OK e a versão do n8n, está tudo certo.

---

### Testes para leigos — copie e cole no chat

**Ver o que já existe no n8n:**
```
Liste todos os workflows que tenho no n8n
```

**Criar algo simples do zero:**
```
Cria um workflow que recebe uma mensagem no webhook e me responde "Recebi!"
```

**Criar uma automação real:**
```
Quero um workflow que todo dia às 9h me manda um e-mail com um resumo do clima
```

**Testar se o Claude entende n8n:**
```
Como eu faria um workflow para salvar dados de um formulário no Google Sheets?
```

---

O Claude vai:
1. Buscar a documentação dos nodes necessários
2. Explicar o que vai construir
3. Construir e validar o workflow
4. Fazer o deploy direto na sua instância n8n

---

## Troubleshooting

**`n8n-mcp` não conecta:**
```bash
# Verifica se as variáveis estão carregadas
echo $N8N_API_URL
echo $N8N_API_KEY

# Se vazio, recarrega:
source ~/.claude/.env
```

**Token bloqueado pelo hook de segurança:**
- Não cole o JWT diretamente em comandos shell
- Use sempre variáveis: `$N8N_API_KEY` no lugar do token

**`npx n8n-mcp` demora na primeira vez:**
- Normal — está baixando o pacote. Aguarde ~30 segundos.

**Workflow criado mas não aparece no n8n:**
- Verifique se a `N8N_API_URL` está correta (sem barra no final)
- Confirme que a API Key tem permissões de escrita

---

## Referências

- [czlonkowski/n8n-mcp](https://github.com/czlonkowski/n8n-mcp) — repositório do MCP server
- [n8n API Docs](https://docs.n8n.io/api/) — documentação oficial da API
- [n8n Community](https://community.n8n.io) — fórum para dúvidas
