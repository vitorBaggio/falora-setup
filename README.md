# Falora CRM — como montar na sua máquina

Este pacote contém tudo que foi feito por cima do CRM original: a correção de um defeito
do instalador, a marca Falora e os scripts que montam um ambiente local.

Você **não precisa de acesso a nenhum repositório privado.** O projeto base é público.

---

## O que tem aqui

| Arquivo | O quê |
|---|---|
| `falora-completo.patch` | as duas mudanças (correção + marca/scripts) |
| `falora-logo.png` | a logo, PNG 1024px |
| `falora-icone.png` | ícone com fundo escuro |
| `RODAR-LOCAL.md` | passo a passo do ambiente local |

---

## Passo 1 — Baixar o projeto base

```bash
git clone --branch v1.19.0 https://github.com/melgarafael/DeskcommCRM.git falora
cd falora
```

A versão **1.19.0** é a mesma em que nosso trabalho foi feito. Usar outra pode fazer o
patch recusar.

## Passo 2 — Aplicar as mudanças

Copie `falora-completo.patch` para dentro da pasta `falora` e rode:

```bash
git am falora-completo.patch
```

Se der erro de "patch does not apply", use a alternativa, que ignora o histórico:

```bash
git apply --3way falora-completo.patch
```

### O que este patch faz

**1. Corrige um defeito que mata a instalação.** Num servidor recém-criado, o comando que
lê tarefas agendadas retorna erro, e o instalador morre no último passo, depois de tudo já
estar quase pronto. É a issue #715 do projeto original. O conserto foi aceito lá, mas
entrou 32 minutos **depois** da versão 1.19.0 sair, então não está nela.

Para conferir que funcionou:

```bash
bash scripts/provar-conserto-crontab.sh
```

Esperado: `TODOS OS CENARIOS PASSARAM`.

**2. Adiciona a marca Falora** (logo, nome e cor verde) e três scripts que montam o
ambiente local sem Docker.

## Passo 3 — Rodar

Siga o `RODAR-LOCAL.md`. Resumo:

```bash
pnpm install
python scripts/configurar-local.py SEU_TOKEN_SUPABASE
bash scripts/criar-usuario-local.sh seu@email.local
bash scripts/criar-organizacao-local.sh SEU_TOKEN_SUPABASE
pnpm dev
```

---

## Três coisas que vão te poupar horas

**1. O menu aparece vazio se o usuário não tiver organização.** Este CRM é multi-empresa e
toda tela é filtrada por empresa. Um usuário sem vínculo entra e vê o menu em branco, **sem
nenhuma mensagem de erro**. É o que o `criar-organizacao-local.sh` resolve. Não pule.

**2. As extensões do Postgres vêm antes do schema.** Sem elas o banco falha com
`type public.vector does not exist`. O `RODAR-LOCAL.md` traz o comando.

**3. O nome do sistema vive em dois lugares.** O banco cobre menus e e-mails, mas a **tela
de login lê do arquivo de configuração**. Mexer só na tela de administração deixa o login
com o nome antigo.

---

## Sobre a licença

O projeto original é MIT: pode usar comercialmente, modificar, colocar marca própria e
vender como serviço, sem pagar nada e sem publicar suas mudanças. A única exigência é
manter o aviso de direitos autorais nos arquivos.
