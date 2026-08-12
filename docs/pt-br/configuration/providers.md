# Provedores e modelos

O Kimi Code CLI permite conectar várias plataformas de LLM simultaneamente, seja por login com um clique no serviço gerenciado do Kimi Code, conectando o Claude com uma chave de API da Anthropic ou usando serviços de inferência de terceiros por meio do protocolo compatível com OpenAI. Cada provedor corresponde a um protocolo de API específico; os modelos são declarados vinculados aos provedores, com seu próprio nome, tamanho de contexto e recursos. Esta página explica como configurar cada tipo de provedor em `config.toml`.

## Tipos de provedor compatíveis

O campo `type` da tabela `providers` determina qual implementação de protocolo será usada:

| Tipo | Protocolo | Uso típico |
| --- | --- | --- |
| `kimi` | Compatível com OpenAI | Serviço gerenciado do Kimi Code, chave de API da Kimi Platform |
| `anthropic` | Anthropic Messages | Família de modelos Claude |
| `openai` | OpenAI Chat Completions | OpenAI e serviços compatíveis, DeepSeek, Qwen etc. |
| `openai_responses` | OpenAI Responses API | Interface Responses mais recente da OpenAI |
| `google-genai` | Google GenAI | API Gemini |
| `vertexai` | Google GenAI no Vertex | Google Cloud Vertex AI |

Por padrão, todos os provedores se comunicam com os modelos em modo de streaming. Recursos como Thinking, visão e uso de ferramentas são identificados automaticamente pelo prefixo do nome do modelo; normalmente não é necessário declará-los manualmente.

