# Registros de decisões arquiteturais

ADRs registram contexto, decisão, consequências e alternativas de escolhas
estruturais. Eles explicam **por que** a arquitetura existe; a documentação viva
explica **como** o produto funciona atualmente.

## Índice

| ADR | Estado | Decisão |
|---|---|---|
| [0001](0001-platform-and-tenant-branding.md) | Substituído por ADR-0007 | Personalização visual por tenant |
| [0002](0002-global-account-and-tenant-profiles.md) | Aceito | Conta global com perfis isolados por orquestra |
| [0003](0003-hierarchical-authorization.md) | Aceito | Autorização hierárquica com pesos administrativos |
| [0004](0004-documentation-and-release-versioning.md) | Aceito | Documentação viva, changelog, ADRs e tags |
| [0005](0005-application-stack-and-sql-first-data-access.md) | Aceito | Stack TypeScript e persistência PostgreSQL SQL-first |
| [0006](0006-database-conventions-and-discoverability.md) | Aceito | Convenções e documentação rastreável do banco |
| [0007](0007-single-product-visual-identity.md) | Aceito | Identidade visual única com contexto institucional controlado |
| [0008](0008-frontend-foundation-routing-and-themes.md) | Aceito | Rotas por slug, design system, showcase e preferência de tema |
| [0009](0009-frontend-data-realtime-and-upload-state.md) | Aceito | Dados frontend, SSE, estado na URL e uploads persistentes à navegação |
| [0010](0010-drafts-sharing-and-concurrent-editing.md) | Aceito | Privacidade de rascunhos, revisão otimista e retenção |
| [0011](0011-shared-validation-and-autosave-timing.md) | Aceito | Zod compartilhado, React Hook Form e debounce do autosave |
| [0012](0012-pwa-cache-installation-and-updates.md) | Aceito | Cache estático, instalação opcional e atualização segura da PWA |
| [0013](0013-accessibility-and-browser-support.md) | Aceito | WCAG 2.2 AA e matriz móvel de navegadores suportados |
| [0014](0014-testing-strategy-and-quality-gates.md) | Aceito | Vitest, PostgreSQL real, Storybook e Playwright por gates |
| [0015](0015-modular-backend-and-data-ownership.md) | Aceito | Módulos NestJS, ownership de tabelas e efeitos após commit |
| [0016](0016-postgresql-backed-jobs-with-pg-boss.md) | Aceito | pg-boss, jobs transacionais e worker separado |
| [0017](0017-security-baseline-and-threat-model.md) | Aceito | Baseline de segurança, threat model e controles P0 iniciais |
| [0018](0018-authentication-sessions-cookies-and-csrf.md) | Aceito | Autenticação, sessão server-side, cookies seguros e CSRF |
| [0019](0019-rls-and-secure-tenant-context.md) | Aceito | RLS e contexto seguro de tenant |
| [0020](0020-rate-limit-and-abuse-controls.md) | Aceito | Rate limit e controle de abuso |
| [0021](0021-secure-upload-antimalware-and-file-serving.md) | Aceito | Upload seguro, antimalware e entrega de arquivos |
| [0022](0022-secrets-backup-restore-and-incident-response.md) | Aceito | Segredos, backup, restore e resposta a incidentes |
| [0023](0023-security-tests-and-acceptance-gates.md) | Aceito | Testes de segurança e critérios de aceite |

## Numeração e alteração

- novos ADRs recebem o próximo número de quatro dígitos;
- ADR aceito não é reescrito para esconder uma mudança de decisão;
- nova decisão cria novo ADR e marca o anterior como substituído;
- correções editoriais que não mudam significado são permitidas.
