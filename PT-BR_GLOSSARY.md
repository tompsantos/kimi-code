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
| turn | turno | uma interação em andamento entre usuário, agente e ferramentas |
| fork a session | criar um fork da sessão | preservar “fork” |
| tool | ferramenta | |
| tool call | chamada de ferramenta | |
| built-in tool | ferramenta integrada | |
| tool approval | aprovação de ferramenta | |
| approval request | solicitação de aprovação | |
| approval panel | painel de aprovação | |
| permission rule | regra de permissão | |
| allowlist | lista de permissões | |
| denylist / blocklist | lista de bloqueio | |
| wildcard | curinga | em padrões de permissão |
| glob pattern | padrão glob | preservar “glob” e explicar pelo contexto quando necessário |
| TUI | TUI | primeira ocorrência: “interface de usuário no terminal” |
| CLI | CLI | primeira ocorrência: “interface de linha de comando” |
| shell | shell | explicar na primeira ocorrência quando necessário |
| runtime | runtime | manter em inglês quando se referir ao ambiente/comportamento em execução |
| print mode | modo print | usado para `kimi -p` |
| standalone binary | binário independente | |
| package manager | gerenciador de pacotes | |
| API key | chave de API | preservar labels literais da interface, como `Kimi Platform API key` |
| checksum | checksum / soma de verificação | manter o termo técnico e explicar na primeira ocorrência |
| read-only operation | operação somente leitura | |
| input box | caixa de entrada | |
| conversation view | área da conversa | |
| status bar | barra de status | |
| status line | linha de status | |
| clipboard | área de transferência | |
| placeholder | marcador temporário | em campos de entrada e mídia colada |
| multimodal capability | recurso multimodal | |
| completion menu | menu de autocompletar | preferido a “menu de conclusão” |
| file reference | referência de arquivo | |
| streaming output | saída em streaming | |
| working directory | diretório de trabalho | |
| workspace | workspace | manter em inglês no contexto formal de diretórios de trabalho adicionais |
| background task | tarefa em segundo plano | |
| sensitive action | ação sensível | segurança e permissões |
| external editor | editor externo | |
| Plan mode | modo Plan | nome do modo preservado |
| YOLO mode | modo YOLO | nome do modo preservado |
| Auto mode | modo Auto | nome do modo preservado |
| Thinking mode | modo Thinking | nome do modo preservado |
| Shell mode | modo shell | |
| slash command | comando de barra | |
| skill | Skill / habilidade | “Skill” para o mecanismo formal; “habilidade” em sentido genérico |
| plugin | plugin | |
| plugin manifest | manifesto do plugin | |
| marketplace | marketplace | |
| hook | hook | explicar brevemente na primeira ocorrência |
| datasource | fonte de dados | exceto `Kimi Datasource` |
| MCP client | cliente MCP | |
| MCP server | servidor MCP | |
| stdio | stdio / entrada e saída padrão | preservar `stdio` como nome do transporte e explicar na primeira ocorrência |
| child process | processo filho | |
| transport | transporte | preservar o campo `transport` em código e configuração |
| endpoint | endpoint | |
| request header | cabeçalho de requisição | |
| static credentials | credenciais estáticas | |
| browser-based authorization | autorização pelo navegador | |
| bearer token | token Bearer | |
| timeout | tempo limite | |
| high-risk tool | ferramenta de alto risco | |
| snake_case | snake_case | preservar como nome da convenção de identificadores |
| scalar | valor escalar | em TOML/configuração |
| nested table | tabela aninhada | em TOML/configuração |
| model alias | alias de modelo | |
| model patch | patch de modelo | preservar “patch” como termo técnico |
| transient failure | falha transitória | |
| rate limit | limite de taxa | HTTP/API |
| quota | cota | |
| telemetry | telemetria | |
| project-local configuration | configuração local do projeto | |
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
- runtime
- modo print
- workspace
- patch

## Como propor uma alteração

Ao sugerir um novo termo ou mudar uma decisão existente, inclua:

1. termo original;
2. tradução proposta;
3. contexto onde aparece;
4. exemplo em uma frase real;
5. motivo da escolha;
6. páginas afetadas.

Evite mudanças globais de terminologia sem antes testar a nova forma em contexto.
