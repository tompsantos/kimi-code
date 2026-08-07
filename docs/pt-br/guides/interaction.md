# Interação e entrada

O Kimi Code CLI funciona como uma TUI interativa (interface de usuário no terminal) organizada em torno de três componentes: a caixa de entrada, a área da conversa e a barra de status. Esta página explica como inserir texto, colar mídia, navegar pelo fluxo de aprovação e alternar entre os modos.

## Noções básicas da caixa de entrada

A caixa de entrada aceita texto livre. Pressione `Enter` para enviar ou `Shift-Enter` / `Ctrl-J` para inserir uma nova linha. Quando a caixa de entrada estiver vazia, pressione `↑` / `↓` para navegar pelo histórico de entradas do diretório de trabalho atual, incluindo comandos de shell anteriores.

**Sair da CLI**: pressione `Ctrl-D` com a caixa de entrada vazia, pressione `Ctrl-C` duas vezes enquanto o agente estiver ocioso ou digite `/exit`. Pressionar `Ctrl-C` ou `Esc` durante a saída em streaming interrompe o turno atual, mas não encerra o programa.

## Colar imagens e vídeos

O Kimi Code CLI permite colar imagens e vídeos diretamente na caixa de entrada. Assim, você pode discutir capturas de tela, mockups de interface, diagramas de arquitetura ou demonstrações de código sem precisar primeiro enviar ou converter os arquivos.

**A entrada de vídeo é um recurso diferenciado do Kimi Code**: você pode colar um clipe de vídeo e pedir ao modelo para analisar o conteúdo, o fluxo da interface ou uma demonstração de código.

Como colar:

- **macOS / Linux**: `Ctrl-V`
- **Windows**: `Alt-V`

Depois da colagem, a caixa de entrada mostra um marcador temporário que pode ser editado como texto normal. Ao enviar a mensagem, esse marcador é substituído pelo conteúdo real. Se a área de transferência contiver apenas texto simples, a colagem funciona normalmente. O suporte a mídia depende dos recursos multimodais do modelo atual (`image_in` / `video_in`) e fica habilitado por padrão quando você está conectado a uma conta Kimi Code.

## Comandos de barra

Qualquer entrada que comece com `/` é tratada como um comando de barra. Digitar `/` abre um menu de autocompletar que é filtrado em tempo real enquanto você continua digitando. Pressione `Esc` para fechar o menu. Se nenhum comando corresponder, a entrada será enviada ao agente como uma mensagem normal.

As [Agent Skills](../../en/customization/skills.md) ativas são registradas automaticamente como comandos de barra. Skills externas comuns são chamadas com `/skill:<name>`, sub-Skills externas aparecem como comandos com ponto, como `/parent.child`, e Skills integradas aparecem diretamente como `/<name>` no painel de comandos de barra. Se o nome de uma Skill externa não entrar em conflito com um comando de barra do sistema, você também pode omitir o prefixo `skill:` e digitar `/<name>` diretamente.

Alguns comandos ficam disponíveis somente quando o agente está ocioso. Nesse caso, pressione `Esc` para interromper a saída em streaming ou a compactação do contexto antes de usá-los. Comandos de alternância de modo e consulta, como `/yolo`, `/plan`, `/help` e `/btw`, ficam sempre disponíveis. Para ver a lista completa, consulte a [referência de comandos de barra](../../en/reference/slash-commands.md).

## Referências de arquivo

Digite `@` para abrir o autocompletar de caminhos de arquivo. Ao selecionar um caminho, sua forma relativa é inserida na mensagem, e o agente carrega diretamente o conteúdo do arquivo quando lê essa mensagem. As referências de arquivo funcionam tanto em diretórios git quanto não-git. Sugestões de pastas terminam com `/`, permitindo continuar o autocompletar de caminhos dentro delas. Se o utilitário de busca rápida ainda estiver sendo baixado, o Kimi Code usa uma varredura básica do sistema de arquivos como fallback. Caminhos ocultos ficam disponíveis, mas `.git` é excluído das sugestões.

> Referências com `@` e comandos de barra são mecanismos diferentes: `@` fornece contexto de arquivos ao agente, enquanto `/` aciona recursos integrados ou Skills. Um `/` digitado depois de espaços no início da linha é tratado como texto normal, e não como abertura do menu de comandos de barra.

## Fluxo de aprovação

Quando o agente chama uma ferramenta capaz de causar alterações, como modificar arquivos ou executar comandos, a TUI exibe um painel de aprovação para sua confirmação. Aprovações não são solicitadas para chamadas comuns de ferramentas no modo YOLO nem para gravações em arquivos de plano no modo Plan.