**Prioridade das credenciais**: campo direto `api_key` > chave na subtabela `[providers.<name>.env]` > se ambos estiverem ausentes, a inicialização falha com um erro. A CLI não usa variáveis de ambiente do shell como fallback automático para credenciais. Consulte [Sobrescritas de configuração: credenciais de provedores](./overrides.md#credenciais-de-provedores).

## `/provider` — gerenciamento interativo de provedores

Prefere não editar TOML manualmente? Digite `/provider` na TUI para abrir o **gerenciador de provedores**, onde você pode adicionar ou remover provedores de forma interativa.

O gerenciador exibe os provedores como uma lista de entradas agrupadas por origem. Navegação:

- `↑`/`↓` para mover o cursor e `←`/`→` para mudar de página
- `d` para excluir o provedor atual, com confirmação `[y/N]`
- pressione `Enter` na linha `[ Add New Platform ]` para adicionar um novo provedor

Há dois caminhos ao adicionar um provedor:

- **Provedor de terceiros conhecido**: busca o catálogo de modelos em [models.dev](https://models.dev/). Selecione um provedor → informe uma chave de API → selecione um modelo padrão. Fornecedores cujo protocolo não é declarado pelo catálogo, como xai, openrouter e outros SDKs específicos de fornecedores, são importados como compatíveis com OpenAI e recebem uma observação `guessed`. Quando o catálogo não fornece um endpoint utilizável, primeiro é solicitado um URL base. Protocolos proprietários, como Amazon Bedrock e Cohere, e protocolos explícitos não reconhecidos são recusados. Modelos obsoletos e em estado alpha são excluídos da lista de importação. Se o catálogo público estiver inacessível, a CLI usa como fallback um snapshot integrado do catálogo, permitindo que a importação continue funcionando offline ou em redes bloqueadas.
- **Registro personalizado (`api.json`)**: cole o URL de um registro personalizado e um token Bearer; a CLI cria automaticamente as entradas em `providers` e `models`. Nas inicializações posteriores, provedores provenientes do mesmo URL de registro são atualizados em conjunto, sincronizando adições e remoções de provedores e alterações nos metadados dos modelos feitas na origem.

::: warning Atenção
Contas gerenciadas do Kimi Code autenticadas por OAuth com `/login` não aparecem em `/provider`. Use `/login` e `/logout` para gerenciá-las.
:::

As mesmas operações também estão disponíveis em ambientes não interativos pelo comando de shell [`kimi provider`](../../en/reference/kimi-command.md#kimi-provider).

## `kimi`

Para conectar à interface da Moonshot AI compatível com OpenAI, incluindo o serviço gerenciado do Kimi Code e chaves de API da Kimi Platform.

- `base_url` padrão: `https://api.moonshot.ai/v1`
- nomes das chaves de credencial: `KIMI_API_KEY`, `KIMI_BASE_URL`
- recurso adicional: suporte ao envio de vídeo

```toml
[providers.kimi]
type = "kimi"
base_url = "https://api.moonshot.ai/v1"
api_key = "sk-xxxxx"
```

> Ao usar o serviço gerenciado do Kimi Code, executar `/login` configura automaticamente `base_url` e as credenciais. Nenhuma configuração manual é necessária.

## `anthropic`

Para conectar à API do Claude. Modelos Claude padrão habilitam automaticamente visão, uso de ferramentas e Thinking, quando compatível. Modelos personalizados ou ainda não reconhecidos precisam declarar explicitamente `capabilities` em `[models.<alias>]`.

- `base_url` padrão: segue o padrão do SDK da Anthropic
- nomes das chaves de credencial: `ANTHROPIC_API_KEY`, `ANTHROPIC_BASE_URL`
- `max_tokens` padrão: inferido por modelo. Para sobrescrever, defina `max_output_size` no alias do modelo

```toml
[providers.anthropic]
type = "anthropic"
api_key = "sk-ant-xxxxx"

[models."claude-opus-4-7"]
provider = "anthropic"
model = "claude-opus-4-7"
max_context_size = 200000
# max_output_size = 32000  # opcional; omita para usar o padrão inferido do modelo
```

## `openai`

Para conectar ao protocolo OpenAI Chat Completions e também a qualquer serviço de terceiros compatível com esse protocolo. Sobrescreva `base_url` conforme necessário.

Modelos de raciocínio de terceiros, como DeepSeek, Qwen e One API, funcionam diretamente: a CLI trata automaticamente o campo `reasoning_content` e a injeção de `reasoning_effort`. Se o seu gateway retornar conteúdo de raciocínio em um campo com nome não padrão, defina `reasoning_key` no alias do modelo para sobrescrevê-lo.

- `base_url` padrão: `https://api.openai.com/v1`
- nomes das chaves de credencial: `OPENAI_API_KEY`, `OPENAI_BASE_URL`

```toml
[providers.openai]
type = "openai"
base_url = "https://api.openai.com/v1"
api_key = "sk-xxxxx"
```

## `openai_responses`

Corresponde à Responses API mais recente da OpenAI e funciona sempre em modo de streaming. A configuração é a mesma de `openai`.

- `base_url` padrão: `https://api.openai.com/v1`
- nomes das chaves de credencial: `OPENAI_API_KEY`, `OPENAI_BASE_URL`

```toml
[providers.openai-responses]
type = "openai_responses"
base_url = "https://api.openai.com/v1"
api_key = "sk-xxxxx"
```

## `google-genai`

Para conectar diretamente à API Google Gemini. Thinking, visão e recursos multimodais são identificados automaticamente pelo nome do modelo.

- nome da chave de credencial: `GOOGLE_API_KEY`

```toml
[providers.gemini]
type = "google-genai"
api_key = "xxxxx"
```

Para encaminhar as requisições por um proxy ou gateway compatível com Gemini, defina `base_url` ou a variável de ambiente `GOOGLE_GEMINI_BASE_URL`. Quando omitido, é usado o padrão do SDK: `https://generativelanguage.googleapis.com`.

> Informe **somente a raiz do host**. O SDK Google GenAI acrescenta por conta própria a versão e o caminho da API, por exemplo `/v1beta/models/<model>:generateContent`. Portanto, terminar o URL com `/v1beta` produziria um caminho duplicado como `/v1beta/v1beta/…`.

```toml
[providers.gemini]
type = "google-genai"
api_key = "xxxxx"
base_url = "https://your-gateway.example"
```

## `vertexai`

Compartilha a mesma implementação de `google-genai`; definir `type = "vertexai"` muda para o caminho de acesso do Vertex AI.

A autenticação segue o fluxo padrão de ADC do Google Cloud, usando `gcloud auth application-default login` ou um JSON de conta de serviço definido em `GOOGLE_APPLICATION_CREDENTIALS`. Essa parte é independente do Kimi Code. **O ID do projeto e a região devem ser definidos na subtabela `[providers.vertexai.env]`**. Executar apenas `export GOOGLE_CLOUD_PROJECT` no shell não será lido pela CLI.

```toml
[providers.vertexai]
type = "vertexai"

[providers.vertexai.env]
GOOGLE_CLOUD_PROJECT = "my-gcp-project"
GOOGLE_CLOUD_LOCATION = "us-central1"
```

```sh
gcloud auth application-default login   # autenticação única
kimi
```

Para encaminhar requisições do Vertex por um endpoint personalizado, como um proxy, defina `base_url` ou a variável de ambiente `GOOGLE_VERTEX_BASE_URL`. Quando omitido, é usado o host regional padrão `*-aiplatform.googleapis.com` do SDK. Assim como em `google-genai`, informe somente a raiz do host; o SDK acrescenta `/v1beta1/publishers/google/models/…` por conta própria.

## OAuth e injeção de credenciais

O serviço gerenciado do Kimi Code usa OAuth em vez de chaves de API estáticas. Depois de executar `/login`, o conjunto integrado de ferramentas de autenticação grava e atualiza as credenciais automaticamente. Nenhuma configuração manual em `config.toml` é necessária para isso.

## Próximos passos

- [Arquivos de configuração](./config-files.md): referência completa dos campos das tabelas `providers` e `models`
- [Sobrescritas de configuração](./overrides.md): regras de prioridade para resolução de credenciais dos provedores
- [Variáveis de ambiente](./env-vars.md): nomes das chaves de credencial para cada tipo de provedor
