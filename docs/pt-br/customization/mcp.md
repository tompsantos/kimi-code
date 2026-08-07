# Model Context Protocol

O [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) é um protocolo aberto que permite que modelos chamem com segurança ferramentas expostas por processos ou serviços externos. Isso inclui, por exemplo, ler issues do GitHub, consultar bancos de dados ou operar o sistema de arquivos local. O Kimi Code CLI atua como um cliente MCP para conectar essas ferramentas externas e disponibilizá-las ao agente junto com as ferramentas integradas (`Read`, `Bash`, `Grep` etc.), sem diferença de comportamento.

## Métodos de conexão

O Kimi Code CLI oferece três métodos de conexão com servidores MCP:

- **stdio**: a CLI inicia o servidor MCP local como um processo filho e se comunica por entrada e saída padrão. É adequado para ferramentas locais de linha de comando.
- **HTTP**: a CLI se conecta a um endpoint HTTP que já está em execução. É adequado para serviços remotos ou processos que precisam permanecer ativos.
- **SSE**: a CLI se conecta a um endpoint legado HTTP+SSE (Server-Sent Events, um mecanismo HTTP de streaming). Prefira HTTP para novos servidores MCP, mas use `transport: "sse"` quando o serviço ainda oferecer somente o transporte SSE antigo.

## Configuração

A configuração dos servidores MCP é armazenada em `mcp.json`, em dois níveis:

- **Nível do usuário**: `~/.kimi-code/mcp.json` (ou `$KIMI_CODE_HOME/mcp.json`), compartilhado entre projetos
- **Nível do projeto**: `.kimi-code/mcp.json` no diretório de trabalho, válido somente para o repositório atual

Quando houver entradas com o mesmo nome, a entrada do nível do projeto tem precedência e sobrescreve a entrada do nível do usuário.

Execute `/mcp-config` na TUI para adicionar, editar ou excluir servidores de forma interativa, sem editar manualmente o arquivo JSON. Execute `/mcp` para consultar o status de conexão de todos os servidores atuais.

Excluir um servidor da configuração não interrompe sessões já abertas. O servidor continua aparecendo em `/mcp` com o status `removed` ("removido"), suas ferramentas continuam visíveis ali e chamadas a elas falham com um aviso de remoção. Novas sessões, porém, não registram essas ferramentas. Da mesma forma, um servidor adicionado durante uma sessão, seja pela edição de `mcp.json` ou pela instalação de um plugin, não é registrado nas sessões já abertas. Ele passa a fazer parte somente das sessões criadas depois.

Estrutura de `mcp.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    },
    "linear": {
      "url": "https://mcp.linear.app/mcp"
    },
    "legacy-events": {
      "transport": "sse",
      "url": "https://mcp.example.com/sse"
    }
  }
}
```

Entradas com o campo `command` representam servidores stdio. Entradas com o campo `url` e sem `transport` representam servidores HTTP. Para servidores SSE legados, defina explicitamente `transport` como `"sse"`.

Campos opcionais:

| Campo | Tipo | Aplicável a | Descrição |
| --- | --- | --- | --- |
| `env` | `Record<string, string>` | stdio | Variáveis de ambiente injetadas no processo filho |
| `cwd` | `string` | stdio | Diretório de trabalho do processo filho |
| `headers` | `Record<string, string>` | HTTP, SSE | Cabeçalhos de requisição estáticos adicionados a todas as requisições |
| `bearerTokenEnvVar` | `string` | HTTP, SSE | Nome de uma variável de ambiente que contém um token Bearer |
| `enabled` | `boolean` | Todos | Defina como `false` para desabilitar o servidor |
| `startupTimeoutMs` | `number` | Todos | Tempo limite de conexão entre `1` e `2147483647` milissegundos; padrão `30000` |
| `toolTimeoutMs` | `number` | Todos | Tempo limite entre `1` e `2147483647` milissegundos para uma única chamada de ferramenta |
| `enabledTools` | `string[]` | Todos | Lista de permissões de ferramentas |
| `disabledTools` | `string[]` | Todos | Lista de bloqueio de ferramentas |

Não é obrigatório definir o tempo limite de conexão nem o tempo limite de uma chamada individual para cada servidor. `[mcp] startup_timeout_ms` / `[mcp] tool_timeout_ms` em `config.toml`, ou as variáveis de ambiente `KIMI_MCP_STARTUP_TIMEOUT_MS` / `KIMI_MCP_TOOL_TIMEOUT_MS`, alteram os valores padrão globais. A precedência é: campo por servidor > variável de ambiente > `config.toml` > padrão integrado. Consulte [Arquivos de configuração](../../en/configuration/config-files.md#mcp).

Servidores HTTP e SSE permitem fornecer credenciais estáticas por `headers` ou `bearerTokenEnvVar`. Quando OAuth for necessário, execute `/mcp-config login <server-name>` para concluir a autorização pelo navegador.

Plugins também podem declarar servidores MCP no próprio manifesto. Servidores declarados por um plugin ficam habilitados por padrão e podem ser desabilitados ou habilitados novamente em `/plugins`. Desabilitar ou remover um servidor interrompe suas ferramentas em sessões abertas, fazendo com que as chamadas falhem com um aviso de remoção. Adicionar ou habilitar um servidor passa a valer em novas sessões ou após `/reload`. Consulte [Plugins](../../en/customization/plugins.md#mcp-servers-in-plugins) para mais detalhes.

::: warning Atenção
Entradas stdio em um `.kimi-code/mcp.json` no nível do projeto executam comandos locais quando uma sessão é iniciada. Habilite essas entradas somente em repositórios nos quais você confia.
:::

## Nomes de ferramentas e permissões

Ferramentas MCP usam o formato de nome `mcp__<server>__<tool>`, por exemplo `mcp__github__create_issue`. Regras de permissão aceitam os curingas `*` e `**`. Por exemplo, `mcp__github__*` corresponde a todas as ferramentas desse servidor. Os parâmetros das ferramentas MCP não participam da correspondência das regras de permissão.

Chamadas que não correspondem a nenhuma regra de permissão geram uma solicitação de aprovação. Selecionar **Approve for this session** ("aprovar nesta sessão") no painel de aprovação permite automaticamente chamadas posteriores do mesmo tipo durante a sessão atual.

Você também pode pré-configurar regras permanentes em `[[permission.rules]]` dentro de `config.toml`:

```toml
[[permission.rules]]
decision = "allow"
pattern = "mcp__github__*"

[[permission.rules]]
decision = "deny"
pattern = "mcp__filesystem__write_file"
```

Para consultar a sintaxe completa das regras de permissão, veja [Arquivos de configuração](../../en/configuration/config-files.md#permission).

## Segurança

Ao conectar servidores MCP externos, observe estes cuidados:

- conecte-se somente a servidores de fontes confiáveis;
- verifique se os nomes e parâmetros das ferramentas parecem adequados nas solicitações de aprovação;
- mantenha a aprovação manual para ferramentas de alto risco, como gravação de arquivos e execução de comandos. Evite usar curingas como `mcp__*` para permitir todas as ferramentas de uma só vez.

::: warning Atenção
No modo YOLO, chamadas de ferramentas MCP são aprovadas automaticamente. Use esse modo somente quando confiar totalmente nos servidores MCP conectados.
:::

## Próximos passos

- [Plugins](../../en/customization/plugins.md): declare servidores MCP no manifesto de um plugin para empacotá-los e distribuí-los em conjunto
- [Arquivos de configuração](../../en/configuration/config-files.md#permission): referência completa dos campos das regras de permissão
