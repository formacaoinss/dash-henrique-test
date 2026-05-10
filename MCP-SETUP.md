# Setup do MCP de Google Sheets (avançado)

Este documento ensina a configurar o **MCP do Google Sheets** no seu Claude Code, pra que o Claude consiga **ler e escrever direto na planilha** durante conversas.

> **Você precisa disto?** Não pra usar o dashboard normalmente — todas as operações de dia-a-dia (cadastrar, renovar, mudar status, excluir aluno) já funcionam pelos botões do dash.
>
> **Quando vale configurar:** se você quiser fazer análises customizadas conversando com o Claude (ex: "me diz quantos alunos vencendo em maio", "atualiza o valor de todos os Trimestral pra 600"), edições em massa, ou migrações de dados.
>
> **Tempo estimado:** ~15 minutos.

## O que você vai instalar

1. **Projeto no Google Cloud Console** — pra emitir credenciais OAuth (gratuito)
2. **Servidor MCP `mkummer225/google-sheets-mcp`** — código Node.js que faz a ponte entre o Claude e a API do Google
3. **Configuração no Claude Code** — registrar o servidor no `~/.claude.json`

## Pré-requisitos

- Node.js instalado (`node --version` deve mostrar v18+ — se não tiver, baixa em https://nodejs.org)
- Git instalado
- Claude Code já funcionando

## Passo 1 — Criar credenciais OAuth no Google Cloud

### 1.1 Criar projeto

1. Acesse https://console.cloud.google.com/projectcreate
2. Nome: **claude-sheets** (ou outro de sua escolha)
3. Aguarde criar e selecione no topo da tela

### 1.2 Habilitar APIs

Com o projeto selecionado, abre os 2 links abaixo e clica **"Ativar"** em cada um:

- https://console.cloud.google.com/apis/library/sheets.googleapis.com
- https://console.cloud.google.com/apis/library/drive.googleapis.com

### 1.3 Configurar tela de consentimento

1. Acesse https://console.cloud.google.com/auth/overview
2. **User Type**: External
3. **App name**: `claude-sheets`
4. **User support email**: seu email
5. Em **Audience / Test users**: adiciona seu próprio email Google
6. Salva

### 1.4 Criar credenciais OAuth

1. Acesse https://console.cloud.google.com/apis/credentials
2. **+ CREATE CREDENTIALS** → **OAuth client ID**
3. **Application type**: **Desktop app**
4. **Name**: `claude-sheets-desktop`
5. Cria
6. Na lista de credenciais, clica no ícone de **download JSON** da credencial criada
7. Salva o arquivo em algum lugar fácil (ex: `~/Downloads/credentials.json`)
8. Renomeia o arquivo pra **`gcp-oauth.keys.json`** (atenção que o Windows às vezes adiciona `.json` duplicado — confira)

## Passo 2 — Clonar e buildar o servidor MCP

No terminal:

```bash
# Cria pasta pros servidores MCP (pode ser onde você quiser)
mkdir -p ~/mcp-servers
cd ~/mcp-servers

# Clona o servidor MCP do Google Sheets
git clone https://github.com/mkummer225/google-sheets-mcp.git
cd google-sheets-mcp

# Instala dependências
npm install

# Builda
npm run build
```

Após o build, deve existir o arquivo `dist/index.js`.

## Passo 3 — Mover o `gcp-oauth.keys.json` pra dentro do MCP

O servidor procura o arquivo de credenciais em `dist/gcp-oauth.keys.json`:

**No Linux/Mac:**
```bash
mv ~/Downloads/gcp-oauth.keys.json ~/mcp-servers/google-sheets-mcp/dist/
```

**No Windows (PowerShell):**
```powershell
Move-Item "$HOME\Downloads\gcp-oauth.keys.json" "$HOME\mcp-servers\google-sheets-mcp\dist\"
```

## Passo 4 — Registrar o MCP no Claude Code

### Forma 1 — comando `claude mcp add` (recomendado)

```bash
claude mcp add --scope user google-sheets node ~/mcp-servers/google-sheets-mcp/dist/index.js
```

> No Windows, o caminho fica `C:\Users\SEU_USUARIO\mcp-servers\google-sheets-mcp\dist\index.js` — use o caminho absoluto exato.

### Forma 2 — editar manualmente `~/.claude.json`

Abre o arquivo `~/.claude.json` (ou `%USERPROFILE%\.claude.json` no Windows) e adicione na seção `mcpServers` (cria se não existir):

```json
{
  "mcpServers": {
    "google-sheets": {
      "command": "node",
      "args": ["/caminho/absoluto/para/mcp-servers/google-sheets-mcp/dist/index.js"]
    }
  }
}
```

## Passo 5 — Primeira autenticação

A primeira vez que você usar o MCP, ele vai abrir o navegador pra você autorizar:

1. **Reinicia o Claude Code** (fecha e abre de novo) — MCPs novos só carregam ao iniciar
2. Pede algo pro Claude que use a planilha (ex: "lista as abas da minha planilha")
3. Vai abrir o navegador pra você logar com Google
4. Pode aparecer aviso **"Acesso bloqueado: claude-sheets não concluiu o processo de verificação do Google"** — isso é normal porque é seu próprio app
5. Pra resolver: volta no console.cloud.google.com → **OAuth consent screen** → adiciona seu email em **Test users**
6. Tenta de novo → na tela de aviso clica em **Avançado** → **Acessar claude-sheets (não seguro)** → **Continuar**
7. Autoriza

Após autorizar, o MCP cria um arquivo de token salvo (não precisa repetir esse passo).

## Passo 6 — Permitir as ferramentas no Claude Code (opcional)

Pra evitar o Claude pedir confirmação a cada chamada do MCP, adiciona na seção `permissions.allow` do `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__google-sheets__create_sheet",
      "mcp__google-sheets__create_spreadsheet",
      "mcp__google-sheets__edit_cell",
      "mcp__google-sheets__edit_column",
      "mcp__google-sheets__edit_row",
      "mcp__google-sheets__insert_column",
      "mcp__google-sheets__insert_row",
      "mcp__google-sheets__list_sheets",
      "mcp__google-sheets__read_all_from_sheet",
      "mcp__google-sheets__read_columns",
      "mcp__google-sheets__read_headings",
      "mcp__google-sheets__read_rows",
      "mcp__google-sheets__refresh_auth",
      "mcp__google-sheets__rename_doc",
      "mcp__google-sheets__rename_sheet"
    ]
  }
}
```

## Verificação

Depois de tudo configurado, testa pedindo pro Claude:

```
lista as abas da minha planilha do dashboard
```

Se der certo, ele responde com algo tipo "Sua planilha tem as abas: ALUNOS, _CONFIG, ...".

## Problemas comuns

| Erro | Causa | Solução |
|---|---|---|
| `Acesso bloqueado: claude-sheets não concluiu...` | Email não está em Test users | Adiciona no console.cloud.google.com → OAuth consent screen → Test users |
| `MCP server "google-sheets" not found` no Claude | MCP não registrado ou caminho errado | Refazer passo 4, conferir caminho absoluto |
| `EACCES` ou `permission denied` ao rodar `node` | Falta permissão de execução | `chmod +x dist/index.js` (Linux/Mac) |
| `gcp-oauth.keys.json not found` | Arquivo no lugar errado | Conferir que está em `mcp-servers/google-sheets-mcp/dist/gcp-oauth.keys.json` |
| Token expirou | Refresh token vence | `refresh_auth` no Claude, ou apaga arquivo de token e refaz passo 5 |

## E os outros MCPs?

Se quiser configurar outros MCPs (Notion, Drive, etc), o processo é parecido: cada um tem o próprio `mkdir`, OAuth se aplicável, e registro no `~/.claude.json`. Veja os repos oficiais de cada um pra instruções específicas.
