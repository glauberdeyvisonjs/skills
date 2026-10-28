# before-pr-protocol

[English](README.md) · **Português** · [SKILL.md](SKILL.md)

Roda os testes automatizados do repositório atual antes de abrir uma PR e aguarda os resultados.

Uso: `/before-pr-protocol`.

## Pré-requisitos

Checkout local com dependências de teste e comandos documentados. Testes com banco ou serviços exigem recursos de teste comprovadamente isolados.

## Fluxo

1. Descobre os comandos reais de teste do repositório atual.
2. Cobre todas as suítes configuradas: unitárias, integração, end-to-end e de navegador.
3. Inspeciona setup e hooks para efeitos sobre bancos e serviços externos antes da execução.
4. Roda as suítes disponíveis e seguras, aguarda o término e informa cobertura ausente ou bloqueada.

## Resultado e limites

Tabela: Suíte, Comando, Resultado, Detalhes. Testa apenas o repositório e a branch atuais. Migrations, resets e seeds exigem pedido explícito do comando exato, inclusive quando chamados por scripts. Não corrige código, cria PR ou faz push.

## Instalação

```bash
npx skills add glauberdeyvisonjs/skills --skill before-pr-protocol -g
```

Ou copie `before-pr-protocol/SKILL.md` para `~/.claude/skills/before-pr-protocol/SKILL.md`.
