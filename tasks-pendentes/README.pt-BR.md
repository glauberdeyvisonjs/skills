# tasks-pendentes

[English](README.md) · **Português** · [SKILL.md](SKILL.md)

Lista suas issues no Linear disponíveis para desenvolvimento ou em andamento, filtradas por bloqueios e ordenadas por prioridade.

Uso: `/tasks-pendentes`.

## Pré-requisitos

Acesso ao Linear, incluindo issues atribuídas, relações de bloqueio e histórico de status.

## Fluxo

1. Seleciona suas issues em Ready for Dev, Doing ou To Fix.
2. Exclui bloqueantes pendentes em In Review ou antes; Preparing for Test ou adiante já não bloqueia o desenvolvimento.
3. Ordena por prioridade, data limite, data de abertura e tempo no status, nessa ordem.

## Resultado e limites

Tabela: Código/Link, Título. Bloqueios desconhecidos são sinalizados. Não altera issues nem inicia implementação.

## Instalação

```bash
npx skills add glauberdeyvisonjs/skills --skill tasks-pendentes -g
```

Ou copie `tasks-pendentes/SKILL.md` para `~/.claude/skills/tasks-pendentes/SKILL.md`.
