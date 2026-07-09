# ADR-0006 — Convenções e descoberta do banco de dados

- Estado: Aceito
- Data: 2026-07-02

## Contexto

O domínio terá dezenas de tabelas, regras multi-tenant e histórico sensível. Uma
pessoa nova precisa localizar dados e compreender sua semântica sem depender da
memória de quem criou o sistema.

## Decisão

- PostgreSQL 18 é a versão mínima inicial;
- identificadores internos usam `uuid` com `uuidv7()` gerado pelo banco;
- nomes técnicos são em inglês, `snake_case`, sem identificadores SQL entre aspas;
- tabelas usam plural e colunas usam nomes explícitos;
- tabelas são agrupadas nos schemas `identity`, `tenancy`, `content`,
  `communication` e `audit`;
- todo dado de tenant carrega `orchestra_id` e relações tenant-scoped impedem
  cruzamento de orquestras também por constraints;
- instantes usam `timestamptz`; datas civis sem horário usam `date`;
- PostgreSQL é a fonte de verdade física;
- documentação semântica vive no dicionário Markdown;
- tabelas e colunas recebem `COMMENT ON` nas migrações;
- mudanças de schema e documentação pertencem ao mesmo trabalho.

## Consequências

- a navegação por domínio fica previsível;
- UUIDv7 reduz aleatoriedade de ordenação dos índices sem expor códigos humanos;
- consultas sempre deixam o tenant explícito;
- existe algum trabalho duplicado entre SQL e documentação, controlado por
  revisão e futura validação automatizada;
- PostgreSQL anterior à versão 18 exigiria geração de UUIDv7 na aplicação ou
  extensão compatível e não é o baseline suportado inicialmente.

## Alternativas rejeitadas

- nomes em português no banco: aproximam o negócio, mas fragmentam código e
  ecossistema técnico;
- todas as tabelas em `public`: simples no começo, menos navegável no crescimento;
- chaves sequenciais expostas: acoplam identificação técnica e humana;
- UUIDv4 para tudo: funcional, porém menos amigável à localidade de índices;
- documentação apenas externa: perde contexto durante inspeção direta do banco;
- documentação apenas em `COMMENT ON`: insuficiente para fluxos, diagramas e
  regras entre tabelas.

