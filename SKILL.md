---
name: resumo-diario
description: Gera o resumo diário de notícias (política e economia, Brasil e mundo) cruzando busca web com newsletters do Gmail, incluindo bloco de câmbio BRL/USD e BRL/EUR com tendência de 30 dias. Use quando o usuário disser "resumo do dia", "resumo de hoje", "notícias de hoje", "me atualiza" ou variação equivalente.
---

# Skill: Resumo Diário de Notícias

## Objetivo

Gerar um resumo diário consolidado das principais notícias das últimas 24h,
cruzando fontes abertas da web com newsletters recebidas no Gmail.

## Fontes

### Web (buscar sempre)
- Agência Brasil (agenciabrasil.ebc.com.br)
- G1 / Globo
- Folha de S.Paulo (manchetes abertas)
- Estadão (manchetes abertas)
- Poder360
- Reuters Brasil
- BBC Brasil
- El País Brasil
- Nexo Jornal (parte aberta)
- Piauí (manchetes abertas)

### Gmail (buscar sempre)
- Remetente: `info@nexojornal.com.br` ou `noreply@newslettersrevistapiaui.com.br`
- Buscar e-mails das últimas 24h desse remetente
- Extrair os principais tópicos/links mencionados

## Processo de execução

### Passo 1 — Buscar no Gmail
Buscar threads do Gmail com remetente `info@nexojornal.com.br` ou `noreply@newslettersrevistapiaui.com.br` das últimas 24h.
Extrair títulos de matérias e temas cobertos nas newsletters recebidas.

### Passo 2 — Buscar na web
Executar buscas cobrindo:
1. Política Brasil — governo, Congresso, STF, eleições 2026
2. Economia Brasil — mercados, Selic, inflação, câmbio, fiscal
3. Internacional — conflitos, geopolítica, economia global
4. Outros destaques — tecnologia, saúde, meio ambiente (se relevante)

Use datas explícitas nas queries (ex: "notícias Brasil política [data de hoje]").
Priorize fontes das últimas 24h.

**Aprofundamento obrigatório**: não se contente com o snippet da busca. Para cada item que for
entrar no resumo, faça fetch em pelo menos 1–2 matérias completas (quando a URL estiver
disponível) para extrair: números exatos, nomes de pessoas/instituições envolvidas, citações
diretas relevantes, datas e prazos, e o encadeamento causa→fato→consequência. Rode buscas de
acompanhamento (follow-up queries) para reunir contexto histórico/antecedentes de cada item
principal (ex.: "quem é X", "o que motivou Y", "cronologia do caso Z") antes de redigir.

### Passo 3 — Câmbio (BRL/USD e BRL/EUR)
Buscar a cotação do dia e o histórico de 30 dias. Fonte primária recomendada: API PTAX do Banco
Central (`https://olinda.bcb.gov.br/olinda/servico/PTAX/versao/v1/odata/...`). Se o ambiente
bloquear chamadas diretas à API (comum em sandboxes com allowlist de rede), usar como fallback
uma busca web + fetch em agregador de mercado (ex.: valordolar.com.br, investing.com) e registrar
essa ressalva no resumo final. Calcular a tendência com base na variação dos últimos 7 dias e na
posição frente à média de 30 dias:
- **VALORIZANDO** (do real): moeda estrangeira em queda consistente frente ao BRL
- **DEPRECIANDO** (do real): moeda estrangeira em alta consistente frente ao BRL
- **ESTÁVEL**: oscilação lateral sem tendência clara

### Passo 4 — Consolidar e redigir

Montar o resumo no formato abaixo. Cruzar informações de web + Gmail quando houver sobreposição de temas.
Priorize profundidade sobre quantidade: é preferível cobrir menos itens com mais contexto do que
listar manchetes soltas.

## Formato de saída obrigatório

```
## 📰 RESUMO DIÁRIO — DD/MM/AAAA

---

### 📍 BRASIL — POLÍTICA
[5–7 itens. Cada item: negrito com título + 4–6 linhas cobrindo: o fato central com números/nomes
exatos, o contexto/antecedentes (o que levou a isso), os atores envolvidos e suas posições, e o
próximo passo esperado (prazo, votação, decisão pendente)]

---

### 📍 BRASIL — ECONOMIA
[5–7 itens. Mesmo padrão de profundidade — incluir sempre os números oficiais (índices, percentuais,
valores em R$/US$) e a fonte primária do dado (IBGE, BCB, Boletim Focus, B3 etc.)]

---

### 💱 CÂMBIO
[USD/BRL e EUR/BRL: cotação do dia, variação diária, faixa e média dos últimos 30 dias, variação
em 7 dias, e tendência classificada (VALORIZANDO/DEPRECIANDO/ESTÁVEL). Citar a fonte usada e, se
for fallback (não-PTAX), registrar a ressalva.]

---

### 📍 MUNDO
[5–7 itens. Mesmo padrão — explicar o histórico do conflito/evento para quem não acompanhou os dias
anteriores, não assumir conhecimento prévio do leitor]

---

### 📍 OUTROS DESTAQUES
[Tecnologia, saúde, clima, cultura — até 4–5 itens, com 2–4 linhas cada, mesmo nível de detalhe factual]

---

### 📍 ANÁLISE RÁPIDA
[4–6 linhas: tensões narrativas principais, ângulos divergentes entre fontes, conexões entre os
itens cobertos (ex.: como a economia afeta a política, como um item internacional impacta o Brasil),
e o que monitorar nos próximos dias com data/evento específico quando houver]

---
📬 Fontes consultadas: [listar web + Gmail quando usado, com nome do veículo]
```

## Regras de redação

- Sem opinião editorial própria
- Fato primeiro, contexto depois — nunca o inverso
- Se houver versões conflitantes do mesmo fato, mencionar brevemente as duas versões e a fonte de cada uma
- Parágrafos de 4–6 linhas por item — priorizar densidade informativa (números, nomes, datas, citações) sobre generalidades
- Cada item deve responder: o quê, quem, quando, por quê aconteceu e o que vem a seguir — não apenas o quê
- Linguagem: português brasileiro; termos técnicos econômicos/políticos em português
- Não repetir o mesmo fato em blocos diferentes — se for relevante para dois blocos, aprofundar ângulos distintos em cada um

## Saída adicional (opcional, requer configuração própria no Claude Code)

Se desejar reproduzir o fluxo completo do Cowork (atualizar uma página HTML e enviar por e-mail),
adicione ao final da execução:

1. Escrever o resumo em HTML autocontido (ver `resumo-diario.html` de referência) sobrescrevendo
   o arquivo do repositório/pasta hospedada (GitHub Pages, servidor local, etc.).
2. Enviar por e-mail usando um MCP server de Gmail configurado no Claude Code (`claude mcp add`),
   com assunto `📰 Resumo Diário — DD/MM/AAAA` e corpo em HTML.

Esses dois passos dependem de infraestrutura própria do usuário (hospedagem + MCP de Gmail) — não
existem nativamente no Claude Code como existem no Cowork (artifact hospedado + conector Gmail
prontos). Ver guia de migração para detalhes.
