# Rodar o Falora CRM na sua máquina

Ambiente de desenvolvimento local, sem Docker e sem WSL. Serve para ver as telas e mexer no
produto, **não** para produção.

## O que funciona e o que não

| | |
|---|---|
| Telas, funil, contatos, configuração | funciona |
| Agente de IA | precisa de chave (Anthropic/OpenRouter/OpenAI) |
| Receber mensagem de WhatsApp | **não funciona**: exige servidor com endereço público |

## O que você precisa

- Node 22 ou superior, pnpm 9
- Uma conta no Supabase com um projeto **ativo**
- Um token pessoal do Supabase: https://supabase.com/dashboard/account/tokens

## Passo a passo

```bash
pnpm install

# 1. Extensões do Postgres que o schema exige.
#    Sem isto o baseline falha em "type public.vector does not exist".
curl -s -X POST \
  -H "Authorization: Bearer SEU_TOKEN" -H "Content-Type: application/json" \
  -d '{"query":"create extension if not exists vector with schema public; create extension if not exists citext with schema public; create extension if not exists pgcrypto with schema public; create extension if not exists \"uuid-ossp\" with schema public; create extension if not exists pg_trgm with schema public; create extension if not exists unaccent with schema public; create extension if not exists btree_gin with schema public; create extension if not exists btree_gist with schema public; create extension if not exists hstore with schema public;"}' \
  "https://api.supabase.com/v1/projects/SEU_REF/database/query"

# 2. Chaves + schema + .env.local, tudo de uma vez
python scripts/configurar-local.py SEU_TOKEN [ref_do_projeto]

# 3. Seu usuário
bash scripts/criar-usuario-local.sh seu@email.local

# 4. A organização e o vínculo de admin  <-- o passo que todo mundo esquece
bash scripts/criar-organizacao-local.sh SEU_TOKEN

# 5. Subir
pnpm dev
```

Abra http://localhost:3000.

## Por que o passo 4 existe

Este CRM é multi-empresa e **toda tela é filtrada por organização** (RLS). Um usuário que
existe no Auth mas não pertence a nenhuma organização entra no sistema e vê **o menu
vazio**, sem nenhum erro na tela.

O `install.sh` faz isso no bloco 8; quem configura à mão pula sem perceber. O
`criar-organizacao-local.sh` replica exatamente aquele SQL, incluindo o
`mfa_required = false` explícito, que evita cair num bloqueador de tela cheia logo depois do
onboarding (o padrão da coluna é `true`).

## Marca

O nome e a cor vivem em **dois lugares**, e é preciso mexer nos dois:

| Onde | Alcança |
|---|---|
| Tela `Admin → Marca` (banco) | menus, título da aba, e-mails |
| `APP_NAME` / `APP_LOGO_URL` / `APP_ACCENT_HEX` no `.env.local` | **login, cadastro e onboarding** |

Mexer só na tela deixa a tela de login com o nome antigo. A própria tela avisa isso.

Logo: **PNG ou JPG**, até **512 KB**. SVG é recusado de propósito, porque pode carregar
código.

## Problemas conhecidos

| Sintoma | Causa | Solução |
|---|---|---|
| Menu vazio depois de entrar | usuário sem organização | passo 4 |
| `type public.vector does not exist` | falta extensão | passo 1 |
| `Another next dev server is already running` | servidor órfão | `taskkill /F /PID <pid>` |
| Login com nome antigo | mexeu só no banco | ajuste o `.env.local` e reinicie |

## Segurança

O `.env.local` tem a chave que dá **acesso total ao banco**. Ele está coberto pelo
`.gitignore` (`.env*`) e nunca deve ser commitado nem colado em chat. O token do Supabase
usado nos scripts não é gravado em disco: vive só no processo. Revogue quando terminar.