Use as setas para selecionar uma opção e pressione `Enter` para confirmar, ou pressione `1` / `2` / `3` para selecionar diretamente pelo número. `Esc`, `Ctrl-C` e `Ctrl-D` têm o mesmo efeito de rejeitar a solicitação.

O painel normalmente inclui a opção **Approve for this session** ("aprovar nesta sessão"). Ao selecioná-la, chamadas do mesmo tipo são aprovadas automaticamente durante o restante da sessão. Para criar regras permanentes, adicione entradas `allow` / `deny` em [Arquivos de configuração](../../en/configuration/config-files.md#permission).

## Alternar modos

### Modo Plan

No modo Plan, o agente primeiro apresenta um plano de ação e aguarda sua aprovação antes de modificar arquivos. Isso é útil para tarefas complexas ou de maior risco.

- Alternar: `Shift-Tab` ou `/plan`
- Limpar o plano atual: `/plan clear` (somente quando o agente estiver ocioso)

Depois de produzir um plano, o agente pausa para sua revisão. Você pode aprová-lo, rejeitá-lo ou pedir alterações. Sair do modo Plan exige sua confirmação mesmo quando o modo YOLO também está ativo. O modo Auto é a exceção: saídas do plano são aprovadas automaticamente e aparecem como **Auto-approved** ("aprovado automaticamente") no histórico da conversa.

### Modos YOLO e Auto

**Modo YOLO** (`/yolo`) aprova automaticamente chamadas comuns de ferramentas, sendo adequado para tarefas em lote que você sabe que são seguras. Ele ainda solicita confirmação antes de ações sensíveis, como acessar arquivos confidenciais, incluindo `.env` ou chaves SSH, e sair do modo Plan. O agente também pode continuar fazendo perguntas.

**Modo Auto** (`/auto`) é o modo totalmente sem supervisão: todas as aprovações de ferramentas são tratadas automaticamente, incluindo acesso a arquivos sensíveis e saídas do modo Plan, e o agente não faz perguntas. Ele toma as decisões por conta própria.

::: warning Atenção
O modo YOLO ignora a confirmação para gravações em arquivos e execução de comandos. Use-o somente em diretórios de trabalho nos quais você confia.
:::

### Modo shell

O modo shell permite executar comandos do terminal sem sair da conversa. A saída dos comandos é registrada no contexto da conversa, permitindo que o agente veja os resultados nos turnos seguintes.

- Entrar: digite `!` em uma caixa de entrada vazia ou cole um comando que comece com `!`.
- Sair: pressione `Backspace` ou `Esc` com a caixa de entrada vazia. Enviar um comando também retorna automaticamente ao modo normal.
- Executar em segundo plano: enquanto um comando estiver em execução, pressione `Ctrl+B` para movê-lo para uma tarefa em segundo plano.
- Recuperar comandos anteriores: com a caixa de entrada vazia no modo shell, pressione `↑` para navegar por comandos de shell anteriores. Recuperar um deles mantém você no modo shell para que ele possa ser executado novamente como comando.

No modo shell, a caixa de entrada exibe um indicador `!` à esquerda e a borda fica violeta. Por exemplo, você pode executar `!gh auth login` para entrar na GitHub CLI sem abrir outro terminal, permitindo que o Kimi Code CLI use `gh` depois.

## Durante a saída em streaming

A caixa de entrada continua disponível enquanto o agente está pensando ou chamando ferramentas e oferece estas ações adicionais:

- **`Ctrl-S`**: inserir imediatamente o conteúdo da caixa de entrada no turno em execução, sem esperar que ele termine
- **`Esc` / `Ctrl-C`**: interromper o turno atual
- **`Ctrl-O`**: alternar globalmente entre os estados recolhido e expandido da saída das ferramentas e dos resumos de compactação

## Editor externo

Pressione `Ctrl-G` para enviar o conteúdo atual da caixa de entrada a um editor externo. Quando você salvar e fechar, o texto será colocado de volta na caixa de entrada. Se fechar sem salvar, o conteúdo original será preservado. Isso é útil para inserir grandes blocos de texto ou conteúdo com formatação complexa.

A prioridade do editor é: configuração de `/editor` → variável de ambiente `$VISUAL` → variável de ambiente `$EDITOR`. Se nenhuma delas estiver definida, execute `/editor` primeiro para escolher um editor padrão.

## Próximos passos

- [Atalhos de teclado](../../en/reference/keyboard.md): tabela de referência rápida com todos os atalhos
- [Comandos de barra](../../en/reference/slash-commands.md): todos os comandos integrados, com descrições e aliases
- [Sessões e contexto](./sessions.md): como retomar sessões, compactar o contexto e exportar conversas
