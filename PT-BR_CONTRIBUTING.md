# Como contribuir com a localização pt-BR

Obrigado por considerar colaborar com a documentação brasileira do Kimi Code.

Este é um esforço comunitário em desenvolvimento e ainda não representa uma localização oficial da MoonshotAI.

## Antes de começar

Leia:

1. `PT-BR_LOCALIZATION.md`
2. `PT-BR_GLOSSARY.md`
3. `PT-BR_STATUS.md`
4. a página correspondente em `docs/en/`

## Escolhendo uma tarefa

Prefira páginas marcadas como pendentes em `PT-BR_STATUS.md`.

Antes de iniciar uma tradução maior, abra uma issue ou comente na tarefa correspondente para evitar trabalho duplicado.

Contribuições menores também são bem-vindas, incluindo:

- revisão de linguagem;
- correções de links;
- ajustes terminológicos;
- validação de comandos;
- identificação de páginas desatualizadas;
- melhoria do glossário.

## Regras de tradução

### Preserve literalmente

- comandos e subcomandos;
- flags;
- caminhos de arquivos;
- nomes de arquivos;
- variáveis de ambiente;
- campos e valores de configuração;
- nomes de ferramentas;
- atalhos de teclado;
- URLs e endpoints;
- nomes oficiais do produto.

### Traduza

- títulos e explicações;
- instruções ao usuário;
- descrições de comportamento;
- exemplos de prompts em linguagem natural;
- avisos e dicas, preservando a sintaxe VitePress.

### Não faça

- não invente comportamento ausente na fonte inglesa;
- não remova conteúdo apenas para encurtar a página;
- não traduza identificadores técnicos;
- não apresente a localização como oficial;
- não faça mudanças globais de terminologia sem discussão prévia.

## Fluxo recomendado

1. Sincronize seu fork com a versão mais recente do projeto.
2. Crie uma branch específica para a tarefa.
3. Traduza somente o escopo necessário.
4. Compare lado a lado com `docs/en/`.
5. Verifique o glossário.
6. Execute o build da documentação quando possível.
7. Abra um pull request pequeno e descreva a página fonte usada.

## Título sugerido de PR

```text
docs(pt-br): translate getting started guide
```

## Descrição sugerida

```markdown
## Source

`docs/en/guides/getting-started.md`

## Scope

- add Brazilian Portuguese translation
- preserve commands and technical identifiers
- follow the shared pt-BR glossary

## Validation

- compared against current English source
- checked relative links
- checked VitePress syntax
- docs build: pass / not run
```

## Checklist antes do PR

- [ ] Fonte inglesa conferida
- [ ] Estrutura preservada
- [ ] Nenhum conteúdo técnico perdido
- [ ] Comandos e flags intactos
- [ ] Links conferidos
- [ ] Glossário seguido
- [ ] Português revisado
- [ ] VitePress válido
- [ ] Build executado quando possível

## Revisão

Comentários de revisão devem focar em precisão técnica, naturalidade do português e consistência terminológica.

Quando houver mais de uma tradução aceitável, prefira a forma mais clara para usuários brasileiros e registre decisões reutilizáveis no glossário.
