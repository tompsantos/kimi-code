# Status da localização pt-BR

Última atualização: 2026-08-07

## Visão geral

| Indicador | Estado |
| --- | --- |
| Páginas físicas por locale | 26 |
| Páginas traduzidas | 6 |
| Em revisão | 0 |
| Pendentes | 20 |
| Divergências críticas conhecidas | 0 |
| Divergências editoriais rastreadas | 1 |
| Glossário inicial | pronto |
| Plano de manutenção | pronto |
| Proposta upstream | aberta |

Progresso de tradução: **6 / 26 páginas**

```text
[██████░░░░░░░░░░░░░░░░░░░░] 23%
```

## Estados

- ✅ sincronizada
- 🔵 em tradução
- 🟡 atualização pendente
- 🔴 divergência crítica
- ⚪ pendente
- ⏸ aguardando decisão upstream

## Guias

| Página | Estado |
| --- | --- |
| `docs/pt-br/index.md` | ⚪ |
| `guides/getting-started.md` | ✅ |
| `guides/goals.md` | ⚪ |
| `guides/ides.md` | ⚪ |
| `guides/interaction.md` | ✅ |
| `guides/migration.md` | ⚪ |
| `guides/sessions.md` | ✅ |
| `guides/use-cases.md` | ⚪ |

## Personalização

| Página | Estado |
| --- | --- |
| `customization/agents.md` | ⚪ |
| `customization/datasource.md` | ⏸ |
| `customization/hooks.md` | ⚪ |
| `customization/mcp.md` | ✅ |
| `customization/plugins.md` | ⚪ |
| `customization/skills.md` | ⚪ |
| `customization/themes.md` | ⚪ |

`datasource.md` existe na documentação fonte, mas sua inclusão na navegação pública ainda merece alinhamento com o upstream.

## Configuração

| Página | Estado |
| --- | --- |
| `configuration/config-files.md` | ✅ |
| `configuration/data-locations.md` | ⚪ |
| `configuration/env-vars.md` | ⚪ |
| `configuration/overrides.md` | ⚪ |
| `configuration/providers.md` | ✅ |

## Referência

| Página | Estado |
| --- | --- |
| `reference/keyboard.md` | ⚪ |
| `reference/kimi-acp.md` | ⚪ |
| `reference/kimi-command.md` | ⚪ |
| `reference/slash-commands.md` | ⚪ |
| `reference/tools.md` | ⚪ |

## Notas de versão

| Página | Estado |
| --- | --- |
| `release-notes/changelog.md` | ⏸ |

A política do changelog ainda será definida. A recomendação atual é traduzir novas versões a partir da eventual adoção oficial do locale e tratar o histórico anterior gradualmente.

## Observações de sincronização

### `guides/getting-started.md`

A página inglesa canônica ainda contém uma frase introdutória que descreve a CLI apenas como distribuída via npm e executada no Node.js. A própria seção de instalação da mesma página documenta duas opções atuais: binário independente pelo script oficial e pacote npm.

A versão pt-BR usa a formulação tecnicamente coerente “binário independente ou pacote npm”, sem adicionar comportamento novo. A correção equivalente no inglês está proposta em `MoonshotAI/kimi-code#2664`. Esta diferença é rastreada como divergência editorial, não como divergência de comportamento.

## Prioridade atual

1. ✅ publicar a estrutura comunitária
2. ✅ registrar o glossário inicial
3. ✅ publicar a página piloto de sessões
4. ✅ traduzir `guides/getting-started.md`
5. ✅ traduzir `guides/interaction.md`
6. ✅ traduzir `customization/mcp.md`
7. ✅ traduzir `configuration/config-files.md`
8. ✅ traduzir `configuration/providers.md`
9. ⚪ traduzir `configuration/env-vars.md`
10. ⚪ traduzir `configuration/overrides.md`
11. ⚪ convidar revisores brasileiros
12. ⚪ consolidar um primeiro lote para nova apresentação ao upstream

## Upstream

- Proposta de localização: `MoonshotAI/kimi-code#2685`
- Primeira contribuição documental: `MoonshotAI/kimi-code#2664`

Este arquivo acompanha o trabalho comunitário e não representa status oficial da MoonshotAI.
