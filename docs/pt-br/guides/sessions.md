# Sessões e contexto

O Kimi Code CLI mantém cada conversa como uma “sessão”, armazenando o histórico de mensagens e os metadados para que você possa fechar o terminal e continuar exatamente de onde parou. Esta página explica como retomar sessões, gerenciar o contexto e exportar ou criar forks de sessões.

## Armazenamento das sessões

Todas as sessões são salvas em `$KIMI_CODE_HOME/sessions/` (por padrão, `~/.kimi-code/sessions/`) e agrupadas por diretório de trabalho:

```text
~/.kimi-code/
├── config.toml
├── session_index.jsonl
└── sessions/
    └── <workDirKey>/
        └── <sessionId>/
            ├── state.json
            └── agents/
                ├── main/
                │   └── wire.jsonl
                └── <subagentId>/
                    └── wire.jsonl
```

- `state.json`: contém os metadados da sessão, como o título e a data de criação.
- `agents/*/wire.jsonl`: contém o fluxo de eventos do agente, usado para recuperar e reproduzir sessões. Também inclui um rastreamento da requisição com os esquemas das ferramentas, os parâmetros da requisição e as listas de ferramentas MCP enviados ao modelo para fins de depuração.

::: warning Atenção
Não edite manualmente os arquivos do diretório `sessions/`. Isso pode impedir que as sessões sejam restauradas corretamente.
:::

## Iniciar e retomar sessões

Sempre que você executa `kimi` diretamente, uma nova sessão é criada. Para retomar uma sessão anterior, use uma das opções abaixo.

**Retomar a sessão mais recente no diretório atual:**

```sh
kimi --continue
```

**Retomar uma sessão específica pelo ID:**

```sh
kimi --session abc123
```

**Navegar interativamente pelo histórico e escolher uma sessão:**

```sh
kimi --session
```

::: warning Atenção
As opções `--continue` e `--session` são mutuamente exclusivas.
:::

## Alternar sessões dentro da TUI

Você pode gerenciar sessões sem sair do terminal. Os comandos de barra abaixo ficam disponíveis somente quando o agente está ocioso:

- **`/new`** (alias `/clear`): muda para uma nova sessão e descarta o contexto atual.
- **`/sessions`** (alias `/resume`): permite navegar e retomar uma sessão anterior.
- **`/fork`**: cria um fork da sessão atual, conforme explicado abaixo.
- **`/title <text>`** (alias `/rename`): define um título para facilitar a identificação da sessão. Quando usado sem argumentos, exibe o título atual.

## Compactação do contexto

Conforme a conversa aumenta, o Kimi Code CLI compacta automaticamente o histórico de mensagens quando o contexto se aproxima do limite da janela, liberando espaço para novos tokens. Você também pode iniciar a compactação manualmente a qualquer momento:

```
/compact
```

Você pode acrescentar uma orientação para indicar ao modelo quais informações devem ser priorizadas durante a compactação:

```
/compact Preserve a discussão sobre migrações de banco de dados
```

## Criar um fork de uma sessão

Para explorar outra direção sem alterar a conversa atual, use `/fork`:

```
/fork
```

A criação do fork não muda a sessão em que você está. Você permanece na sessão original e a conversa continua sem alterações. O fork é uma cópia independente para a qual você pode mudar a qualquer momento usando `/sessions`.

Uma meta salva com `/goal` não é copiada para o fork. Para realizar um novo trabalho autônomo baseado em metas nessa sessão, inicie uma nova meta dentro dela.

## Exportar uma sessão

Use `kimi export` para empacotar uma sessão em um arquivo ZIP. Isso é útil para compartilhamento, arquivamento ou envio de um relatório de bug:

```sh
kimi export <sessionId>
```

Quando `sessionId` é omitido, o comando exporta a sessão mais recente no diretório atual e solicita uma confirmação interativa. Adicione `-y` para ignorar essa confirmação. Use `-o` para definir o caminho de saída:

```sh
kimi export <sessionId> -o ~/Desktop/my-session.zip
```

A exportação inclui todos os arquivos do diretório da sessão, incluindo os logs de diagnóstico. Por padrão, o log global de diagnóstico (`~/.kimi-code/logs/kimi-code.log`) também é incluído. Adicione `--no-include-global-log` para excluí-lo.

Você também pode exportar uma sessão de dentro da TUI, sem sair da interface interativa:

- **`/export-debug-zip`**: produz o mesmo ZIP de depuração que `kimi export`.
- **`/export-md`** (alias `/export`): exporta a conversa como um arquivo Markdown legível, adequado para compartilhamento ou arquivamento. O comando aceita um caminho opcional. Sem esse argumento, o arquivo é salvo como `kimi-export-<short-id>-<timestamp>.md` no diretório de trabalho atual.

Na interface web, `/export` baixa a sessão atual como um ZIP de diagnóstico. O arquivo contém os dados persistidos da sessão, os logs de diagnóstico e um registro limitado, contendo somente metadados, dos principais eventos do navegador em `logs/kimi-web.jsonl`. O texto dos prompts, o conteúdo das mensagens WebSocket e os argumentos do console não são copiados para esse log do navegador.

O comportamento desse comando na interface web é diferente do alias `/export` da TUI descrito acima.

O navegador mantém o ZIP em memória antes de salvá-lo. Por isso, as exportações pela interface web são limitadas a 64 MiB. Para exportar uma sessão maior, use `kimi export <sessionId>` ou o comando `/export-debug-zip` na TUI.

::: tip Dica
Os arquivos exportados podem conter código, saídas de comandos e caminhos de arquivos com informações sensíveis. Revise o conteúdo antes de compartilhá-lo.
:::

## Próximos passos

- [Locais de armazenamento](../../en/configuration/data-locations.md): estrutura completa dos diretórios usados pelos arquivos de sessão.
- [Referência do comando `kimi`](../../en/reference/kimi-command.md): referência completa dos parâmetros de `--continue`, `--session`, `export` e outros comandos.
