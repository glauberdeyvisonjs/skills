# fix-prs

[English](README.md) · **Português** · [SKILL.md](SKILL.md)

Lista suas PRs do repositório atual com feedback pendente e identifica PRs corrigidas em que faltou solicitar re-request.

Uso: `/fix-prs`.

## Pré-requisitos

GitHub CLI (`gh`) autenticado com acesso ao repositório atual. Linear é opcional para verificar links de issues.

## Fluxo

1. Busca PRs abertas do repositório atual criadas por você ou atribuídas a você, sem duplicar resultados.
2. Confere alterações solicitadas, comentários acionáveis, commits e histórico de pedidos de revisão.
3. Lista feedback que ainda exige correção; exclui PRs corrigidas já aguardando nova revisão.
4. Destaca PRs corrigidas sem novo pedido de revisão; não presume que qualquer commit posterior corrigiu tudo.

## Resultado e limites

Tabela: Código, Título, Link da Issue no Linear, Branch. PRs mais antigas primeiro, com seção separada de Re-request pendente. Não altera código nem envia re-requests.

## Instalação

```bash
npx skills add glauberdeyvisonjs/skills --skill fix-prs -g
```

Ou copie `fix-prs/SKILL.md` para `~/.claude/skills/fix-prs/SKILL.md`.
