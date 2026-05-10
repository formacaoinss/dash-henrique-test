# CLAUDE.md — Instruções pro Claude

> Este arquivo é lido automaticamente pelo Claude Code quando aberto neste repositório. Ele orienta o Claude a guiar o setup do dashboard pro novo dono.

## Contexto do projeto

Este é um **dashboard CRM pra personal trainer / consultor online**. Ele lê dados de uma planilha Google Sheets e mostra MRR, receita, renovações, alunos ativos, etc. Também permite cadastrar/renovar/excluir alunos direto pelo dash.

**Stack:**
- HTML + CSS + JS vanilla (sem build, sem framework)
- Google Sheets como banco de dados
- Google Apps Script como API de escrita (no `apps-script.gs`)
- Hospedagem: Vercel (grátis)
- Bibliotecas: PapaParse + Chart.js (via CDN)

**Arquivos:**
- `index.html` — dashboard completo (HTML + CSS + JS num só arquivo)
- `config.js` — configuração (URLs que o usuário precisa preencher)
- `apps-script.gs` — código que vai dentro do Apps Script da planilha
- `README.md` — visão geral + roadmap
- `HANDOFF.md` — passo-a-passo detalhado de setup
- `MCP-SETUP.md` — setup opcional do MCP de Google Sheets (avançado)
- `CLAUDE.md` — este arquivo

## Comportamento esperado do Claude

Quando o usuário abrir este projeto pela primeira vez, **assuma que ele acabou de receber o repositório** e ainda não fez setup. Comece perguntando:

> "Você já fez o setup deste dashboard antes? Posso te guiar do zero — em ~30 min você está rodando na sua conta. Quer começar?"

Se ele aceitar, **siga rigorosamente** o checklist abaixo, **um passo por vez**. Não despeje tudo de uma vez. Espera ele confirmar cada passo antes de seguir.

## Checklist de setup — siga em ordem

Cada passo tem um **critério de OK**. Só siga pro próximo quando ele confirmar.

---

### Passo 1 — Verificar pré-requisitos

Pergunte se ele tem:
- [ ] Conta Google (pra Sheets + Apps Script)
- [ ] Conta GitHub
- [ ] Conta Vercel (vai criar no passo 7 se não tiver)

✅ **OK quando:** ele confirma que tem as 3 (ou só Google + GitHub se Vercel for criar agora).

---

### Passo 2 — Verificar se ele já tem a planilha

Pergunte: **"Você recebeu uma planilha Google Sheets transferida? Ou vai criar do zero?"**

#### Caso A — Recebeu planilha transferida
- Confirma que ele aceitou a transferência (email do Google)
- Pede a URL da planilha dele
- Anota mentalmente: ele vai precisar **republicar o Apps Script** (passo 4)
- Pula pro passo 3

#### Caso B — Vai criar do zero
- Manda ele acessar https://sheets.new
- Renomear pra "Garbo CRM" (ou nome livre)
- Renomear primeira aba pra **`ALUNOS`** (case-sensitive, exato)
- Colar na linha 1 estes cabeçalhos (uma coluna cada):
  ```
  ID, NOME, DATA_INICIO, PLANO, DATA_INICIO_CICLO, VENCIMENTO, VALOR, FORMA_PGTO, STATUS, ONBOARDING_OK, RENOVOU_VEZES, UPSELL, ORIGEM, CONGELADO_ATE, MOTIVO_SAIDA, ULTIMA_INTERACAO, OBSERVACOES
  ```
- Pede a URL da planilha pra ele

✅ **OK quando:** ele tem a aba `ALUNOS` com cabeçalhos corretos.

---

### Passo 3 — Publicar CSV

Pede pra ele fazer:
1. Na planilha: **Arquivo → Compartilhar → Publicar na web**
2. Em "Documento inteiro" troca pra **`ALUNOS`**
3. Em formato troca pra **Valores separados por vírgula (.csv)**
4. **Publicar** → confirma → **copia a URL** (começa com `https://docs.google.com/spreadsheets/d/e/2PACX-...`)
5. Manda essa URL pra você

> **Se ele já tinha planilha publicada antes:** confirma que **republicou na conta dele**. URLs antigas de outras contas param de funcionar quando muda dono.

