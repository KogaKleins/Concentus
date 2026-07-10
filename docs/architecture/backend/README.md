# Arquitetura backend

O backend é um monólito modular NestJS. O processo pode ser implantado junto no
início, mas as fronteiras de código e dados são tratadas como contratos reais.

## Estado

| Área | Estado |
|---|---|
| Módulos e propriedade de tabelas | Definido no ADR-0015 |
| Escrita entre módulos | Proibida diretamente |
| Consultas cruzadas | Permitidas na camada de leitura |
| Transações e efeitos assíncronos | Definido nos ADR-0015 e ADR-0016 |
| Fila transacional | pg-boss definido no ADR-0016 |
| Erros HTTP e idempotência de comandos | Pendente |
| Observabilidade por módulo | Pendente |

## Documentos

- [Módulos e dependências](modules-and-dependencies.md)
- [Jobs assíncronos e workers](async-jobs-and-workers.md)
- [ADR-0015](../decisions/0015-modular-backend-and-data-ownership.md)
- [ADR-0016](../decisions/0016-postgresql-backed-jobs-with-pg-boss.md)

## Estrutura conceitual
```text
apps/api/src/modules/<module>/
├── application/       casos de uso, comandos, queries e portas
├── domain/            regras puras quando existirem
├── infrastructure/    Kysely, storage e adaptadores
├── presentation/      controllers, DTO adapters e serializers
└── module.ts           composição NestJS e exports públicos
```

Essa estrutura é uma direção, não obrigação de criar pastas vazias. Um módulo
começa simples e ganha camadas quando a complexidade real justificar.
