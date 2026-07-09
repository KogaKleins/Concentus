# ADR-0001 — Marca da plataforma e identidade do tenant

- Estado: Substituído por ADR-0007
- Data: 2026-07-02

## Contexto

A plataforma pode atender várias orquestras independentes. Nenhum nome de
orquestra deve estar acoplado ao software.

## Decisão

- a plataforma chama-se **Concentus**;
- cada orquestra é um tenant com nome, logo, cores e slug armazenados no banco;
- dentro do tenant, a interface mostra somente a identidade da orquestra ativa;
- Concentus aparece em superfícies globais, como autenticação e painel master;
- documentação, código e banco não presumem a existência de uma orquestra
  específica.

## Consequências

- o mesmo produto pode atender novas orquestras sem rebranding de código;
- componentes de interface precisam resolver marca pelo tenant atual;
- links, e-mails de conteúdo e arquivos usam a identidade da orquestra;
- superfícies globais permanecem neutras e identificadas como Concentus.
