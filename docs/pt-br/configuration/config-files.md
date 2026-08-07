# Arquivos de configuração

O Kimi Code CLI armazena todas as preferências de longo prazo, como qual modelo usar, qual chave de API preencher e quantas etapas um agente pode executar por turno, em arquivos TOML (um formato de configuração em texto simples e de estrutura clara). Altere essas preferências uma vez e elas passarão a valer em todas as inicializações. As configurações do agente e do runtime ficam em `config.toml`; as preferências da interface do terminal e do cliente, como tema, editor, notificações e atualização automática, ficam no arquivo complementar `tui.toml`.

Local padrão: `~/.kimi-code/config.toml`, criado automaticamente na primeira execução.

## Local do arquivo de configuração

A CLI lê a configuração de `~/.kimi-code/config.toml`. Para mover o diretório de dados, substitua o local padrão usando a variável de ambiente `KIMI_CODE_HOME`:

```sh
export KIMI_CODE_HOME=/path/to/kimi-home
```

O caminho do arquivo de configuração passa então a ser `$KIMI_CODE_HOME/config.toml`. Independentemente de onde o diretório esteja, o nome do arquivo é sempre `config.toml`.

::: tip Dica
Os nomes dos campos TOML sempre usam `snake_case`, por exemplo `default_model` e `max_context_size`. Se uma chave contiver `.`, coloque-a entre aspas, por exemplo `[models."gpt-4.1"]`; caso contrário, o TOML interpreta `.` como separador de tabelas aninhadas.
:::

## Exemplo completo

O exemplo abaixo cobre os campos de configuração usados com mais frequência. Você pode copiá-lo e ajustá-lo conforme necessário:

```toml
default_model = "kimi-code/k3"
default_permission_mode = "manual"
default_plan_mode = false
merge_all_available_skills = true
telemetry = true

[providers."managed:kimi-code"]
type = "kimi"
base_url = "https://api.kimi.com/coding/v1"
api_key = ""

[models."kimi-code/k3"]
provider = "managed:kimi-code"
model = "k3"
max_context_size = 1048576
capabilities = [ "thinking", "always_thinking", "image_in", "video_in", "tool_use" ]
display_name = "K3"
support_efforts = [ "max" ]
default_effort = "max"

[models."kimi-code/kimi-for-coding"]
provider = "managed:kimi-code"
model = "kimi-for-coding"
max_context_size = 262144
capabilities = [ "thinking", "always_thinking", "image_in", "video_in", "tool_use" ]

[models."kimi-code/kimi-for-coding-highspeed"]
provider = "managed:kimi-code"
model = "kimi-for-coding-highspeed"
max_context_size = 262144
capabilities = [ "thinking", "always_thinking", "image_in", "video_in", "tool_use" ]

[thinking]
enabled = true
effort = "high"
keep = "all"

[loop_control]
max_attempts_per_step = 10
reserved_context_size = 50000

[background]
max_running_tasks = 4
keep_alive_on_exit = false

[services.moonshot_search]
base_url = "https://api.kimi.com/coding/v1/search"
api_key = ""

[services.moonshot_fetch]
base_url = "https://api.kimi.com/coding/v1/fetch"
api_key = ""

[[permission.rules]]
decision = "allow"
pattern = "Read"

[[permission.rules]]
decision = "deny"
pattern = "Bash(rm -rf*)"

[[hooks]]
event = "PreToolUse"
matcher = "Bash"
command = "node ~/.kimi-code/hooks/check-bash.mjs"
timeout = 5
```

## Campos de nível superior

