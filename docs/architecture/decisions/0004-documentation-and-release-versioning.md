# ADR-0004 — Versionamento da documentação e das releases

- Estado: Aceito
- Data: 2026-07-02

## Contexto

Copiar a documentação inteira a cada versão criaria duplicação e divergência. Ao
mesmo tempo, é necessário consultar mudanças e recuperar estados antigos.

## Decisão

- `docs/` contém a documentação viva;
- `CHANGELOG.md` resume mudanças relevantes por versão;
- `docs/releases/` contém comunicação amigável de releases;
- tags Git preservam estados completos e permitem diffs;
- ADRs preservam justificativas estruturais;
- o produto usa `MAJOR.MINOR.PATCH` e pré-releases;
- não existe versão documental independente.

## Consequências

- não há pastas duplicadas `docs/v1.0` e `docs/v1.1`;
- toda mudança funcional atualiza documentação e changelog no mesmo trabalho;
- versões antigas exigem repositório Git e tags imutáveis.

