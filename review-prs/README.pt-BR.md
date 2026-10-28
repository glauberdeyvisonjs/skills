# review-prs

[English](README.md) · **Português** · [SKILL.md](SKILL.md)

Lista PRs do repositório atual pendentes da sua revisão ou re-review, respeitando qual desenvolvedor já assumiu a revisão.

Uso: `/review-prs`.

## Pré-requisitos

GitHub CLI (`gh`) autenticado com acesso ao repositório atual. Linear é opcional para verificar links de issues.

## Fluxo

1. Busca PRs abertas do repositório atual que solicitam sua revisão.
2. Confere reviewers, comentários, reviews e histórico de re-request.
3. Inclui pedidos só para você, pedidos compartilhados ainda não assumidos e seus re-requests após correções.
4. Exclui pedidos compartilhados já assumidos por outro desenvolvedor; sinaliza históricos ambíguos separadamente.

## Resultado e limites

Tabela: Código, Título, Link da Issue no Linear, Branch, Autor. PRs mais antigas primeiro. Lista a fila sem executar ou publicar reviews.

## Instalação

```bash
npx skills add glauberdeyvisonjs/skills --skill review-prs -g
```

Ou copie `review-prs/SKILL.md` para `~/.claude/skills/review-prs/SKILL.md`.
