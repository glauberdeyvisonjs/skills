# custom-code-review

[English](README.md) · **Português** · [SKILL.md](SKILL.md)

Skill do Claude Code que revisa a PR da branch atual comparando com a spec da issue (Linear, Jira, texto
colado ou outra fonte). Opcionalmente roda os testes automatizados e separa as falhas causadas pela PR das que
já existiam. Pode rodar uma segunda revisão (Codex ou outro subagent Claude) em paralelo. O resultado é um
relatório em tabelas com uma recomendação. É só leitura: nunca edita código, nunca posta no GitHub nem no
tracker e nunca roda migrations.

Uso: `/custom-code-review` na branch da PR.

## Pré-requisitos

- [Claude Code](https://claude.com/claude-code) e [GitHub CLI](https://cli.github.com/) (`gh`) autenticado.
- Opcional: MCP do Linear ou do Jira pra ler a issue, [Codex CLI](https://github.com/openai/codex) pra segunda revisão e uma skill de review instalada (`code-review`, `superpowers:requesting-code-review`, `caveman:caveman-review`…).

## Fluxo

| # | Etapa | Quem faz |
|---|-------|----------|
| 1 | Levantamento: PR, comentários inline, checks de CI, local × remoto, chave da task, modo re-request, skills de review, Codex | Agent principal |
| 2 | Perguntas (AskUserQuestion, 2 rodadas) | Agent principal + usuário |
| 3 | Ler a spec: critérios de aceite, regras de negócio, fora de escopo; comparar com a descrição da PR | Agent principal |
| 4 | Testes: checa se o banco de teste está isolado → roda → classifica as falhas | Agent principal + subagent de testes |
| 5 | Em paralelo: revisor principal (subagent, skill de review opcional) + segundo revisor + testes | Subagents / Codex |
| 6 | Consolida, remove duplicados, confere cada P0–P2 no código e descarta falsos positivos | Agent principal |
| 7 | Relatório + recomendação; salvo em `~/.claude/custom-code-review/<owner>-<repo>/pr-<n>/` | Agent principal |

## Perguntas

| Rodada | Pergunta | Opções | Padrão |
|--------|----------|--------|--------|
| 1 | Q1 — Fonte da spec | Linear / Jira / Texto (colar) / Outro (especificar) | Linear |
| 1 | Q3 — Rodar testes automatizados? | sim / não | sim |
| 1 | Q5 — Nível mínimo que bloqueia | P0 / P1 / P2 / P3 | P2 |
| 1 | Q6 — Segunda revisão em paralelo? | Codex / outro subagent Claude / não | Codex |
| 2 | Q2 — Código/link da task | chave da descrição da PR ou da branch | chave da branch |
| 2 | Q4 — Reprovar por falha de testes? *(se Q3 = sim)* | só falhas da PR / qualquer falha / não | só da PR |
| 2 | Q7 — Skill de review *(se houver 2+)* | skills instaladas / nenhuma | — |
| 2 | Q8 — Modelo do segundo revisor *(Codex: modelo + effort)* | — | modelo do `config.toml` · high |

## Re-review

Detectada automaticamente (reviews ou comentários no GitHub, ou um relatório local anterior). São três passadas:

1. Situação de cada comentário anterior: corrigido / parcial / não corrigido / regrediu.
2. Diff desde o último commit revisado: regressões causadas pelas correções.
3. Revisão completa do diff inteiro de novo, procurando o que as revisões anteriores deixaram passar.

## Testes

- **Checagem do banco antes de qualquer teste:** só roda se o banco de teste for SQLite ou um banco exclusivo de teste, diferente do de dev. Se não der pra confirmar, a skill não roda os testes e pergunta.
- Classes de falha: `PR` (teste criado ou alterado pela PR, ou que passa na base e falha na branch), `pre-existing` (já existia), `flaky` (instável), `unclassified` (não classificada). A comparação com a base usa um `git worktree` separado; nunca usa `stash` nem `checkout` no seu working tree.

## Severidade e recomendação

| Sev | Significado |
|-----|-------------|
| P0 | perda de dados, falha de segurança, queda em produção, build quebrado |
| P1 | bug em fluxo principal, critério de aceite não atendido, regra de negócio violada, regressão |
| P2 | edge case, validação faltando, comportamento novo sem teste, performance, mudança fora do escopo arriscada |
| P3 | legibilidade, nomes, estilo, sugestão |

Regra fixa, vale a primeira que bater: qualquer item bloqueante → **Reprovar**; spec ambígua ou critério que não dá pra verificar → **Comentar sem reprovar**; só itens abaixo do nível de bloqueio → **Aprovar com sugestões**; nenhum achado → **Aprovar**. Outras sugestões vêm depois do resultado da regra, nunca no lugar dele.

## Relatório

Achados · comentários anteriores (re-review) · critérios de aceite · testes + CI · regras de negócio novas · perguntas sobre a spec · achados descartados · recomendação. Escrito no idioma da PR.

## Instalação

```bash
npx skills add glauberdeyvisonjs/skills --skill custom-code-review -g
```

Ou copie `custom-code-review/SKILL.md` para `~/.claude/skills/custom-code-review/SKILL.md`.
