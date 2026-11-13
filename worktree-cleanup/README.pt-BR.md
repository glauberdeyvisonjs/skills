# worktree-cleanup

[English](README.md) · **Português** · [SKILL.md](SKILL.md)

Remove o que uma worktree git deixou para trás: os containers, imagens, volumes e redes Docker dela e, se você confirmar, os bancos e chaves de cache dela. Também pode apagar a pasta da worktree. Recursos compartilhados nunca são apagados, como o banco de dev que todas as worktrees usam.

Uso:
- `/worktree-cleanup <caminho|branch|nome>...`: limpa as worktrees informadas.
- `/worktree-cleanup --all --keep a,b`: limpa todas as worktrees, exceto o checkout principal e as mantidas.

## Pré-requisitos

Precisa de Docker e git. A GitHub CLI (`gh`) é opcional e só serve para checar se há PR aberta.

## Fluxo

1. Entra no modo plan e faz três perguntas de sim ou não: limpar os recursos Docker? os bancos e chaves de cache? a pasta?
2. Lê os scripts de setup e os arquivos de compose para saber o que é criado por worktree e o que é compartilhado.
3. Levanta o inventário de cada alvo, sem alterar nada.
4. Pede que você marque os itens exatos a apagar, com uma seleção múltipla por categoria e nada pré-marcado. Recusa apagar uma pasta com trabalho não commitado ou sem push.
5. Remove só os itens aprovados, confere e reporta. A branch é mantida.

## Instalação

```bash
npx skills add glauberdeyvisonjs/skills --skill worktree-cleanup -g
```

Ou copie a pasta `worktree-cleanup/` para `~/.claude/skills/` (Claude Code) e `~/.agents/skills/` (Codex).
