# Glossário de localização pt-BR

Este glossário registra as decisões terminológicas do projeto comunitário de localização brasileira da documentação do Kimi Code.

## Regras gerais

- Use português brasileiro técnico, direto e natural.
- Preserve nomes oficiais, comandos, flags, caminhos, variáveis, campos e identificadores.
- Explique jargões na primeira ocorrência quando isso ajudar leitores menos técnicos.
- Use sentence case em títulos e subtítulos.
- Evite traduções literais que se afastem do vocabulário comum de desenvolvimento de software no Brasil.

## Nomes e identificadores que devem permanecer inalterados

`Kimi Code CLI`, `Kimi Code for VS Code`, `Kimi Code platform`, `Kimi Open Platform`, `Kimi Datasource`, `Model Context Protocol`, `MCP`, `Agent Skills`, `OAuth`, `API`, `JSON`, `JSONL`, `YAML`, `TOML`, `TypeScript`, `Node.js`, `npm`, `pnpm`, `macOS`, `GitHub`, `VS Code`, `kimi`, `PATH`, `KIMI_CODE_HOME`, `SYSTEM.md`, `AGENTS.md`, `SKILL.md`, `config.toml`, `mcp.json`.

## Termos adotados

| English | pt-BR | Observação |
| --- | --- | --- |
| agent | agente | termo genérico |
| main agent | agente principal | |
| subagent | subagente | sem hífen |
| custom agent | agente personalizado | |
| built-in agent | agente integrado | |
| prompt | prompt | manter em inglês |
| system prompt | prompt do sistema | |
| session | sessão | |
| context | contexto | |
| context window | janela de contexto | |
| context compression | compactação do contexto | preferido a “compressão” |
| token | token | |
| fork a session | criar um fork da sessão | preservar “fork” |
| tool | ferramenta | |
| tool call | chamada de ferramenta | |
| built-in tool | ferramenta integrada | |
| approval request | solicitação de aprovação | |
| permission rule | regra de permissão | |
| allowlist | lista de permissões | |
| denylist / blocklist | lista de bloqueio | |
| TUI | TUI | primeira ocorrência: “interface de usuário no terminal” |
| CLI | CLI | primeira ocorrência: “interface de linha de comando” |
| shell | shell | explicar na primeira ocorrência quando necessário |
| input box | caixa de entrada | |
| conversation view | área da conversa | |
| status bar | barra de status | |
| streaming output | saída em streaming | |
| working directory | diretório de trabalho | |
| background task | tarefa em segundo plano | |
| Plan mode | modo Plan | nome do modo preservado |
| YOLO mode | modo YOLO | nome do modo preservado |
| Auto mode | modo Auto | nome do modo preservado |
| Thinking mode | modo Thinking | nome do modo preservado |
| Shell mode | modo shell | |
| slash command | comando de barra | |
| skill | Skill / habilidade | “Skill” para o mecanismo formal; “habilidade” em sentido genérico |
| plugin | plugin | |
| marketplace | marketplace | |
| hook | hook | explicar brevemente na primeira ocorrência |
| datasource | fonte de dados | exceto `Kimi Datasource` |
| MCP client | cliente MCP | |
| MCP server | servidor MCP | |
| endpoint | endpoint | |
| bearer token | token Bearer | |
| timeout | tempo limite | |
| frontmatter | frontmatter | primeira ocorrência: bloco de metadados no início do arquivo |
| config file | arquivo de configuração | |
| environment variable | variável de ambiente | |
| data location | local de armazenamento | |
| override | sobrescrita | |
| precedence | precedência | |
| build | build | usar “executar o build” |
| log | log | usar “consultar os logs” |
| changelog | changelog | |

## Navegação proposta

| English | pt-BR |
| --- | --- |
| Guides | Guias |
| Getting started | Primeiros passos |
| Migrating from kimi-cli | Migrar do kimi-cli |
| Common use cases | Casos de uso comuns |
| Interaction and input | Interação e entrada |
| Sessions and context | Sessões e contexto |
| Using Goals | Usar metas |
| Using in IDEs | Usar em IDEs |
| Customization | Personalização |
| Agents and Subagents | Agentes e subagentes |
| Hooks | Hooks |
| Custom themes | Temas personalizados |
| Configuration | Configuração |
| Config files | Arquivos de configuração |
| Providers and models | Provedores e modelos |
| Config overrides | Sobrescritas de configuração |
| Environment variables | Variáveis de ambiente |
| Data locations | Locais de armazenamento |
| Reference | Referência |
| Built-in tools | Ferramentas integradas |
| Slash commands | Comandos de barra |
| Keyboard shortcuts | Atalhos de teclado |
| Release notes | Notas de versão |

## Termos para revisão contínua

Os termos abaixo permanecem sujeitos a validação em páginas reais antes de serem tratados como decisões definitivas:

- Skill
- Agent Skills
- comando de barra
- hook
- marketplace
- fork
- frontmatter
- build
- modo Auto
- locais de armazenamento
- sobrescritas de configuração

## Como propor uma alteração

Ao sugerir um novo termo ou mudar uma decisão existente, inclua:

1. termo original;
2. tradução proposta;
3. contexto onde aparece;
4. exemplo em uma frase real;
5. motivo da escolha;
6. páginas afetadas.

Evite mudanças globais de terminologia sem antes testar a nova forma em contexto.