✅ **OK quando:** ele te manda a URL do CSV. Você testa internamente fazendo `curl` na URL — deve retornar texto com cabeçalhos `ID,NOME,DATA_INICIO,...`

---

### Passo 4 — Configurar Apps Script

#### Caso A — Planilha transferida (Apps Script já existe nela)

A planilha veio com o Apps Script dentro, mas autorizado pela conta antiga. Ele precisa **republicar pela conta dele**:

1. Abre a planilha
2. **Extensões → Apps Script**
3. **Implantar → Gerenciar implantações**
4. Ícone de **lápis ✏️** na implantação ativa
5. **Executar como**: confirma que está **"Eu"** (a conta dele)
6. **Versão**: troca pra **"Nova versão"**
7. **Implantar** → vai pedir reautorização → autoriza com a conta dele (Avançado → Acessar → Permitir)
8. **Copia a URL nova** (a URL pode ter mudado!)
9. Testa: abre a URL no navegador — deve responder `{"ok":true,"message":"Garbo CRM API. Use POST."}`

#### Caso B — Planilha criada do zero (Apps Script ainda não existe)

1. Na planilha: **Extensões → Apps Script**
2. Apaga `function myFunction() {}` que vier por padrão
3. Cola o conteúdo do arquivo **`apps-script.gs`** deste repo (pega via `Read` tool ou pede pra ele copiar do GitHub)
4. **Ctrl+S** → nome do projeto: **GarboCRM**
5. **Implantar → Nova implantação**
   - Engrenagem ⚙️ → **App da Web**
   - Descrição: `API GarboCRM`
   - Executar como: **Eu**
   - Quem pode acessar: **Qualquer pessoa** ⚠️ (sem isso o dash não consegue chamar)
6. **Implantar** → autoriza com a conta dele (Avançado → Acessar → Permitir)
7. **Copia a URL** que aparece (termina em `/exec`)
8. Testa: abre a URL — deve responder `{"ok":true,...}`

✅ **OK quando:** ele te manda a URL `/exec` e você confirma com `curl` que está respondendo o JSON esperado.

---

### Passo 5 — Editar `config.js`

Use a tool **Edit** pra substituir as URLs no `config.js` com:
- `CSV_URL`: a do passo 3
- `APPS_SCRIPT_URL`: a do passo 4
- `SHEET_URL`: URL da planilha dele (até `/edit`, sem o `#gid=...`)

Opcional: também pode trocar `BRAND_NAME` se ele quiser personalizar.

Depois faça commit e push:
```bash
git add config.js
git commit -m "config: URLs da minha conta"
git push origin main
```

✅ **OK quando:** o `config.js` está commitado no GitHub dele com as 3 URLs corretas.

---

### Passo 6 — Forkar (se ainda não tiver feito)

Se o repo ainda for o original `formacaoinss/dash-henrique-test`, ele precisa **forkar pra conta dele** antes do passo 5 funcionar:

1. https://github.com/formacaoinss/dash-henrique-test
2. Botão **Fork** (canto superior direito)
3. Owner: a conta dele

Depois ele clona o **fork dele** localmente:
```bash
git clone https://github.com/SEU_USUARIO/dash-henrique-test.git
cd dash-henrique-test
```

> **Se ele clonou direto o original sem forkar**, ele não consegue dar push. Tem que forkar e atualizar o remote:
> ```bash
> git remote set-url origin https://github.com/SEU_USUARIO/dash-henrique-test.git
> ```

✅ **OK quando:** o repo origin aponta pro GitHub dele e push funciona.

---

### Passo 7 — Deploy no Vercel

1. Acessa https://vercel.com/signup
2. **Continue with GitHub** → autoriza
3. **Add New → Project**
4. Importa o repo `dash-henrique-test` (do fork dele)
5. Framework Preset: **Other**
6. Build / Output: deixa em branco
7. **Deploy**

Em ~30s ele recebe URL `https://NOME-PROJETO.vercel.app`. Cada `git push` daqui pra frente redeploya automático.

✅ **OK quando:** ele abre a URL do Vercel e vê o dashboard renderizado com seus alunos.

---

### Passo 8 — Setup avançado: MCP de Google Sheets (opcional)

Pergunte ao usuário:

> "Quer também configurar o MCP do Google Sheets? Isso permite que eu (Claude) leia e escreva direto na sua planilha durante conversas — útil pra análises customizadas, edições em massa, etc. Setup adicional de ~15 min. Posso te guiar?"

