# Migração: Resumo Diário — Cowork → Claude Code

## Mapeamento de conceitos

| Cowork | Claude Code | Observação |
|---|---|---|
| Skill (`resumo-diario:resumo-diario`) | Skill em `.claude/skills/resumo-diario/SKILL.md` | Formato quase idêntico, só muda o frontmatter |
| Scheduled task (9h) | Routine (`claude routine create`) | Equivalente direto, roda na nuvem, tolerância de até 30 min |
| Artifact hospedado (URL própria, atualizável) | Não existe equivalente nativo | Vira arquivo HTML estático que você mesmo hospeda |
| Conector Gmail (MCP pronto do Cowork) | MCP server de Gmail configurado à mão | Precisa instalar/configurar você mesmo |

---

## Passo 1 — Instalar a skill

Crie a pasta e o arquivo (nível de projeto, para funcionar em routines):

```bash
mkdir -p .claude/skills/resumo-diario
```

Salve o conteúdo do arquivo `SKILL.md` (anexo) em `.claude/skills/resumo-diario/SKILL.md`.

Diferença chave em relação ao Cowork: o frontmatter YAML agora é obrigatório:

```yaml
---
name: resumo-diario
description: Gera o resumo diário de notícias (política e economia, Brasil e mundo)...
---
```

Faça commit dessa pasta no repositório — routines só leem skills versionadas em `.claude/skills/`, não as que estão em `~/.claude/skills/` da sua máquina.

Teste local antes de agendar:

```bash
claude -p "/resumo-diario"
```

---

## Passo 2 — Configurar o MCP do Gmail

O Cowork já vinha com um conector Gmail pronto; no Claude Code você precisa apontar para um MCP server de Gmail (comunidade ou próprio):

```bash
claude mcp add --transport stdio gmail -- npx -y <pacote-mcp-gmail>
```

ou, se for um servidor HTTP:

```bash
claude mcp add --transport http gmail https://<seu-endpoint-mcp-gmail>
```

Para a skill funcionar em routines (nuvem), declare o servidor em `.mcp.json` na raiz do repositório (não só localmente com `claude mcp add`, que grava em `~/.claude/mcp.json`, restrito à sua máquina).

Você vai precisar de credenciais OAuth do Google configuradas nesse MCP server — isso é setup manual, não existe "conectar com um clique" como no Cowork.

---

## Passo 3 — Agendar (routine)

```bash
claude routine create \
  --name "resumo-diario-9h" \
  --schedule "0 9 * * 1-5" \
  --prompt "/resumo-diario"
```

- `0 9 * * 1-5` = todo dia útil às 9h (ajuste para `0 9 * * *` se quiser incluir fins de semana, como no Cowork).
- A routine roda na nuvem, sem precisar do computador ligado — igual ao scheduled task do Cowork.
- Alternativa sem routines: cron do seu próprio sistema chamando `claude -p "/resumo-diario" >> resumo.log` — mas aí depende da sua máquina estar ligada no horário.

---

## Passo 4 — Substituir o "artifact" por HTML estático + hospedagem própria

O Cowork mantém uma página com URL fixa que se atualiza sozinha. Isso não existe pronto no Claude Code. Duas opções:

**Opção A — GitHub Pages (recomendada, grátis, URL fixa)**
1. Crie um repositório (ou pasta `docs/` num repo existente) com o arquivo `resumo-diario-template.html` (anexo) renomeado para `index.html`.
2. Ative GitHub Pages apontando para essa pasta/branch.
3. Na skill, instrua Claude a sobrescrever esse `index.html` a cada execução e commitar/dar push.
4. A URL fica fixa (`https://seu-usuario.github.io/seu-repo/`); só o conteúdo muda a cada rodada.

**Opção B — Servidor local**
Sirva a pasta com `python3 -m http.server` ou similar na sua própria máquina/VPS — perde a vantagem de acesso de qualquer lugar, mas não depende de conta no GitHub.

O arquivo `resumo-diario-template.html` anexo já tem a mesma estrutura visual (CSS, seções, badges de tendência) do artifact do Cowork — só falta a skill preencher o conteúdo do dia e, se for pelo GitHub Pages, adicionar ao final do prompt da skill: "sobrescreva `docs/index.html` com o novo conteúdo e rode `git add/commit/push`".

---

## Passo 5 — Enviar e-mail

Depois de configurado o MCP do Gmail (Passo 2), adicione ao final da skill ou da routine a instrução de enviar o e-mail (assunto `📰 Resumo Diário — DD/MM/AAAA`, corpo em HTML) — o texto de instrução é o mesmo já usado no Cowork, só troca o nome da ferramenta MCP conforme o servidor que você configurar.

---

## Resumo do que muda de fato

- **Skill**: copiar e ajustar frontmatter — 5 minutos.
- **Agendamento**: `claude routine create` — 5 minutos.
- **Gmail**: precisa de um MCP server próprio + OAuth — é a parte mais trabalhosa (30 min a algumas horas, dependendo do servidor escolhido).
- **Artifact/HTML**: só existe se você hospedar (GitHub Pages é o caminho mais rápido) — 15-20 minutos.

## Arquivos anexos

- `SKILL.md` — skill pronta para `.claude/skills/resumo-diario/SKILL.md`
- `resumo-diario-template.html` — mesma estrutura visual do artifact do Cowork, pronta para virar `index.html` no GitHub Pages
