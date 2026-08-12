# Sobrescritas de configuração

O Kimi Code CLI tem três lugares em que parâmetros de runtime podem ser influenciados: o arquivo de configuração, as opções de linha de comando e as variáveis de ambiente. A relação entre eles não é simplesmente “quem tem maior prioridade vence”; os três atendem a cenários diferentes e possuem escopos que não se sobrepõem completamente:

- **Arquivo de configuração**: armazena preferências de longo prazo, como modelo, chaves e controle de loop; entra em vigor em todas as inicializações
- **Opções de linha de comando**: fazem alterações pontuais para a inicialização atual; são descartadas ao sair
- **Variáveis de ambiente**: cuidam principalmente da localização do diretório de dados, da troca de endpoints OAuth e de um pequeno conjunto de chaves de runtime; **não funcionam como um mecanismo geral de fallback para campos de configuração**

Essa distinção é importante: muitos usuários executam `export KIMI_API_KEY=xxx` no shell esperando que a CLI use a chave automaticamente, mas isso não acontece. Consulte [Credenciais de provedores](#credenciais-de-provedores) abaixo para entender o motivo.

## Três funções das variáveis de ambiente

As variáveis de ambiente se dividem em três categorias de acordo com sua função e não podem ser reduzidas a uma única ordem linear de prioridade:

1. **Localizar o arquivo de configuração**: `KIMI_CODE_HOME` define o diretório raiz de dados, fazendo com que o caminho do arquivo de configuração seja `$KIMI_CODE_HOME/config.toml`. Essa etapa acontece antes de todas as outras resoluções e não é um fallback para parâmetros individuais.
2. **Chaves de runtime**: um pequeno conjunto de variáveis, como `KIMI_DISABLE_TELEMETRY`, desativa diretamente o subsistema correspondente. Mesmo que `config.toml` tenha `telemetry = true`, definir essa variável com um valor verdadeiro desativa a telemetria. A semântica é “desativar adicionalmente”, e não uma sobrescrita comum.
3. **Endpoints de runtime e diagnóstico**: variáveis como `KIMI_CODE_OAUTH_HOST`, `KIMI_CODE_BASE_URL` e `KIMI_LOG_LEVEL` são lidas durante a inicialização dos subsistemas de OAuth ou logging. Para a lista completa, consulte [Variáveis de ambiente](./env-vars.md).

## Prioridade para parâmetros comuns de runtime

Para parâmetros comuns de runtime, como alias de modelo, modo Plan, modo YOLO e diretórios de Skills, a prioridade da maior para a menor é:

1. **Opções de linha de comando** (`-m`, `--plan`, `--yolo` etc.): aplicam-se somente à inicialização atual
2. **Arquivo de configuração do usuário** (`~/.kimi-code/config.toml`): armazena preferências de longo prazo

Um pequeno número de variáveis de ambiente sobrescreve explicitamente campos específicos do arquivo de configuração. Por exemplo, `KIMI_CODE_BACKGROUND_KEEP_ALIVE_ON_EXIT` tem prioridade sobre `[background].keep_alive_on_exit`. Essas exceções estão indicadas em [Variáveis de ambiente](./env-vars.md) e nas descrições dos campos correspondentes em [Arquivos de configuração](./config-files.md).

::: warning Atenção
**Parâmetros comuns de runtime não usam variáveis de ambiente do shell como fallback.** Os campos `api_key` / `base_url` dos provedores são lidos somente de `config.toml`, incluindo a subtabela `[providers.<name>.env]`, e não usam como fallback variáveis definidas com `export` no shell. A única exceção é o canal explícito `KIMI_MODEL_*`; consulte a seção **Definir um modelo por variáveis de ambiente (`KIMI_MODEL_*`)** em [Variáveis de ambiente](./env-vars.md).
:::

Atualmente, a CLI lê um único arquivo de configuração no nível do usuário e não possui um mecanismo geral de arquivo de configuração no nível do projeto. Para isolar configurações entre projetos diferentes, aponte `KIMI_CODE_HOME` para diretórios de dados distintos; veja **Cenários comuns** abaixo.

## Credenciais de provedores

As credenciais dos provedores (`api_key`, `base_url`) seguem regras próprias de resolução, separadas da cadeia de prioridade dos parâmetros comuns.

Para um único provedor, as credenciais são resolvidas nesta ordem:

1. `[providers.<name>].api_key`: chave gravada diretamente no arquivo de configuração; maior prioridade
2. A chave correspondente dentro da subtabela `[providers.<name>.env]`, como `KIMI_API_KEY`, `ANTHROPIC_API_KEY` etc.: consultada somente quando `api_key` está vazio
3. Se ambas estiverem ausentes: a inicialização falha com um erro informando que faltam credenciais para o provedor

`base_url` é resolvido da mesma forma: primeiro `[providers.<name>].base_url`, depois a chave `*_BASE_URL` em `[providers.<name>.env]`.

> A subtabela `[providers.<name>.env]` é apenas uma seção TOML dentro do arquivo de configuração. Ela não grava nada no ambiente do shell e só é consultada quando o campo direto correspondente (`api_key` / `base_url`) está vazio.

Para a lista completa dos nomes de chaves de credencial, consulte a seção sobre chaves de credencial dos provedores em [Variáveis de ambiente](./env-vars.md).

## Opções de linha de comando

As opções informadas na inicialização têm a maior prioridade e valem somente para a sessão atual:

| Opção | Efeito |
| --- | --- |
| `-S, --session [id]` | Retomar uma sessão específica; abre a seleção interativa quando nenhum ID é informado |
| `-c, --continue` | Retomar a última sessão do diretório de trabalho atual |
| `-y, --yolo` | Aprovar automaticamente chamadas comuns de ferramentas; o agente ainda pode fazer perguntas |
| `--auto` | Iniciar no modo de permissão Auto: totalmente autônomo, sem perguntas do agente |
| `--plan` | Iniciar no modo Plan |
| `-m, --model <model>` | Usar um alias de modelo específico nesta sessão |
| `-p, --prompt <prompt>` | Executar em modo não interativo: processar um único prompt e sair |
| `--output-format <format>` | Formato de saída do modo `-p`: `text` ou `stream-json` |
| `--skills-dir <dir>` | Substituir os diretórios de Skills descobertos automaticamente; pode ser repetido e vale somente para esta sessão |

Regras de exclusão mútua, que fazem a inicialização falhar quando são violadas:

- `--output-format` só pode ser usado com `-p`
- `--prompt` não pode ser combinado com `--yolo` ou `--plan`
- `--continue` e `--session` não podem ser usados juntos
- fora do modo prompt, `--yolo` e `--plan` não podem ser combinados com `--continue` ou `--session`

::: tip Dica
`--skills-dir` é uma substituição pontual que afeta somente a inicialização atual. Para adicionar diretórios de busca de forma persistente, defina `extra_skill_dirs` em `config.toml`; consulte [Agent Skills](../../en/customization/skills.md).
:::

## Cenários comuns

**Ambiente de teste isolado**: use um diretório de dados separado para não misturar a configuração e as sessões principais:

```sh
KIMI_CODE_HOME="$PWD/.kimi-sandbox" kimi
```

**Chave de teste pontual**: como as credenciais dos provedores são lidas somente do arquivo de configuração, grave a chave de teste na subtabela `env`:

```toml
[providers.kimi.env]
KIMI_API_KEY = "sk-test"
```

**Ignorar aprovações em tarefas em lote**:

```sh
kimi --yolo -p "Renomeie em lote os seguintes arquivos..."
```

**Entrar temporariamente no modo Plan**: para torná-lo permanente, defina `default_plan_mode = true` no arquivo de configuração:

```sh
kimi --plan
```

## Próximos passos

- [Arquivos de configuração](./config-files.md): referência completa de todos os campos configuráveis
- [Variáveis de ambiente](./env-vars.md): lista completa e descrição de `KIMI_CODE_HOME` e das variáveis relacionadas
