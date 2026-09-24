# custom-workflow

[English](README.md) · **Português** · [SKILL.md](SKILL.md)

Skill do Claude Code para desenvolver uma ou mais tasks (tickets de um issue tracker ou texto colado)
com um workflow multi-agent planejado: agents desenvolvedores, revisão pelo Codex a cada
fase e revisão geral no final. Tudo passa pela aprovação do usuário antes de começar.

Uso: `/custom-workflow` — opcionalmente seguido das tasks (chaves, URLs ou texto).

## Pré-requisitos

- [Claude Code](https://claude.com/claude-code) com a ferramenta Workflow.
- Skills `grill-me` e `grilling` ([mattpocock/skills](https://github.com/mattpocock/skills)). Se faltarem, a skill oferece instalar; se você recusar, ela aborta.
- [Codex CLI](https://github.com/openai/codex), se for revisar com Codex.
- Opcional: `superpowers:writing-plans`, usado quando o modo Plan não está ativo. MCP do Linear ou do Jira, pra ler tasks de um tracker.

## Fluxo

| # | Etapa | Quem faz | Gate |
|---|-------|----------|------|
| 0 | Checar se `grill-me` e `grilling` estão instalados. Se faltar, oferecer instalação; se recusar, abortar | Agent principal | Instalação |
| 1–2 | Grill: fonte da task, tasks e modelos (ver abaixo) | Agent principal + usuário | Respostas |
| 3 | Ler as issues e explorar o código necessário | Agent principal | — |
| 4 | Montar o Plan: modo Plan se ativo → senão `superpowers:writing-plans` → senão pedir pra trocar pro modo Plan | Agent principal | — |
| 5 | Com múltiplas tasks: ordem sugerida + o que roda em paralelo | Agent principal | — |
| 6 | Apresentar o plano de implementação e o plano do workflow (fase × tracks × devs × revisores × modelos) | Agent principal | **Aprovação** |
| 7 | Um único Workflow: um step por fase do plano + um step de Revisão geral | Workflow | — |
| 8 | Fim de cada step: `log()` com resumo. Falha → para | Workflow | Para se falhar |
| 9 | Resumo consolidado. Se parou: pergunta como seguir e retoma com `resumeFromRunId` | Agent principal | — |
| 10 | Pergunta se quer commit/PR. Só commita depois da aprovação | Agent principal | Aprovação |

## Grill (a cada uso)

| Rodada | Pergunta | Opções | Recomendação |
|--------|----------|--------|--------------|
| 1 | T1 — Fonte da task | Linear (se houver MCP) / Jira (se houver MCP) / Texto (colar) | o que o prompt indicar, senão Texto |
| 2 | T2 — As tasks *(pulada se o prompt já trouxer)* | chaves/URLs ou texto colado | — |
| 1 | Q1 — Modelo dos agents de desenvolvimento | haiku / sonnet / opus / fable | haiku |
| 1 | Q2 — Revisar com Codex? | sim / não | sim |
| 2 | Q3 — Modelo dos agents de revisão | haiku / sonnet / opus / fable | haiku |
| 2 | Q4 — Modelo + effort do Codex na revisão por fase *(se Q2 = sim)* | modelo do `~/.codex/config.toml` ou outro · low / medium / high | gpt-6-astra · high |
| 2 | Q5 — Modelo + effort do Codex na revisão geral *(se Q2 = sim)* | idem | gpt-6-astra · medium (diff maior, evita timeout) |

- Nenhum tracker é assumido: a fonte é sempre perguntada, e só aparecem as fontes com MCP funcionando.
- **Q2 = sim** → o revisor é só um **relay**: roda o Codex e repassa o resultado. Se o Codex falhar ou recusar o modelo, a revisão falha e o usuário é consultado. O relay nunca revisa por conta própria, e não há fallback silencioso pra outro modelo.
- **Q2 = não** → o agent revisor (modelo da Q3) revisa o diff ele mesmo.

## Revisão: por fase + geral

- **Por fase:** pega os problemas cedo, antes de a fase seguinte construir em cima deles.
- **Geral:** pega o que só aparece com tudo junto: integração entre fases e tasks, contratos, nomes, duplicação e critérios de aceite.
- Nos dois níveis: no máximo **3 rodadas de revisão, com 2 correções no meio**. Se a 3ª rodada ainda pedir mudança, o usuário decide como seguir.
- Só P0–P2 bloqueiam. P3 é listado, mas não bloqueia.

```
F1: Dev ─ Review#1 ─(fix)─ Review#2 ─(fix)─ Review#3 ─┐
F2: Dev ─ Review ... (tracks paralelos dentro da fase)  ├─ todas aprovadas
...                                                     ┘
Revisão geral: Review#1 (diff completo) ─(fix)─ #2 ─(fix)─ #3 ─ aprovado → resumo final
```

## Regras

- Paralelismo no mesmo working tree só com arquivos disjuntos. Worktree isolada só se aprovada no plano.
- Os devs não commitam, não rodam migrations e não saem da lista de arquivos do track.
- Timeout do Codex (10 min no Bash) conta como falha, e o usuário é consultado.
- No `/workflows`, cada fase aparece como um step, com os grupos `F<n> Dev · <track>` e `F<n> Review · <track>`.

## Instalação

```bash
npx skills add glauberdeyvisonjs/skills --skill custom-workflow -g
```

Ou copie `custom-workflow/SKILL.md` para `~/.claude/skills/custom-workflow/SKILL.md`.
