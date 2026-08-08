# Primeiros passos

## O que é o Kimi Code CLI

O Kimi Code CLI é um agente de IA que funciona no terminal e ajuda você a realizar tarefas de desenvolvimento de software e operações do dia a dia no terminal. Ele pode ler e modificar código, executar comandos de shell (o interpretador de comandos do terminal), pesquisar arquivos, buscar páginas da web e planejar e ajustar autonomamente os próximos passos com base no feedback recebido durante o trabalho.

Ele é útil em cenários como:

- **Escrever e modificar código**: implementar novos recursos, corrigir bugs e concluir refatorações
- **Entender um projeto**: explorar uma base de código desconhecida e responder a perguntas sobre arquitetura e implementação
- **Automatizar tarefas**: processar arquivos em lote, executar builds e testes e encadear vários scripts

A CLI (interface de linha de comando) é escrita em TypeScript e está disponível tanto como binário independente quanto como pacote npm.

## Instalação

Há duas opções de instalação: o script oficial de instalação, recomendado e sem necessidade de Node.js pré-instalado, e a instalação global via npm.

::: tip Antes de instalar
O Kimi Code CLI é um aplicativo TUI totalmente interativo, isto é, uma interface de usuário no terminal. Para uma melhor experiência visual, execute-o em um terminal com suporte a true color, ou cores de 24 bits, e ligaduras tipográficas, como [Kitty](https://sw.kovidgoyal.net/kitty/) ou [Ghostty](https://ghostty.org/).
:::

### Script de instalação (recomendado)

- **macOS / Linux**:

```sh
curl -fsSL https://code.kimi.com/kimi-code/install.sh | bash
```

- **Windows (PowerShell)**:

```powershell
irm https://code.kimi.com/kimi-code/install.ps1 | iex
```

> No Windows, instale o [Git for Windows](https://gitforwindows.org/) antes da primeira execução. O Kimi Code CLI usa o Git Bash incluído como ambiente de shell. Se o Git Bash estiver instalado em um local personalizado, defina `KIMI_SHELL_PATH` com o caminho absoluto de `bash.exe`.

O script baixa automaticamente a versão mais recente, verifica o checksum, ou soma de verificação, e coloca o executável `kimi` no seu `PATH`.

### Instalação via npm

Requer Node.js 22.19.0 ou posterior:

```sh
node --version
npm install -g @moonshot-ai/kimi-code
```

Ou com pnpm:

```sh
pnpm add -g @moonshot-ai/kimi-code
```

## Atualizar e desinstalar

Depois da instalação, verifique se o executável está pronto:

```sh
kimi --version
```

**Atualizar**: execute `kimi upgrade`. A CLI verifica a versão mais recente e apresenta as opções de atualização. Escolha `Install update now` ("instalar atualização agora") para atualizar de acordo com a origem da instalação atual. Você também pode atualizar diretamente pelo gerenciador de pacotes:

```sh
npm install -g @moonshot-ai/kimi-code@latest
```

**Desinstalar**: se você instalou pelo script, exclua o executável `kimi`. Se instalou via npm:

```sh
npm uninstall -g @moonshot-ai/kimi-code
```

## Primeira execução

Entre no diretório do seu projeto e execute `kimi` para iniciar a interface interativa:

```sh
cd your-project
kimi
```

Para executar uma única instrução sem entrar na interface interativa, use `-p`:

```sh
kimi -p "Dê uma olhada na estrutura de diretórios deste projeto"
```

Para retomar a sessão anterior, adicione `-c`:

```sh
kimi -c
```

Na primeira execução, você precisa configurar como o Kimi Code CLI acessará a API. Na interface interativa, digite `/login` para iniciar o fluxo de login:

```
/login
```

`/login` abre um seletor de plataforma com duas opções:

- **Kimi Code (OAuth)**: fluxo por código de dispositivo. Abra o link em qualquer dispositivo, faça login e informe o código para autorizar.
- **Kimi Platform API key**: informe uma chave de API de `platform.kimi.com` ou `platform.kimi.ai`.

Para sair da conta, digite `/logout`, que limpa as credenciais atuais.

::: tip Outros provedores de IA
Para conectar Anthropic, OpenAI, Google ou outros provedores, edite diretamente `~/.kimi-code/config.toml` e configure a chave de API. Consulte [Provedores e modelos](../configuration/providers.md) para mais detalhes. Para a referência completa das opções de configuração, consulte [Arquivos de configuração](../configuration/config-files.md), [Variáveis de ambiente](../configuration/env-vars.md) e [Sobrescritas de configuração](../../en/configuration/overrides.md).
:::

## Sua primeira conversa

Depois de fazer login, descreva uma tarefa em linguagem natural. Um bom ponto de partida é deixar o Kimi Code CLI se familiarizar com o projeto:

```
Analise a estrutura de diretórios deste projeto e descreva brevemente para que serve cada diretório.
```

O Kimi Code CLI chama automaticamente ferramentas de leitura de arquivos, busca e outras ferramentas para explorar o conteúdo relevante antes de responder. Por padrão, operações somente leitura são executadas automaticamente, sem exigir confirmação. Para operações que modificam arquivos ou executam comandos de shell, ele solicita sua confirmação antes de continuar.

Você também pode descrever diretamente uma tarefa mais concreta:

```
Adicione uma função em src/utils que converta qualquer string para kebab-case e crie um teste unitário para ela.
```

O Kimi Code CLI planeja as etapas, modifica o código, executa os testes e informa o que fez em cada etapa.

::: tip Dica
Se não souber o que fazer, digite `/help` a qualquer momento para abrir o painel integrado de comandos e atalhos de teclado. Use `↑`/`↓` para navegar e `Esc` para fechar. Para sair, digite `/exit`, pressione `Ctrl-C` duas vezes ou pressione `Ctrl-D` com a caixa de entrada vazia.
:::

## Comandos comuns e atalhos de teclado

Para quem está usando o Kimi Code CLI pela primeira vez, isto é o essencial:

**Comandos de sessão**

| Comando | Descrição |
| --- | --- |
| `/new` | Iniciar uma nova sessão e limpar o contexto atual |
| `/sessions` | Navegar pelo histórico de sessões e escolher uma para retomar |
| `/model` | Alternar o modelo atual |
| `/compact` | Compactar manualmente o contexto para liberar espaço para tokens |
| `/fork` | Criar um fork da sessão atual como uma cópia independente com todo o histórico, permanecendo na sessão atual |

**Atalhos de teclado mais usados**

| Atalho | Descrição |
| --- | --- |
| `Esc` | Interromper a saída em streaming / fechar um pop-up |
| `Ctrl-C` | Interromper a saída; pressione duas vezes quando o agente estiver ocioso para sair |
| `Shift-Tab` | Alternar o modo Plan |
| `Ctrl-S` | Inserir uma mensagem durante a saída em streaming sem esperar a resposta atual terminar |
| `Ctrl-O` | Recolher / expandir a saída das ferramentas e os resumos de compactação |

Para ver a lista completa, digite `/help` ou consulte a [Referência de comandos de barra](../../en/reference/slash-commands.md) e os [Atalhos de teclado](../../en/reference/keyboard.md).

## Onde os dados são armazenados

Por padrão, o Kimi Code CLI armazena os dados locais em `~/.kimi-code/`, incluindo arquivos de configuração, registros de sessões, logs e o cache de atualização. Para usar outro local, aponte a variável de ambiente `KIMI_CODE_HOME` para um novo caminho. Para ver a estrutura completa de diretórios, consulte [Locais de armazenamento](../../en/configuration/data-locations.md) e [Variáveis de ambiente](../configuration/env-vars.md).

## Próximos passos

- [Interação e entrada](./interaction.md): operações da caixa de entrada, fluxo de aprovação, modo Plan e modo YOLO
- [Sessões e contexto](./sessions.md): retomar sessões, compactar o contexto e exportar sessões
- [Casos de uso comuns](../../en/guides/use-cases.md): exemplos de prompts para tarefas frequentes