Os campos do arquivo de configuração se dividem em duas categorias: **valores escalares de nível superior**, que controlam diretamente o comportamento padrão, e **tabelas aninhadas** (`providers`, `models`, `thinking` etc.), cada uma com sua própria estrutura, descrita nas seções abaixo.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `default_model` | `string` | — | Alias do modelo padrão; deve estar definido em `models` |
| `default_permission_mode` | `string` | `manual` | Modo de permissão padrão para novas sessões: `manual` (solicitar confirmação a cada vez), `yolo` (aprovar automaticamente ações de ferramentas, mas o agente ainda pode fazer perguntas) ou `auto` (totalmente autônomo; o agente decide tudo sem perguntar) |
| `default_plan_mode` | `boolean` | `false` | Define se novas sessões começam por padrão no modo Plan, produzindo um plano antes da execução |
| `merge_all_available_skills` | `boolean` | `true` | Define se as Agent Skills de todos os diretórios disponíveis serão combinadas |
| `extra_skill_dirs` | `array<string>` | — | Diretórios adicionais de busca por Skills, acrescentados aos diretórios padrão |
| `extra_agent_dirs` | `array<string>` | — | Diretórios adicionais de busca por agentes personalizados, acrescentados aos diretórios padrão |
| `builtin_product_skills` | `boolean` | `true` | Define se as Skills integradas que documentam o próprio Kimi Code são oferecidas ao modelo: `update-config`, `custom-theme`, `mcp-config`, `check-kimi-code-docs` e `import-from-cc-codex`. Desabilitá-las remove seus nomes e descrições do prompt do sistema, mas também elimina os fluxos guiados dessas tarefas. O campo é lido pelo mecanismo padrão `agent-core-v2` e ignorado quando `KIMI_CODE_LEGACY_FLAG=1` seleciona o mecanismo legado |
| `telemetry` | `boolean` | `true` | Define se a telemetria anônima está habilitada; ela só é desabilitada quando o campo é definido explicitamente como `false` |
| `providers` | `table` | `{}` | Tabela de provedores de API → [`providers`](#providers) |
| `models` | `table` | — | Tabela de aliases de modelos → [`models`](#models) |
| `thinking` | `table` | — | Parâmetros padrão do modo Thinking → [`thinking`](#thinking) |
| `loop_control` | `table` | — | Parâmetros de controle do loop do agente → [`loop_control`](#loop-control) |
| `background` | `table` | — | Parâmetros de runtime das tarefas em segundo plano → [`background`](#background) |
| `tools` | `table` | — | Controle global de ferramentas → [`tools`](#tools) |
| `image` | `table` | — | Parâmetros de compactação de imagens → [`image`](#image) |
| `services` | `table` | — | Configuração de serviços externos integrados → [`services`](#services) |
| `permission` | `table` | — | Regras iniciais de permissão → [`permission`](#permission) |
| `hooks` | `array<table>` | — | Hooks do ciclo de vida; consulte [Hooks](../../en/customization/hooks.md) |
| `identity` | `table` | — | Identidade personalizada do agente → [`identity`](#identity) |

As seções seguintes descrevem cada uma das tabelas aninhadas: `providers`, `models`, `thinking`, `loop_control`, `background`, `tools`, `image`, `services` e `permission`.

## `providers`

Cada entrada na tabela `providers` define um provedor de API e é identificada por um nome exclusivo. A CLI lê as credenciais somente daqui. Ela **não** usa automaticamente variáveis de ambiente do shell como fallback. Executar `export KIMI_API_KEY` no terminal não fornece a chave a nenhum provedor; você precisa registrá-la explicitamente no arquivo de configuração. Consulte [Sobrescritas de configuração](../../en/configuration/overrides.md#provider-credentials).

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `type` | `string` | Sim | Tipo do provedor: `kimi`, `anthropic`, `openai`, `openai_responses`, `google-genai`, `vertexai` |
| `api_key` | `string` | Não | Chave de API, armazenada em texto simples no arquivo de configuração |
| `base_url` | `string` | Não | URL-base da API |
| `oauth` | `table` | Não | Referência de credenciais OAuth, com os campos `storage` e `key`; injetada automaticamente pelo fluxo de login e normalmente não precisa ser escrita manualmente |
| `env` | `table<string, string>` | Não | Fonte de fallback para credenciais do provedor; consulte abaixo |
| `custom_headers` | `table<string, string>` | Não | Cabeçalhos HTTP personalizados adicionados a cada requisição |

**Subtabela `env`**: você pode registrar nomes de chaves convencionais do provedor, como `KIMI_API_KEY`, em `[providers.<name>.env]` para usá-los como fonte de fallback de `api_key` / `base_url`. Essa subtabela é **lida somente do arquivo de configuração** e não modifica o ambiente do shell:

```toml
[providers.kimi.env]
KIMI_API_KEY = "sk-xxx"
KIMI_BASE_URL = "https://api.moonshot.ai/v1"
```

Prioridade: campo `api_key` > chave da subtabela `env` > se ambos estiverem ausentes, a inicialização falha com um erro.

## `models`

Cada entrada na tabela `models` define um alias de modelo, isto é, o nome usado em `default_model` ou na flag `-m`, e é identificada por um nome exclusivo.

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `provider` | `string` | Sim | Nome do provedor a ser usado; deve estar definido em `providers` |
| `model` | `string` | Sim | Identificador do modelo enviado ao servidor na chamada da API |
| `max_context_size` | `integer` | Sim | Tamanho máximo do contexto em tokens; deve ser pelo menos 1 |
| `max_input_size` | `integer` | Não | Limite declarado de entrada por requisição quando for menor que a janela total, por exemplo gpt-5 com janela de 400k e entrada de 272k. Compactação, verificações de estouro de contexto e proporções de uso dão preferência a esse valor; o orçamento de conclusão continua usando a janela total. A resolução limita esse valor a `max_context_size` |
| `max_output_size` | `integer` | Não | Limite de tokens de saída por requisição, mapeado para `max_tokens`. Atualmente apenas o provedor `anthropic` respeita esse campo. Quando definido para um modelo Claude, o valor explícito substitui o máximo integrado do servidor |
| `capabilities` | `array<string>` | Não | Tags de capacidade adicionadas explicitamente: `thinking`, `always_thinking`, `image_in`, `video_in`, `audio_in`, `tool_use`. Elas são combinadas com as capacidades detectadas automaticamente pelo provedor; entradas podem ser adicionadas, mas nunca removidas |
| `support_efforts` | `array<string>` | Não | Níveis de esforço de Thinking aceitos pelo modelo. Para `kimi`, selecionar outro valor em runtime falha; quando a resolução do modelo carrega um valor configurado ou anterior que não é aceito, a sessão usa como fallback o `default_effort` do modelo-alvo e informa o valor efetivo à interface. Um modelo Kimi com Thinking e sem esse campo usa `on` / `off` booleano. Outros provedores repassam valores concretos sem alteração quando o protocolo possui um campo nativo de esforço; protocolos que expõem apenas níveis ou orçamentos de tokens fazem a conversão necessária. Atualizações de modelos gerenciados e da plataforma aberta podem reescrever esse campo; para fixá-lo manualmente, defina `support_efforts` em `[models."<alias>".overrides]` |
| `default_effort` | `string` | Não | Esforço de Thinking padrão do modelo. Atualizações de modelos gerenciados e da plataforma aberta podem reescrever esse campo; para fixá-lo manualmente, defina `default_effort` em `[models."<alias>".overrides]` |
| `off_effort` | `string` | Não | Valor de esforço enviado na requisição para desabilitar Thinking, por exemplo `none` para xAI Grok. Só faz sentido em modelos que declaram essa codificação, como os importados de catálogos. Ao desligar Thinking, esse valor é enviado em vez de omitir o campo de esforço, o que é a única forma de realmente interromper o raciocínio em modelos que raciocinam por padrão |
| `base_url` | `string` | Não | Sobrescrita de endpoint por modelo, gravada por importações de catálogo para modelos de gateway servidos fora do padrão do provedor. A resolução dá preferência a esse valor em relação ao `base_url` do provedor e ele só entra em vigor junto com `protocol` |
| `display_name` | `string` | Não | Nome exibido na interface; usa `model` como fallback quando não definido |
| `reasoning_key` | `string` | Não | Somente para o provedor `openai`. Sobrescreve o nome do campo usado para conteúdo de raciocínio quando o gateway o retorna com um nome fora do padrão; por padrão, `reasoning_content`, `reasoning_details` e `reasoning` são detectados automaticamente |
| `adaptive_thinking` | `boolean` | Não | Somente para o provedor `anthropic`. Força Thinking adaptativo ligado ou desligado, substituindo a inferência de versão pelo nome do modelo. Omita para detectar automaticamente; Claude ≥ 4.6 usa modo adaptativo |

Quando um alias contiver `.`, use uma chave entre aspas:

```toml
[models."gpt-4.1"]
provider = "openai"
model = "gpt-4.1"
max_context_size = 1047576
```

### Sobrescritas de modelo

Use `[models."<alias>".overrides]` para sobrescritas do usuário que precisam sobreviver às atualizações de modelos do provedor. Os consumidores em runtime usam o valor efetivo: a sobrescrita, quando existir, ou o campo de nível superior.

```toml
[models."kimi-code/kimi-for-coding"]
provider = "managed:kimi-code"
model = "kimi-for-coding"
max_context_size = 262144

[models."kimi-code/kimi-for-coding".overrides]
max_context_size = 131072
display_name = "Kimi for Coding (custom)"
```

`[models."<alias>".overrides]` aceita campos comuns de modelo, como `max_context_size`, `max_input_size`, `max_output_size`, `capabilities`, `display_name`, `reasoning_key`, `adaptive_thinking`, `support_efforts`, `default_effort` e `off_effort`. Ela não aceita campos de identidade ou roteamento: `provider`, `model`, `protocol`, `beta_api` e `base_url`.

Também é possível alternar temporariamente de modelo sem modificar o arquivo de configuração. Ao definir variáveis de ambiente `KIMI_MODEL_*`, a CLI cria em memória um provedor temporário que não persiste após a reinicialização. Consulte [Definir um modelo por variáveis de ambiente](../../en/configuration/env-vars.md#define-a-model-from-environment-variables-kimi-model).

## `secondary_model`

O modelo secundário é uma segunda configuração de modelo ao lado do modelo principal, normalmente mais barata, destinada a recursos que não exigem toda a capacidade do modelo principal. Hoje, seu consumidor é a criação de subagentes: quando configurado, novos subagentes (`Agent` / `AgentSwarm`) usam esse modelo por padrão em vez de herdar o modelo do agente principal; quando não configurado, os subagentes herdam o modelo do agente principal.

Esse vínculo é um padrão, não uma imposição. Com o experimento habilitado, as ferramentas `Agent` / `AgentSwarm` ganham um parâmetro `model`, que aceita somente os valores simbólicos `"secondary"` / `"primary"`, e a descrição da ferramenta lista os modelos disponíveis com o padrão indicado. Na criação de um subagente, o modelo é resolvido nesta ordem: parâmetro `model` explícito da chamada da ferramenta → [`model_preference`](../../en/customization/agents.md#agent-file-format) do perfil → modelo secundário configurado, que é o padrão. Aqui, `"primary"` significa o modelo que o agente principal está usando naquele momento, e não necessariamente `default_model`, por exemplo após uma troca com `/model` durante a sessão.

Como substituir o padrão é uma decisão do próprio agente principal, e a descrição da ferramenta apenas sugere `"secondary"` para tarefas rotineiras e `"primary"` para tarefas difíceis ou sensíveis à qualidade, não existe um seletor por criação de subagente do lado do usuário. Para direcionar um subagente específico ao modelo principal, peça ao agente principal no prompt que envie `model: "primary"`, ou defina `model_preference: "primary"` no perfil correspondente.

Esse recurso é experimental e vem desabilitado por padrão. Habilite-o com `KIMI_CODE_EXPERIMENTAL_SECONDARY_MODEL=1` ou com a flag-mestra `KIMI_CODE_EXPERIMENTAL_FLAG=1`. Ele funciona em todos os modos de execução, incluindo a TUI interativa.

Na TUI interativa, o comando [`/secondary_model`](../../en/reference/slash-commands.md) abre um seletor de modelos que grava esta seção e aplica a alteração imediatamente à sessão atual, de modo que novos subagentes usem o novo modelo secundário sem reiniciar.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `model` | `string` | — | Alias de uma entrada [`[models]`](#models) configurada, por exemplo `kimi-code/kimi-k2.5`; pode ser de qualquer provedor, não apenas modelos Kimi |
| `default_effort` | `string` | — | Esforço de Thinking aplicado quando subagentes usam o modelo secundário. Quando não definido, o esforço é resolvido naturalmente, pela configuração global `[thinking]` e depois pelo esforço padrão do modelo vinculado, em vez de herdar o esforço do agente principal. Segue a mesma semântica de esforço do modelo principal: modelos com validação estrita, como modelos Kimi, usam seu esforço padrão como fallback para valores não aceitos; outros provedores recebem o valor sem alteração |
| Outros campos | — | — | Aceita todos os campos de [`[models."<alias>".overrides]`](#models), como `max_context_size`, `max_output_size`, `support_efforts` etc., como um patch aplicado somente aos subagentes |

Todo campo além de `model` forma um patch. Quando pelo menos um campo de patch é definido, o runtime cria em memória uma entrada de modelo derivada, copiando a entrada apontada e combinando o patch com suas sobrescritas, com o patch prevalecendo em conflitos. Os subagentes usam essa entrada derivada. Sem campos de patch, eles usam diretamente a entrada apontada. A entrada derivada existe somente em memória, nunca é gravada de volta em `config.toml` e fica oculta das listas de seleção de modelos.

```toml
[secondary_model]
model = "kimi-code/kimi-k2.5"
default_effort = "low"
max_output_size = 8192
```

`model` / `default_effort` podem ser sobrescritos pelas variáveis de ambiente `KIMI_SECONDARY_MODEL` / `KIMI_SECONDARY_EFFORT`, que têm prioridade sobre `config.toml`.

Com o experimento habilitado, a configuração é validada no início da sessão. Um `model` que não possa ser resolvido ou um `default_effort` que não esteja listado pelo modelo, já considerando o patch, gera um aviso de inicialização, também retornado pela API de avisos da sessão. Essa verificação é apenas consultiva: um modelo secundário inválido ainda falhará na criação do subagente, com a mesma indicação de origem anexada ao erro.

## `thinking`

`thinking` define o comportamento global padrão do modo Thinking.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `enabled` | `boolean` | `true` | Define se Thinking vem habilitado por padrão em novas sessões; use `false` para forçar Thinking desligado |
| `effort` | `string` | — | Nível de esforço de Thinking, como `low`, `medium`, `high`, `xhigh` ou `max`. Provedores que não sejam Kimi não remapeiam valores concretos quando o protocolo upstream os aceita; se o provedor rejeitar o valor, escolha um aceito pelo modelo. Protocolos que expõem apenas níveis ou orçamentos de tokens ainda fazem a conversão de formato necessária. Modelos Kimi com `support_efforts` usam o padrão do modelo como fallback quando o valor configurado não estiver na lista; modelos Kimi sem essa lista tratam qualquer valor habilitado como `on` booleano |
| `keep` | `string` | `"all"` | Passagem preservada de Thinking. Em `kimi`, é enviada como `thinking.keep`; em `anthropic`, incluindo Claude e o modo compatível com Anthropic do Kimi, é enviada como uma edição `clear_thinking_20251015` de `context_management`. Habilitar `keep` direciona requisições Anthropic para a API beta Messages; um valor de desligamento desabilita `keep` e retorna ao endpoint padrão. `"all"` preserva o raciocínio dos turnos anteriores, como `reasoning_content` ou blocos de thinking da Anthropic. Use um valor de desligamento (`false`/`0`/`no`/`off`/`none`/`null`) para desabilitar. O campo é sobrescrito por `KIMI_MODEL_THINKING_KEEP` e só é injetado enquanto Thinking estiver ativo |

### Campos obsoletos

| Campo | Obsoleto desde | Descrição |
| --- | --- | --- |
| `default_thinking` | 0.21.0 | Booleano de nível superior substituído por `[thinking] enabled`. Migre `default_thinking = true` para `enabled = true` e `default_thinking = false` para `enabled = false` |
| `thinking.mode` | 0.21.0 | Um de `auto` / `on` / `off`, substituído por `[thinking] enabled`. `mode = "off"` vira `enabled = false`; `mode = "on"` e `mode = "auto"` equivalem a `enabled = true`, que é o padrão, e podem ser removidos |
| `loop_control.max_retries_per_step` | 0.32.0 | Substituído por `loop_control.max_attempts_per_step`; o valor sempre foi um limite total de tentativas, incluindo a primeira. A chave antiga é ignorada e gera um aviso na inicialização; renomeie-a em `config.toml` |
| `loop_control.max_steps_per_run` | 0.32.0 | Substituído por `loop_control.max_steps_per_turn`. A chave antiga é ignorada e gera um aviso na inicialização; renomeie-a em `config.toml` |

## `loop_control`

`loop_control` controla o limite de etapas, o limite de tentativas por etapa e o limiar que aciona a compactação automática do contexto no loop de execução do agente.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `max_steps_per_turn` | `integer` | — | Máximo de etapas por turno; ausente ou `0` significa ilimitado |
| `max_attempts_per_step` | `integer` | `10` | Máximo total de tentativas para uma etapa com falha, incluindo a tentativa inicial |
| `reserved_context_size` | `integer` | — | Número de tokens reservados para a saída do modelo; a compactação automática é acionada quando a janela de contexto restante fica abaixo desse valor |

`max_steps_per_turn` pode ser sobrescrito por `KIMI_LOOP_MAX_STEPS_PER_TURN`, e `max_attempts_per_step` por `KIMI_LOOP_MAX_ATTEMPTS_PER_STEP`; ambas as variáveis têm prioridade sobre o arquivo de configuração. A variável antiga `KIMI_LOOP_MAX_RETRIES_PER_STEP` está obsoleta, mas ainda é respeitada, com aviso na inicialização, quando a nova não estiver definida.

As novas tentativas só se aplicam a falhas transitórias, como erros de conexão, timeouts, limites de taxa HTTP 429 e erros 5xx do servidor. Um 429 causado por cota esgotada ou saldo insuficiente não é repetido e falha imediatamente, porque não pode funcionar até a conta ser recarregada.

## `token_counting`

`token_counting` escolhe qual contagem de tokens do contexto é informada externamente, isto é, o valor exibido como tamanho do contexto. A lógica interna, incluindo gatilhos de compactação automática, orçamentos e recuo após estouro de contexto, sempre usa tanto o uso informado pelo provedor quanto estimativas, independentemente dessa configuração.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `strategy` | `"measured+estimated" \| "measured" \| "estimated"` | `"measured+estimated"` | `measured+estimated` informa o tamanho em tempo real, somando o uso informado pelo provedor em cada troca com uma estimativa da parte ainda não medida e nunca ficando abaixo do último total medido; `measured` informa apenas o uso do provedor, então a exibição só muda quando uma troca termina; `estimated` informa apenas uma estimativa e ignora o uso do provedor, servindo como fallback para provedores que não informam uso ou o fazem de forma pouco confiável |

`strategy` pode ser sobrescrito pela variável de ambiente `KIMI_TOKEN_COUNTING_STRATEGY`, que tem prioridade sobre `config.toml`.

## `background`

`background` controla o comportamento de concorrência das tarefas em segundo plano, iniciadas pela ferramenta `Bash` ou pelo parâmetro `run_in_background=true` da ferramenta `Agent`.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `max_running_tasks` | `integer` | — | Número máximo de tarefas em segundo plano executadas simultaneamente |
| `keep_alive_on_exit` | `boolean` | `false` | Define se tarefas ainda em execução continuam ativas quando a sessão é encerrada. Por padrão, o Kimi Code solicita que todas as tarefas em segundo plano parem antes do processo terminar; use `true` apenas quando quiser que elas sobrevivam à sessão. No modo print (`kimi -p`), esse campo é apenas um fallback legado usado quando `print_background_mode` não está definido: `true` equivale a `print_background_mode = "drain"` |
| `kill_grace_period_ms` | `integer` | `5000` | Período de tolerância em milissegundos depois que o encerramento da sessão, uma parada manual ou o timeout de uma tarefa solicita o término gracioso. Se a tarefa continuar executando após esse período, o Kimi Code tenta encerrá-la à força |
| `bash_auto_background_on_timeout` | `boolean` | `true` | Quando um comando `Bash` em primeiro plano atinge o timeout, move-o para uma tarefa em segundo plano em vez de encerrá-lo. O agente é avisado quando ele termina, e o comando movido fica limitado pelo timeout padrão `bash_task_timeout_s`. Use `false` para encerrar comandos em primeiro plano que atinjam o timeout |
| `bash_task_timeout_s` | `integer` | `600` | Timeout padrão, em segundos, para tarefas `Bash` em segundo plano quando a chamada omite `timeout`; também é usado para redefinir o limite de comandos em primeiro plano movidos para segundo plano após timeout. `0` significa sem timeout: a tarefa executa até terminar ou até o modelo interrompê-la. Valores explícitos de `timeout` por chamada não são afetados. No modo print (`kimi -p`), o padrão é `0` quando o campo não é definido explicitamente |
| `print_background_mode` | `"exit" \| "drain" \| "steer"` | `"steer"` | Somente no modo print (`kimi -p`). Controla como tarefas pendentes em segundo plano são tratadas quando o turno do agente principal termina: `"exit"` encerra imediatamente; `"drain"` aguarda todas as tarefas chegarem a um estado final antes de encerrar, sem devolver os resultados ao agente principal; `"steer"` mantém o processo ativo para que uma tarefa concluída, como um subagente em segundo plano, injete uma mensagem sintética de usuário que conduz o agente principal a um novo turno. O ciclo continua até um turno terminar sem tarefas pendentes ou até um limite ser atingido. Tem prioridade sobre o fallback `keep_alive_on_exit` do modo print |
| `print_wait_ceiling_s` | `integer` | `2147483` | No modo print (`kimi -p`), limite de tempo de relógio, em segundos, para o ciclo de espera/direcionamento quando `print_background_mode` é `"drain"` ou `"steer"`. O padrão é cerca de 24,8 dias, efetivamente sem limite. Não tem efeito fora do modo print nem quando o valor é `"exit"` |
| `print_max_turns` | `integer` | `100000` | No modo print (`kimi -p`) com `print_background_mode = "steer"`, número máximo de novos turnos que podem ser acionados por conclusões de tarefas em segundo plano, para manter o ciclo limitado; o padrão é efetivamente sem limite |

`keep_alive_on_exit` pode ser sobrescrito por `KIMI_CODE_BACKGROUND_KEEP_ALIVE_ON_EXIT` e `max_running_tasks` por `KIMI_CODE_BACKGROUND_MAX_RUNNING_TASKS`; ambas as variáveis têm prioridade sobre `config.toml`.

No modo print (`kimi -p "<prompt>"`), o Kimi Code permanece ativo depois do turno do agente principal enquanto houver tarefas em segundo plano pendentes. Cada conclusão é devolvida ao agente principal como uma mensagem sintética de usuário, conduzindo-o a um novo turno, com `print_background_mode = "steer"` por padrão. A execução termina quando um turno acaba sem nada pendente. O ciclo é limitado por `print_wait_ceiling_s` e `print_max_turns`, ambos efetivamente ilimitados por padrão. O trabalho em segundo plano também nunca é encerrado por um limite global de tempo no modo print: tarefas `Bash` em segundo plano têm, por padrão, nenhum timeout (`bash_task_timeout_s = 0`), e subagentes executam sem timeout (`[subagent] timeout_ms = 0`), então apenas o próprio modelo interrompe uma tarefa. Use `print_background_mode = "drain"` para aguardar tarefas sem devolver os resultados ao agente ou `"exit"` para encerrar a execução assim que o agente principal terminar.

## `subagent`

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `timeout_ms` | `integer` | `7200000` (2 horas) | Tempo máximo de relógio, em milissegundos, durante o qual um único subagente (`Agent` / `AgentSwarm`) pode executar antes de ser marcado como `timed_out`. `0` significa sem timeout: o subagente executa até terminar ou até o modelo interrompê-lo. Esse é o timeout por tarefa do gerenciador de tarefas em segundo plano para cada subagente e, portanto, vale para subagentes em primeiro ou segundo plano. No modo print (`kimi -p`), o padrão é `0` quando o campo não é definido explicitamente. Valores acima de `2147483647`, cerca de 24,8 dias, são limitados pelo runtime a aproximadamente 24,8 dias |

`timeout_ms` pode ser sobrescrito pela variável de ambiente `KIMI_SUBAGENT_TIMEOUT_MS`, que tem prioridade sobre `config.toml`.

## `mcp`

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `startup_timeout_ms` | `integer` | `30000` (30 segundos) | Timeout global padrão de conexão, incluindo inicialização e descoberta de ferramentas, em milissegundos para todos os servidores MCP. Aceita `1`–`2147483647`. Um `startupTimeoutMs` por servidor em `mcp.json` sempre prevalece sobre esta seção e a variável de ambiente; quando nenhum dos dois é definido, vale o padrão |
| `tool_timeout_ms` | `integer` | `60000` (60 segundos) | Timeout global padrão de uma única chamada de ferramenta, em milissegundos, para todos os servidores MCP. Aceita `1`–`2147483647`. Um `toolTimeoutMs` por servidor em `mcp.json` sempre prevalece sobre esta seção e a variável de ambiente; quando nenhum dos dois é definido, vale o padrão integrado do cliente |

`startup_timeout_ms` e `tool_timeout_ms` podem ser sobrescritos pelas variáveis de ambiente `KIMI_MCP_STARTUP_TIMEOUT_MS` e `KIMI_MCP_TOOL_TIMEOUT_MS`, respectivamente, que têm prioridade sobre `config.toml`. Consulte [MCP](../customization/mcp.md) para a configuração completa de servidores MCP.

## `identity`

Personaliza a forma como o agente se identifica. Se esta seção não for definida, nada muda.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `name` | `string` | — | Nome de exibição que o agente usa para se referir a si mesmo no prompt do sistema; preenche o espaço `${product_name}`, inclusive no seu próprio `SYSTEM.md` e nos arquivos de agente |
| `slug` | `string` | derivado de `name` | Identificador de máquina usado em campos de protocolo: o token de produto `User-Agent` enviado a provedores de terceiros e o nome do cliente anunciado aos servidores MCP. Quando omitido, é derivado de `name`: convertido para minúsculas e com cada sequência de caracteres não alfanuméricos substituída por `-` |

```toml
[identity]
name = "Acme Dev Agent"
slug = "acme-dev"        # optional
```

Ambos os campos podem ser definidos pelas variáveis de ambiente `KIMI_CODE_IDENTITY_NAME` e `KIMI_CODE_IDENTITY_SLUG`, que têm prioridade sobre `config.toml` e nunca são gravadas de volta nele. Isso é útil em containers e CI, onde escrever um arquivo de configuração pode ser inconveniente.

Um nome sem letras ou dígitos ASCII, por exemplo um nome escrito apenas em chinês, não deixa conteúdo a partir do qual derivar um slug e usa `agent` como fallback. Defina `slug` explicitamente quando precisar de um token de protocolo específico.

A identidade é resolvida uma única vez na inicialização e permanece fixa durante toda a vida do processo. Ela é anunciada a servidores MCP e provedores durante a conexão, portanto não pode mudar no meio da execução. Alterações nesta seção entram em vigor na próxima inicialização e em novas sessões. Uma sessão retomada mantém o prompt do sistema com o qual foi registrada, já que seus turnos anteriores ocorreram sob aquela identidade. Da mesma forma, uma autorização OAuth MCP mantém o registro de cliente concedido sob a identidade anterior; redefina a autenticação desse servidor para registrá-lo sob a nova identidade.

Esta seção é lida pelo mecanismo padrão `agent-core-v2`. Ela é ignorada pelo caminho legado `kimi` / `kimi -p` selecionado com `KIMI_CODE_LEGACY_FLAG=1`; `kimi web` sempre usa `agent-core-v2`.

## `tools`

`tools` é o controle global de ferramentas: ele se aplica a todos os agentes em todas as sessões e se combina com as políticas `tools` / `disallowedTools` de cada agente.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `enabled` | `array<string>` | — | Lista global de permissões: quando não vazia, apenas as ferramentas listadas ficam disponíveis; omitir o campo ou usar uma lista vazia não impõe restrição |
| `disabled` | `array<string>` | — | Lista global de bloqueio, aplicada depois de `enabled` |

A correspondência de nomes segue as mesmas regras dos campos equivalentes em um arquivo de agente: ferramentas integradas são comparadas pelo nome exato, como `Read`, enquanto ferramentas MCP usam padrões glob, como `mcp__github__*`. Três formatos de entrada nunca correspondem a nada e geram um aviso: um curinga fora de um padrão `mcp__` (`enabled = ["*"]` desabilita todas as ferramentas e `disabled = ["*"]` não desabilita nenhuma), um literal `mcp__` sem o segmento da ferramenta (`mcp__github`; use `mcp__github__*` para um servidor inteiro) e um nome que não corresponda a nenhuma ferramenta registrada ou integrada. A correspondência diferencia maiúsculas de minúsculas.

```toml
[tools]
disabled = ["EnterPlanMode", "ExitPlanMode", "mcp__github__*"]
```

::: warning Atenção
Assim como os campos `tools` / `disallowedTools` de um arquivo de agente, esta seção define as ferramentas apresentadas ao modelo e é aplicada novamente antes da execução. As [regras de permissão](#permission) continuam sendo um controle separado para operações que exigem aprovação.
:::

## `image`

`image` controla como as imagens são compactadas antes de serem enviadas ao modelo em todos os pontos de entrada, incluindo imagens coladas, leituras com `ReadMediaFile`, imagens presentes em resultados de ferramentas MCP e outros casos.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `max_edge_px` | `integer` | `2000` | Limite em pixels para a maior dimensão. Imagens maiores são reduzidas proporcionalmente para caber; aumentar o valor preserva mais detalhes ao custo de corpos de requisição maiores |
| `read_byte_budget` | `integer` | `262144` (256 KB) | Orçamento de bytes por imagem para imagens que o modelo lê por conta própria, como leituras padrão de `ReadMediaFile`. Limita o tamanho acumulado do corpo da requisição quando o modelo continua capturando telas e lendo imagens. Detalhes finos continuam acessíveis pelo parâmetro `region`, que lê um recorte em fidelidade total; `region` e `full_resolution` não estão sujeitos a esse orçamento |

`max_edge_px` pode ser sobrescrito por `KIMI_IMAGE_MAX_EDGE_PX` e `read_byte_budget` por `KIMI_IMAGE_READ_BYTE_BUDGET`; ambas as variáveis têm prioridade sobre `config.toml`.

<!--
## `experimental`

`experimental` armazena sobrescritas persistentes para flags de recursos experimentais. Atualmente, `micro_compaction` é a única entrada voltada ao usuário e usa `false` por padrão; defina-a como `true` para habilitar o corte automático de resultados antigos e grandes de ferramentas.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `micro_compaction` | `boolean` | `false` | Remove do contexto resultados antigos e grandes de ferramentas, preservando a conversa recente |
-->

## `services`

`services` configura dois serviços integrados: pesquisa na web (`moonshot_search`) e busca de conteúdo web (`moonshot_fetch`). Apenas essas duas chaves fixas são reconhecidas; outras chaves são ignoradas. As duas entradas compartilham os mesmos campos:

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `base_url` | `string` | Não | URL da API do serviço |
| `api_key` | `string` | Não | Chave de API |
| `oauth` | `table` | Não | Referência de credenciais OAuth, com a mesma estrutura de `providers.*.oauth` |
| `custom_headers` | `table<string, string>` | Não | Cabeçalhos HTTP personalizados adicionados a cada requisição |

`base_url` e `api_key` também podem vir de variáveis de ambiente, que têm prioridade sobre o arquivo de configuração: `KIMI_WEB_SEARCH_BASE_URL` / `KIMI_WEB_SEARCH_API_KEY` para `moonshot_search` e `KIMI_WEB_FETCH_BASE_URL` / `KIMI_WEB_FETCH_API_KEY` para `moonshot_fetch`. Uma URL-base definida por variável de ambiente cria um endpoint de serviço separado, portanto a chave de API persistida, a referência OAuth e os cabeçalhos personalizados não são encaminhados a ele; defina a chave de API correspondente no ambiente quando esse endpoint exigir autenticação. Uma chave de API no ambiente sem URL-base no ambiente mantém o endpoint e os cabeçalhos personalizados configurados, mas substitui as duas formas de credencial configuradas. Definir URL-base e chave de API pelo ambiente sem nenhuma seção de configuração também habilita o serviço.

```toml
[services.moonshot_search]
base_url = "https://api.moonshot.cn/v1/search"
api_key = "sk-xxx"

[services.moonshot_fetch]
base_url = "https://api.moonshot.cn/v1/fetch"
api_key = "sk-xxx"
```

## `permission`

`permission` define regras de permissão carregadas automaticamente quando uma sessão começa, controlando se o agente precisa da confirmação do usuário antes de chamar uma ferramenta. As regras são escritas como um array de tabelas `[[permission.rules]]` e comparadas em ordem; a primeira regra correspondente entra em vigor.

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `decision` | `string` | Sim | Ação quando houver correspondência: `allow` (permitir imediatamente), `deny` (rejeitar imediatamente), `ask` (perguntar a cada vez) |
| `scope` | `string` | Não | Escopo da regra: `turn-override`, `session-runtime`, `project`, `user`; o padrão é `user` |
| `pattern` | `string` | Sim | Padrão de correspondência no formato `ToolName` ou `ToolName(arg-pattern)`, por exemplo `Read` ou `Bash(rm -rf*)` |
| `reason` | `string` | Não | Descrição da regra para depuração e auditoria |

Os nomes das ferramentas integradas estão listados em [Ferramentas integradas](../../en/reference/tools.md). A maioria das ferramentas integradas que aceita argumentos em regras define o próprio objeto de correspondência, como `Bash(command-pattern)` ou `Read(path-pattern)`. `AgentSwarm`, ferramentas MCP e ferramentas personalizadas só podem ser comparadas pelo nome da ferramenta; padrões de argumentos não são aceitos nesses casos.

```toml
[[permission.rules]]
decision = "allow"
pattern = "Read"

[[permission.rules]]
decision = "allow"
pattern = "Grep"

[[permission.rules]]
decision = "deny"
pattern = "Bash(rm -rf*)"

[[permission.rules]]
decision = "ask"
pattern = "Bash"
```

::: tip Dica
As declarações de servidores MCP são configuradas em `~/.kimi-code/mcp.json` ou no arquivo `.kimi-code/mcp.json` do projeto, e não em `config.toml`. O ponto de entrada da configuração interativa é `/mcp-config`; consulte [Model Context Protocol](../customization/mcp.md).
:::

## `tui.toml`

Além de `config.toml`, a CLI mantém as preferências da interface do terminal e do cliente em um arquivo complementar `tui.toml` no mesmo diretório (`~/.kimi-code/tui.toml`, ou `$KIMI_CODE_HOME/tui.toml` quando sobrescrito). Ele é criado com os valores padrão na primeira execução, e os comandos interativos `/config`, `/theme` e `/editor` gravam nele para você, então raramente é necessário editá-lo manualmente. Se o arquivo estiver malformado, a CLI usa os padrões e mostra um aviso em vez de falhar na inicialização.

| Campo | Tipo | Padrão | Descrição |
| --- | --- | --- | --- |
| `theme` | `string` | `auto` | Tema de cores: `auto` (segue o terminal), `dark`, `light` ou o nome de um [tema personalizado](../../en/customization/themes.md) |
| `disable_paste_burst` | `boolean` | `false` | Desabilita o fallback para colagens rápidas sem bracketed paste, evitando que colagens multilinha muito rápidas sejam enviadas linha por linha |
| `cache_expiry_hint` | `boolean` | `true` | Exibe um diálogo ao retomar uma sessão ociosa há muito tempo ou ao enviar algo após um longo período de inatividade, avisando que o cache de contexto provavelmente expirou e oferecendo compactar o contexto ou iniciar uma nova sessão; somente no mecanismo v2 |
| `[editor].command` | `string` | `""` | Comando do editor externo usado para escrever entradas longas; quando vazio, usa `$VISUAL` / `$EDITOR` |
| `[notifications].enabled` | `boolean` | `true` | Define se notificações da área de trabalho são enviadas |
| `[notifications].notification_condition` | `string` | `unfocused` | Quando notificar: `unfocused` (somente quando o terminal não estiver em foco) ou `always` |
| `[upgrade].auto_install` | `boolean` | `true` | Define se novas versões são instaladas automaticamente |
| `[status_line].items` | `string[]` | `[]` | Componentes integrados exibidos na primeira linha do rodapé e sua ordem: `mode`, `goal`, `model`, `tasks`, `cwd`, `git`, `tips`. Quando não definido, mantém o layout padrão; identificadores desconhecidos são ignorados com um aviso |
| `[status_line].command` | `string` | `""` | Comando personalizado da linha de status. Sua primeira linha de stdout substitui a primeira linha do rodapé, recebendo via stdin um snapshot JSON com modelo, cwd, branch git, modo de permissão, modo Plan, uso de contexto, id da sessão e versão. As execuções são limitadas a 300 ms e a uma vez por segundo; falhas usam o layout integrado como fallback |

```toml
# ~/.kimi-code/tui.toml
theme = "auto" # "auto" | "dark" | "light" | custom theme name
disable_paste_burst = false # true disables non-bracketed paste-burst fallback
cache_expiry_hint = true # false disables the "cache expired" dialog on resume / idle submit

[editor]
command = "" # empty uses $VISUAL / $EDITOR

[notifications]
enabled = true
notification_condition = "unfocused" # "unfocused" | "always"

[upgrade]
auto_install = true

# [status_line]
# items = ["mode", "goal", "model", "tasks", "cwd", "git", "tips"]
# command = "~/.kimi-code/statusline.sh"
```

As alterações entram em vigor na próxima inicialização ou imediatamente com `/reload-tui`, que recarrega apenas `tui.toml`; `/reload` recarrega `config.toml` e `tui.toml`.

## Configuração local do projeto

Além dos arquivos de nível do usuário em `~/.kimi-code`, o Kimi Code lê um arquivo de configuração local do projeto em `<project-root>/.kimi-code/local.toml`. Ele armazena configurações específicas de um checkout do projeto e, normalmente, não deve ser compartilhado com colegas de equipe.

O arquivo é criado automaticamente quando você adiciona um diretório extra ao workspace com [`/add-dir`](../../en/reference/slash-commands.md) e escolhe lembrar esse diretório para o projeto. Raramente é necessário editá-lo manualmente.

### `[workspace]`

A tabela `[workspace]` agrupa configurações de workspace no nível do projeto:

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `additional_dir` | `array<string>` | Não | Diretórios adicionais do workspace, armazenados como caminhos absolutos. Gravado automaticamente quando você confirma "remember this directory" em `/add-dir`; lido novamente na inicialização para que os diretórios fiquem disponíveis em todas as sessões deste projeto |

```toml
[workspace]
additional_dir = ["/absolute/path/to/shared"]
```

Como os diretórios são armazenados em caminhos absolutos específicos da sua máquina, recomendamos adicionar `.kimi-code/local.toml` ao `.gitignore` do projeto para evitar que seja commitado.

## Próximos passos

- [Provedores e modelos](../../en/configuration/providers.md): exemplos de conexão para cada tipo de provedor (Kimi, Claude, OpenAI, Gemini)
- [Sobrescritas de configuração](../../en/configuration/overrides.md): regras de prioridade para opções da CLI, arquivo de configuração e variáveis de ambiente
- [Variáveis de ambiente](../../en/configuration/env-vars.md): lista completa de variáveis de runtime como `KIMI_CODE_HOME`
