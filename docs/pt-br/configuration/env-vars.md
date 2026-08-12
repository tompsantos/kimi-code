# Variáveis de ambiente

O Kimi Code CLI usa variáveis de ambiente para controlar um pequeno conjunto de comportamentos de runtime, como mover o diretório de dados, desativar a telemetria e trocar temporariamente de modelo sem alterar o arquivo de configuração.

::: warning Importante: as chaves de API não são configuradas aqui
Variáveis de credencial como `KIMI_API_KEY`, `ANTHROPIC_API_KEY` e `OPENAI_API_KEY` **não** são lidas automaticamente das variáveis de ambiente do shell. Executar `export KIMI_API_KEY=xxx` no terminal não fornece a chave a nenhum provedor; elas precisam ser gravadas em `config.toml`, em `[providers.<name>]` ou na subtabela `[providers.<name>.env]`.

A única exceção é a família `KIMI_MODEL_*`, que funciona como um canal explícito que *de fato* lê credenciais do shell. Consulte a seção **Definir um modelo por variáveis de ambiente (`KIMI_MODEL_*`)** abaixo.

Para entender o contexto, consulte [Sobrescritas de configuração: credenciais de provedores](./overrides.md#credenciais-de-provedores).
:::

## Caminhos principais

### `KIMI_CODE_HOME`

Sobrescreve o diretório raiz de dados; o padrão é `~/.kimi-code`. Depois de definido, o arquivo de configuração, as sessões, os logs, as credenciais OAuth e todos os outros dados passam a ficar no novo caminho:

```sh
export KIMI_CODE_HOME="/path/to/custom/kimi-code"
```

> Verifique se o diretório permite gravação. Várias instâncias de `kimi` que compartilham o mesmo `KIMI_CODE_HOME` também compartilharão os arquivos de configuração e credenciais.

Para ver a estrutura completa do diretório de dados, consulte [Locais de armazenamento](../../en/configuration/data-locations.md).

### `KIMI_DISABLE_TELEMETRY`

Defina como `1` para desativar o envio de telemetria anônima. Também são aceitos `true`, `yes` e `y`, sem distinção entre maiúsculas e minúsculas:

```sh
export KIMI_DISABLE_TELEMETRY=1
```

### Família `KIMI_MODEL_*`

Permite trocar temporariamente de modelo sem modificar `config.toml`. Quando `KIMI_MODEL_NAME` está definido, a CLI cria em memória um provedor temporário; a alteração não persiste depois da reinicialização. Consulte a seção **Definir um modelo por variáveis de ambiente (`KIMI_MODEL_*`)** abaixo.

## Nomes de chaves de credencial dos provedores (gravados em config.toml)

Os nomes de chave abaixo não são lidos diretamente do shell. Eles são nomes de chave gravados dentro da subtabela `[providers.<name>.env]` de `config.toml`, servindo como valores de fallback para `api_key` / `base_url`. A CLI lê somente o arquivo de configuração, e não `process.env`.

Esse desenho permite manter convenções conhecidas de nomes de chave enquanto centraliza o gerenciamento de segredos no arquivo de configuração:

```toml
[providers.kimi.env]
KIMI_API_KEY = "sk-xxx"
KIMI_BASE_URL = "https://api.moonshot.ai/v1"
```

Nomes de chave por provedor:

| Chave | Provedor aplicável | Padrão |
| --- | --- | --- |
| `KIMI_API_KEY` | Kimi / Moonshot | Nenhum |
| `KIMI_BASE_URL` | Kimi / Moonshot | `https://api.moonshot.ai/v1` |
| `ANTHROPIC_API_KEY` | Anthropic | Nenhum |
| `ANTHROPIC_BASE_URL` | Anthropic | Segue o padrão do SDK da Anthropic |
| `OPENAI_API_KEY` | OpenAI (`openai` e `openai_responses`) | Nenhum |
| `OPENAI_BASE_URL` | OpenAI (`openai` e `openai_responses`) | `https://api.openai.com/v1` |
| `GOOGLE_API_KEY` | Google GenAI, Vertex AI | Nenhum |
| `VERTEXAI_API_KEY` | Vertex AI | Nenhum |
| `GOOGLE_CLOUD_PROJECT` | Vertex AI | Nenhum |
| `GOOGLE_CLOUD_LOCATION` | Vertex AI | Nenhum |

::: warning Atenção
`GOOGLE_APPLICATION_CREDENTIALS`, que aponta para um arquivo JSON de conta de serviço, é a única exceção que passa pelo mecanismo de variáveis de ambiente do sistema. Ela é lida diretamente pelo SDK do Google por meio do fluxo padrão de ADC, sem participação da CLI. Todos os outros nomes de chave precisam estar na subtabela `[providers.<name>.env]` para terem efeito.
:::

Para a referência completa dos tipos e campos de provedores, consulte [Provedores e modelos](./providers.md).

## OAuth e serviços gerenciados

Este grupo de variáveis redireciona a autenticação OAuth e os endpoints dos serviços gerenciados para um ambiente auto-hospedado ou de testes. Elas não são necessárias no uso cotidiano.

| Variável | Finalidade | Padrão |
| --- | --- | --- |
| `KIMI_CODE_OAUTH_HOST` | Host de autenticação OAuth; maior prioridade | Usa `KIMI_OAUTH_HOST` como fallback quando não definido |
| `KIMI_OAUTH_HOST` | Host de autenticação OAuth; fallback de `KIMI_CODE_OAUTH_HOST` | Usa `https://auth.kimi.com` como fallback quando não definido |
| `KIMI_CODE_BASE_URL` | URL base da API gerenciada usada depois do login OAuth | `https://api.kimi.com/coding/v1` |

::: warning Atenção
`KIMI_CODE_BASE_URL`, usado pelo serviço gerenciado via OAuth e direcionado a `kimi.com`, e `KIMI_BASE_URL`, usado na conexão direta por chave de API e direcionado a `moonshot.ai`, são duas variáveis distintas. Use cada uma em seu contexto apropriado.
:::

## Definir um modelo por variáveis de ambiente (`KIMI_MODEL_*`)

Quer trocar de modelo para testes sem mexer em `config.toml`? Quando `KIMI_MODEL_NAME` está definido, a CLI cria em memória um provedor temporário e um alias de modelo a partir das variáveis `KIMI_MODEL_*`; nada é gravado de volta no arquivo de configuração. Essas variáveis têm prioridade sobre `default_model` em `config.toml`, mas a opção `-m <alias>` usada na inicialização ainda tem a maior prioridade.

```sh
export KIMI_MODEL_NAME="kimi-for-coding"
export KIMI_MODEL_API_KEY="YOUR_API_KEY"
export KIMI_MODEL_BASE_URL="https://api.example.com/v1"
export KIMI_MODEL_MAX_CONTEXT_SIZE="262144"
export KIMI_MODEL_CAPABILITIES="image_in,thinking"
kimi
```

Lista completa de variáveis:

| Variável | Obrigatória | Finalidade | Padrão |
| --- | --- | --- | --- |
| `KIMI_MODEL_NAME` | Sim (também habilita o recurso) | ID do modelo enviado à API | — |
| `KIMI_MODEL_API_KEY` | Sim | Chave de API | — |
| `KIMI_MODEL_PROVIDER_TYPE` | Não | Tipo de provedor: `kimi`, `anthropic`, `openai` | `kimi` |
| `KIMI_MODEL_BASE_URL` | Não | URL base da API | Cada tipo tem seu próprio padrão |
| `KIMI_MODEL_MAX_CONTEXT_SIZE` | Não | Tamanho máximo do contexto em tokens | `262144` (256 K) |
| `KIMI_MODEL_CAPABILITIES` | Não | Tags de recursos separadas por vírgula, combinadas com os recursos detectados automaticamente | `image_in,thinking` |
| `KIMI_MODEL_DISPLAY_NAME` | Não | Nome mostrado em `/model` | Usa `KIMI_MODEL_NAME` como fallback |
| `KIMI_MODEL_MAX_OUTPUT_SIZE` | Não | Limite de saída por requisição, somente em `anthropic`; quando definido, sobrescreve o limite integrado do Claude | Padrão do modelo |
| `KIMI_MODEL_REASONING_KEY` | Não | Sobrescrita do nome do campo de raciocínio, somente em `openai` | Detectado automaticamente |
| `KIMI_MODEL_THINKING_EFFORT` | Não | Nível de esforço de Thinking: `low`/`medium`/`high`/`xhigh`/`max` | — |
| `KIMI_MODEL_ADAPTIVE_THINKING` | Não | Força o Thinking adaptativo a ficar ligado ou desligado, somente em `anthropic` | Inferido pelo nome do modelo |

Se `KIMI_MODEL_NAME` estiver definido e uma variável obrigatória estiver ausente, a inicialização falha imediatamente com uma mensagem de erro clara.

## Chaves de runtime

Chaves que controlam o comportamento de subsistemas como telemetria, tarefas em segundo plano e o marketplace de plugins:

| Variável | Finalidade | Valores válidos |
| --- | --- | --- |
| `KIMI_DISABLE_TELEMETRY` | Desativar o envio de telemetria anônima | `1`, `true`, `yes`, `y` (sem distinção entre maiúsculas e minúsculas) |
| `KIMI_CODE_BACKGROUND_KEEP_ALIVE_ON_EXIT` | Define se tarefas em segundo plano continuam ativas quando a sessão é encerrada; tem prioridade sobre `config.toml`. O padrão é interrompê-las ao sair | Verdadeiros: `1`/`true`/`yes`/`on`; falsos: `0`/`false`/`no`/`off` |
| `KIMI_CODE_BACKGROUND_MAX_RUNNING_TASKS` | Limite de tarefas em segundo plano executadas simultaneamente; tem prioridade sobre `[background] max_running_tasks` em `config.toml` (não definido significa sem limite) | Inteiro positivo; valores inválidos são ignorados |
| `KIMI_IMAGE_MAX_EDGE_PX` | Limite do maior lado, em px, na compactação de imagens; tem prioridade sobre `[image] max_edge_px` em `config.toml` (padrão `2000`) | Inteiro positivo; valores inválidos são ignorados |
| `KIMI_IMAGE_READ_BYTE_BUDGET` | Orçamento de bytes por imagem para leituras iniciadas pelo modelo, nas leituras padrão de `ReadMediaFile`; tem prioridade sobre `[image] read_byte_budget` em `config.toml` (padrão `262144`, ou 256 KB) | Inteiro positivo; valores inválidos são ignorados |
| `KIMI_CODE_PLUGIN_MARKETPLACE_URL` | Sobrescreve o JSON do marketplace de plugins carregado por `/plugins`; útil para servidores locais de desenvolvimento, arquivos em CDN de staging ou diretórios alternativos de marketplace | `https://code.kimi.com/kimi-code/plugins/marketplace.json`; também aceita URLs `http://`, `file://` e caminhos locais |
| `KIMI_CODE_AGENT_SWARM_MAX_CONCURRENCY` | Limita quantos subagentes do AgentSwarm executam simultaneamente durante a subida inicial; deixe não definido para não impor limite | Inteiro positivo; valores inválidos causam falha imediata |
| `KIMI_SUBAGENT_TIMEOUT_MS` | Tempo máximo de execução, em ms, de um único subagente (`Agent` / `AgentSwarm`); tem prioridade sobre `[subagent] timeout_ms` em `config.toml` (padrão `7200000`, ou 2 horas) | Inteiro positivo; valores inválidos usam como fallback a configuração ou o padrão |
| `KIMI_CODE_IDENTITY_NAME` | Nome de exibição usado pelo agente para se identificar no prompt do sistema; tem prioridade sobre `[identity] name` em `config.toml` e nunca é gravado de volta nele | Qualquer string não vazia; valores em branco são tratados como não definidos |
| `KIMI_CODE_IDENTITY_SLUG` | Identificador de protocolo usado no token de produto `User-Agent` enviado a provedores externos e como nome do cliente MCP; tem prioridade sobre `[identity] slug`. Quando não definido, é derivado do nome | Qualquer string não vazia; normalizada para minúsculas e com sequências de caracteres não alfanuméricos convertidas em `-` |
| `KIMI_CODE_BUILTIN_PRODUCT_SKILLS` | Define se as Skills integradas que documentam o próprio Kimi Code são oferecidas ao modelo; tem prioridade sobre `builtin_product_skills` em `config.toml` (habilitado por padrão) | Verdadeiros: `1`/`true`/`yes`/`on`; falsos: `0`/`false`/`no`/`off` |
| `KIMI_CODE_EXPERIMENTAL_SECONDARY_MODEL` | Habilita o recurso experimental de modelo secundário em todos os modos de inicialização, incluindo a TUI interativa; a chave mestra `KIMI_CODE_EXPERIMENTAL_FLAG=1` também o habilita | Verdadeiros: `1`/`true`/`yes`/`on`; falsos: `0`/`false`/`no`/`off` |
| `KIMI_SECONDARY_MODEL` | Modelo secundário; tem prioridade sobre [`[secondary_model] model`](./config-files.md#secondary-model) em `config.toml`. Quando o experimento de modelo secundário está habilitado, novos subagentes (`Agent` / `AgentSwarm`) usam esse modelo por padrão em vez de herdar o modelo do agente principal | Alias de uma entrada `[models]` configurada, por exemplo `kimi-code/kimi-k2.5`; valores em branco são ignorados |
| `KIMI_SECONDARY_EFFORT` | Esforço de Thinking do modelo secundário; tem prioridade sobre `[secondary_model] default_effort` em `config.toml` e só se aplica quando tanto o modelo quanto o experimento estão habilitados | Um valor de esforço, por exemplo `low`; valores em branco são ignorados |
| `KIMI_MCP_STARTUP_TIMEOUT_MS` | Tempo limite global de conexão, em ms, para todos os servidores MCP; tem prioridade sobre `[mcp] startup_timeout_ms` em `config.toml`, mas um `startupTimeoutMs` por servidor em `mcp.json` ainda vence (padrão `30000`) | Inteiro de `1` a `2147483647`; valores inválidos são ignorados |
| `KIMI_MCP_TOOL_TIMEOUT_MS` | Tempo limite global, em ms, para uma única chamada de ferramenta em todos os servidores MCP; tem prioridade sobre `[mcp] tool_timeout_ms` em `config.toml`, mas um `toolTimeoutMs` por servidor em `mcp.json` ainda vence (padrão `60000`) | Inteiro de `1` a `2147483647`; valores inválidos são ignorados |
| `KIMI_LOOP_MAX_STEPS_PER_TURN` | Número máximo de etapas do agente por turno; tem prioridade sobre `[loop_control] max_steps_per_turn` em `config.toml` (não definido ou `0` significa sem limite) | Inteiro não negativo; valores inválidos são ignorados |
| `KIMI_LOOP_MAX_ATTEMPTS_PER_STEP` | Número máximo total de tentativas para uma etapa com falha, incluindo a tentativa inicial; tem prioridade sobre `[loop_control] max_attempts_per_step` em `config.toml` (padrão `10`). A variável obsoleta `KIMI_LOOP_MAX_RETRIES_PER_STEP` continua sendo aceita, com um aviso, quando esta variável não está definida | Inteiro não negativo; valores inválidos são ignorados |
| `KIMI_TOKEN_COUNTING_STRATEGY` | Define qual contagem de tokens do contexto é exibida externamente, no indicador de tamanho de contexto; tem prioridade sobre `[token_counting] strategy` em `config.toml` (padrão `measured+estimated`) | `measured+estimated`, `measured`, `estimated`, sem distinção entre maiúsculas e minúsculas; valores inválidos são ignorados |
| `KIMI_WEB_SEARCH_BASE_URL` | URL da API do serviço de busca web (`WebSearch`); tem prioridade sobre `[services.moonshot_search] base_url` em `config.toml` e habilita o serviço mesmo sem essa seção de configuração. Credenciais persistidas e cabeçalhos personalizados não são encaminhados para um endpoint selecionado por variável de ambiente | String não vazia; valores em branco são ignorados |
| `KIMI_WEB_SEARCH_API_KEY` | Chave de API do serviço de busca web (`WebSearch`); quando definida, substitui tanto a chave de API configurada quanto a credencial OAuth | String não vazia; valores em branco são ignorados |
| `KIMI_WEB_FETCH_BASE_URL` | URL da API do serviço de busca de conteúdo (`FetchURL`); tem prioridade sobre `[services.moonshot_fetch] base_url`. Credenciais persistidas e cabeçalhos personalizados não são encaminhados para um endpoint selecionado por variável de ambiente. Sem endpoint em variável de ambiente ou configuração, usuários autenticados tentam primeiro o serviço gerenciado de fetch via OAuth do Kimi, antes de requisições locais diretas | String não vazia; valores em branco são ignorados |
| `KIMI_WEB_FETCH_API_KEY` | Chave de API do serviço de busca de conteúdo (`FetchURL`); quando definida, substitui tanto a chave de API configurada quanto a credencial OAuth | String não vazia; valores em branco são ignorados |
| `KIMI_CODE_EXPERIMENTAL_FLAG` | Habilita todos os recursos experimentais registrados para este processo; não seleciona o mecanismo do agente | `1`, `true`, `yes`, `on` |
| `KIMI_CODE_LEGACY_FLAG` | Usa o mecanismo legado `agent-core` para `kimi`, `kimi -p`, `kimi doctor`, `kimi acp`, `kimi export` e `kimi provider`; esses comandos usam `agent-core-v2` por padrão | `1`, `true`, `yes`, `on` |
| `KIMI_SHELL_PATH` | Sobrescreve o caminho do Git Bash no Windows, usado quando a detecção automática falha | Caminho absoluto |
| `KIMI_MODEL_MAX_COMPLETION_TOKENS` | Limite rígido de `max_completion_tokens` por etapa do LLM; aplica-se somente ao provedor `kimi` | Inteiro positivo; `0` ou negativo desabilita o limite |
| `KIMI_MODEL_TEMPERATURE` | Temperatura de amostragem para todas as requisições; aplica-se somente ao provedor `kimi` e é global, independente de `KIMI_MODEL_NAME` | Número, por exemplo `0.3` |
| `KIMI_MODEL_TOP_P` | `top_p` de nucleus sampling para todas as requisições; aplica-se somente ao provedor `kimi` e é global | Número, por exemplo `0.95` |
| `KIMI_MODEL_THINKING_EFFORT` | Força um esforço específico de Thinking enviado pela API em `thinking.effort`, ignorando `support_efforts` declarado pelo modelo; aplica-se somente ao provedor `kimi` e apenas quando Thinking está ligado | Um valor de esforço, por exemplo `max` |
| `KIMI_MODEL_THINKING_KEEP` | Passagem preservada de Thinking; em `kimi`, é enviada como `thinking.keep`; em `anthropic`, incluindo Claude e o modo do Kimi compatível com Anthropic, é enviada como uma edição `context_management` `clear_thinking_20251015`. Habilitar `keep` direciona requisições Anthropic para a beta Messages API. Sobrescreve `[thinking] keep`, cujo padrão é `"all"`, e só é injetada quando Thinking está ligado | Um valor aceito pela API, por exemplo `all`; um valor de desativação (`false`/`0`/`no`/`off`/`none`/`null`) desabilita o recurso |
| `KIMI_CODE_NO_AUTO_UPDATE` | Desabilita totalmente a verificação prévia de atualização: sem checagem, instalação em segundo plano ou aviso. O alias legado `KIMI_CLI_NO_AUTO_UPDATE` também é aceito | Verdadeiros: `1`/`true`/`yes`/`on` |
| `KIMI_DISABLE_CRON` | Desabilita a ferramenta de tarefas agendadas: `CronCreate` rejeita novos agendamentos e as tarefas existentes não são executadas | `1` para desabilitar |

As três variáveis `KIMI_CODE_IDENTITY_*` / `KIMI_CODE_BUILTIN_PRODUCT_SKILLS` são lidas pelo mecanismo padrão `agent-core-v2`. O caminho legado `kimi` / `kimi -p` selecionado por `KIMI_CODE_LEGACY_FLAG=1` as ignora.

## Logs de diagnóstico

Estas variáveis controlam o nível de log e a rotação de arquivos, e são lidas uma vez na inicialização do processo:

| Variável | Finalidade | Padrão |
| --- | --- | --- |
| `KIMI_LOG_LEVEL` | Nível de log: `off`, `error`, `warn`, `info`, `debug` | `info` |
| `KIMI_LOG_GLOBAL_MAX_BYTES` | Máximo de bytes por arquivo de log global | `6291456` (6 MB) |
| `KIMI_LOG_GLOBAL_FILES` | Número de arquivos de log global a manter | `5` |
| `KIMI_LOG_SESSION_MAX_BYTES` | Máximo de bytes por arquivo de log de sessão | `5242880` (5 MB) |
| `KIMI_LOG_SESSION_FILES` | Número de arquivos de log de sessão a manter | `3` |

## Variáveis de ambiente do sistema

A CLI também lê diversas variáveis padrão do sistema para detectar o ambiente de runtime; ela não as modifica:

- `HOME`: usada para resolver o caminho padrão dos dados
- `VISUAL`, `EDITOR`: comando do editor externo (`VISUAL` tem precedência)
- `PATH`: usada para localizar dependências como `rg`, `fd`, `fdfind` e `git`; no Windows, a detecção do Git Bash verifica cada `git.exe` encontrado em `PATH`, incluindo shims de gerenciadores de pacotes como Scoop
- `NO_COLOR`, `FORCE_COLOR`: controlam a saída colorida, seguindo a convenção do [no-color.org](https://no-color.org)
- `CI`: quando não está vazia nem é `"0"`, desativa a detecção de tema e usa o tema escuro como fallback
- `TERM_PROGRAM`, `TERM`, `TMUX`: detectam recursos do terminal e suporte a notificações
- `DISPLAY`, `WAYLAND_DISPLAY`, `XDG_SESSION_TYPE`: detectam sessões gráficas no Linux, usadas por recursos de área de transferência e imagens
- `WSL_DISTRO_NAME`, `WSLENV`: detectam WSL para a ponte de área de transferência via PowerShell
- `LOCALAPPDATA`: usada no Windows como fallback ao procurar o caminho de instalação do Git Bash

## Proxy HTTP

O Kimi Code respeita as variáveis de ambiente padrão de proxy para todo o tráfego de saída, incluindo chamadas às APIs dos modelos, servidores MCP, ferramentas web, telemetria, login e verificações de atualização:

- `HTTP_PROXY` / `http_proxy`: proxy para requisições `http://`
- `HTTPS_PROXY` / `https_proxy`: proxy para requisições `https://`
- `ALL_PROXY` / `all_proxy`: proxy de fallback usado quando a variável específica do protocolo não está definida; normalmente é aqui que um proxy SOCKS é configurado
- `NO_PROXY` / `no_proxy`: lista de hosts separados por vírgula que ignoram o proxy

Proxies HTTP(S) e SOCKS são compatíveis. Um proxy SOCKS é reconhecido pelo esquema, como `socks5://`, `socks5h://`, `socks4://` ou `socks://`, que é um alias de `socks5://`, e normalmente é configurado por `ALL_PROXY`, formato usado por ferramentas como Clash e V2RayN. Para tráfego HTTP/HTTPS, um proxy HTTP(S) tem prioridade sobre `ALL_PROXY`.

O proxy só é aplicado quando uma dessas variáveis está definida; caso contrário, as conexões são feitas diretamente. Hosts de loopback (`localhost`, `127.0.0.1`, `::1`) sempre ignoram o proxy, portanto um servidor local, como um servidor MCP em localhost, continua funcionando mesmo com proxy configurado. Adicione seus próprios hosts internos a `NO_PROXY` para excluí-los também.

Servidores MCP stdio executados como processos filhos Node respeitam automaticamente `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` quando a versão do Node do processo filho oferece suporte a `NODE_USE_ENV_PROXY` (Node ≥ 22.21 ou ≥ 24.5). O proxy SOCKS se aplica somente ao tráfego do próprio Kimi Code.

## Próximos passos

- [Sobrescritas de configuração](./overrides.md): como variáveis de ambiente, opções da CLI e o arquivo de configuração interagem por prioridade
- [Locais de armazenamento](../../en/configuration/data-locations.md): estrutura de diretórios afetada por `KIMI_CODE_HOME`
- [Provedores e modelos](./providers.md): exemplos completos de conexão para cada tipo de provedor
