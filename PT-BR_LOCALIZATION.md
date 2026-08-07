# Kimi Code PT-BR

Community-maintained Brazilian Portuguese localization proposal for the Kimi Code documentation.

> This work is not an official MoonshotAI localization yet. The goal is to build a high-quality, reviewable `pt-BR` documentation set that can serve Brazilian users today and be ready for upstream adoption if the maintainers approve it.

## Project goals

- Provide clear and technically accurate Brazilian Portuguese documentation.
- Keep `docs/pt-br/` aligned with the canonical English documentation under `docs/en/`.
- Preserve commands, flags, paths, identifiers, product names, and literal interface behavior.
- Maintain a shared terminology glossary so translations remain consistent.
- Make progress visible and easy for new contributors to understand.
- Keep changes small and reviewable.

## Current status

| Area | Status |
| --- | --- |
| Technical impact assessment | Complete |
| Initial terminology glossary | Complete |
| Pilot translation | Complete |
| Maintenance plan | Complete |
| Upstream localization proposal | Open |
| Full `pt-BR` documentation tree | In progress |

Upstream proposal: MoonshotAI/kimi-code#2685

Initial documentation contribution: MoonshotAI/kimi-code#2664

## Source of truth

`docs/en/` is the canonical source for this localization.

```text
docs/en/     -> canonical documentation
docs/pt-br/  -> Brazilian Portuguese localization
docs/zh/     -> additional reference, not the translation source
```

When the English and Chinese versions differ, contributors should verify the current product behavior before translating. Do not silently invent a third interpretation in Portuguese.

## Translation principles

1. Translate meaning, not syntax.
2. Keep commands, paths, configuration fields, environment variables, shortcuts, and code unchanged.
3. Preserve official names such as Kimi Code CLI, Kimi Code for VS Code, Agent Skills, MCP, OAuth, JSON, and JSONL.
4. Translate natural-language prompt examples.
5. Explain technical jargon briefly on first use when needed.
6. Keep literal UI labels in their original language when the product itself is not localized, adding a Portuguese explanation when useful.
7. Do not add product behavior that is not documented by the canonical source.
8. While a referenced pt-BR page is still pending, link temporarily to the canonical English page instead of leaving a dead link. Replace the fallback with the local pt-BR path when that page is published.

## Proposed locale

```text
locale: pt-BR
path:   docs/pt-br/
route:  /pt-br/
label:  Português (Brasil)
```

## Suggested contribution flow

1. Pick a page listed as pending in `PT-BR_STATUS.md`.
2. Compare it with the latest `docs/en/` source.
3. Check terminology in `PT-BR_GLOSSARY.md`.
4. Translate the page while preserving structure and technical literals.
5. Verify links, code blocks, callouts, and headings.
6. Run the VitePress documentation build when possible.
7. Submit a focused pull request to this fork for review.

## Review checklist

- [ ] Same source-page scope and section order
- [ ] No technical content removed without explanation
- [ ] Commands and flags preserved
- [ ] Paths and identifiers preserved
- [ ] Official product names preserved
- [ ] Terminology consistent with the glossary
- [ ] Relative links preserved or intentionally adjusted
- [ ] Pending pt-BR targets use canonical English fallbacks instead of dead links
- [ ] VitePress containers correctly closed
- [ ] Code fences keep their language tags
- [ ] Portuguese reads naturally, not as literal machine translation

## Maintenance model

Product behavior, security, authentication, installation, permissions, and configuration changes take priority over editorial polish.

When `docs/en/` changes, the corresponding `pt-BR` page should ideally be updated in the same development cycle. If that is not practical, the difference should be tracked openly until synchronized.

Temporary English fallback links are part of the incremental rollout only. Each fallback should be replaced with the corresponding local pt-BR link as soon as the translated target page is published.

The release changelog remains a separate policy decision pending upstream guidance.

## How to help

Contributors can help by:

- translating a pending page;
- reviewing Brazilian Portuguese wording;
- checking commands and technical accuracy;
- reporting outdated translations;
- proposing glossary improvements;
- validating documentation builds and links.

Small contributions are welcome. A single carefully reviewed paragraph is more useful than a large unverified translation dump.

## License

This repository is licensed under the MIT License. The localization follows the same repository license and attribution requirements.