**Se ele topar:**
- Abra `MCP-SETUP.md` deste repo e siga as instruções **rigorosamente**, um passo por vez
- O setup envolve: criar projeto Google Cloud, habilitar Sheets/Drive APIs, criar OAuth credentials, clonar `mkummer225/google-sheets-mcp`, build, registrar via `claude mcp add`, primeira autenticação no browser
- **Atenção crítica:** o `gcp-oauth.keys.json` baixado pelo Google Chrome às vezes vira `gcp-oauth.keys.json.json` (extensão duplicada). Confira/renomeie antes de mover.
- **Atenção crítica:** o Claude Code precisa ser **reiniciado** depois de registrar o MCP — MCPs novos só carregam ao iniciar.
- Após autenticar pela primeira vez, considere também configurar `~/.claude/settings.json` com `permissions.allow` pras ferramentas do MCP, pra evitar prompts a cada chamada (o MCP-SETUP.md mostra como).

**Se ele recusar / não quiser agora:**
- Avise que pode voltar e configurar quando quiser, e siga pro passo 9 (teste end-to-end).

✅ **OK quando:** ele consegue rodar no Claude um comando tipo "lista as abas da minha planilha" e ver resposta correta.

---

### Passo 9 — Teste end-to-end

Pede pra ele testar as 4 ações principais (na ordem):

1. **Cadastrar:** botão "+ Novo aluno" → preenche um teste → confirma → toast verde + aluno aparece na planilha
2. **Renovar:** clica num aluno → painel lateral → "Renovar" → preenche → confirma → `RENOVOU_VEZES` incrementa
3. **Mudar status:** clica num aluno → "Mudar status" → escolhe outro → confirma → status muda na planilha
4. **Excluir:** clica no aluno teste do passo 1 → "Excluir" → digita o nome pra confirmar → linha some

✅ **OK quando:** as 4 ações funcionam.

---

## Problemas comuns + soluções

| Sintoma | Causa provável | Solução |
|---|---|---|
| Botões "Novo aluno", "Renovar", etc. ficam cinza | `APPS_SCRIPT_URL` vazio em `config.js` | Confirmar passo 5 |
| `Erro: HTTP 401` ou "permission denied" ao salvar | Apps Script não autorizado pela conta dele | Refazer passo 4 |
| Cadastrei aluno mas não aparece no dash | Cache de 5min do Google | Clicar "Atualizar" no header (já tem cache-buster) |
| Dash em branco / "Erro: HTTP 401" no fetch CSV | `CSV_URL` errado ou planilha não publicada | Refazer passo 3 |
| Erro "Acao desconhecida: delete" | Apps Script desatualizado | Republicar com Nova versão (passo 4 do "Atualizar Apps Script") |
| Botão Excluir não libera mesmo digitando nome | Bug antigo (já corrigido) — pull dos updates |  `git pull` |

## Atualizar o código depois do setup

### Mudou HTML/CSS/JS?
Commit + push → Vercel redeploya em 30s. Pronto.

### Mudou `apps-script.gs`?
**Não basta dar push.** O código novo só vale depois de republicar manualmente:
1. Cola o código novo no editor do Apps Script
2. **Ctrl+S**
3. **Implantar → Gerenciar implantações** → ✏️ → **Nova versão** → Implantar
4. URL não muda

## Roadmap de evoluções

Veja `README.md` na seção "Roadmap" — tem várias sugestões de melhorias (editar aluno, histórico de pagamentos, notificações WhatsApp, etc).

## Princípios pro Claude seguir

1. **Um passo por vez** — não despeje os 8 passos de uma vez. Espera confirmação a cada um.
2. **Validar antes de seguir** — se passo 3 pede uma URL, testa a URL com `curl` antes de avançar pro passo 4.
3. **Seja paciente com erros do Google** — autorização do Apps Script é confusa pra usuários não-técnicos. Aviso "Google não verificou este app" é normal — explica que é seguro porque é o próprio script dele.
4. **Não invente URLs** — se ele não te passou uma URL, peça antes de tentar editar `config.js`.
5. **Não use comandos destrutivos** — não rode `rm`, `git reset --hard`, etc. sem permissão explícita.
6. **Faça commits pequenos com mensagens claras** — cada mudança de configuração = 1 commit.
