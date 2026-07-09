# Changelog

Todas as mudanças relevantes do Concentus serão documentadas neste arquivo.

O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/) e o
produto adota a política definida em
[Versionamento](docs/reference/versioning.md).

## [Unreleased]

### Added

- Especificação inicial de produto, arquitetura, segurança e experiência mobile.
- Política formal de versionamento e notas de release.
- Registros de decisões arquiteturais.
- Stack principal e política SQL-first com Kysely registradas no ADR-0005.
- Convenções de banco, mapa lógico e padrão do dicionário de dados.
- Fundação frontend com Storybook, Tailwind, shadcn/ui e preferência clara/escura.
- Estratégia frontend de dados, SSE para atualizações quase instantâneas, filtros
  na URL e painel global de uploads.
- Autosave para obras, comunicados e uploads em lote; notificações de comentários
  agrupadas para o autor do comunicado.
- Rascunhos privados com compartilhamento explícito, controle de revisão contra
  sobrescrita e aviso configurável antes da limpeza de arquivos técnicos.
- Validação compartilhada com Zod 4 e React Hook Form, mais autosave após 1,5
  segundo de inatividade ou imediatamente na troca de etapa; erros aparecem após
  sair do campo ou tentar avançar/publicar.
- Política da PWA para cache somente de assets estáticos, download explícito fora
  do cache, instalação opcional e atualizações sem recarga automática.
- Download com padrão herdado da biblioteca, exceção por material e apenas log
  técnico temporário, sem relatório comportamental para maestros.
- WCAG 2.2 nível AA, matriz móvel de navegadores e retenção padrão de 90 dias para
  logs técnicos de download.
- Estratégia de testes com Vitest, Storybook, Playwright, PostgreSQL real e gates
  progressivos para pull requests e releases.
- Catálogo E2E crítico inicial para identidade, isolamento multi-tenant,
  publicação, hierarquia e comunicados.
- Testcontainers como PostgreSQL de integração padrão e pisos de cobertura com
  ratchet, incluindo 90% de branches nos módulos críticos de segurança.
- Backend dividido em nove módulos NestJS, ownership explícito de tabelas,
  consultas cruzadas somente para leitura e efeitos não críticos após commit.

### Changed

- A plataforma passa a se chamar Concentus.
- Nomes de orquestras passam a existir exclusivamente como dados de tenants.
- Documentação reorganizada por finalidade, sem cópias completas por versão.
- Identidade visual passa a ser única do Concentus; tenants não redefinem tema,
  tipografia, componentes ou navegação.
- Preferência de tema explícita é sincronizada pela conta; o modo `system`
  acompanha cada dispositivo sem solicitar permissão.
