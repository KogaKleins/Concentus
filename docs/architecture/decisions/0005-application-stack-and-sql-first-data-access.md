# ADR-0005 — Stack da aplicação e acesso SQL-first

- Estado: Aceito
- Data: 2026-07-02

## Contexto

O Concentus precisa iniciar pequeno, ser mantido inicialmente por uma pessoa e
preservar espaço para novas áreas, como blogs e chats. O domínio possui regras
relacionais, autorização hierárquica, RLS, constraints e consultas que não devem
ser escondidas por um ORM tradicional.

## Decisão

- TypeScript é a linguagem principal da aplicação;
- Next.js com React atende frontend e PWA;
- NestJS implementa API e workers num monólito modular;
- PostgreSQL é a fonte de verdade para esquema, integridade e autorização de
  segunda camada;
- Kysely fornece composição SQL tipada, inferência de resultados e transações;
- não serão usados Active Record, sincronização automática ou ORM tradicional;
- migrações são explícitas, sequenciais, revisadas e versionadas;
- SQL nativo é permitido e esperado para RLS, recursos específicos do PostgreSQL
  e consultas em que seja mais claro;
- frontend, API e worker vivem num monorepo pnpm com contratos compartilhados;
- a implantação inicial será conteinerizada e compatível com Docker Compose.

## Limites da decisão

Este ADR não escolhe:

- provedor ou implementação de armazenamento S3 compatível;
- provedor de e-mail;
- biblioteca final de autenticação;
- estratégia detalhada de sessão e RLS;
- runner exato das migrações;
- provedor de hospedagem.

Esses itens exigem decisões próprias.

## Consequências positivas

- uma linguagem atravessa frontend e backend;
- consultas comuns possuem tipos e autocomplete;
- PostgreSQL permanece visível e plenamente utilizável;
- módulos podem ser separados no futuro sem começar com microserviços;
- PWA e futuras interfaces públicas podem compartilhar a base React.

## Consequências e custos

- a equipe precisa conhecer SQL e PostgreSQL de verdade;
- tipos do banco precisam permanecer sincronizados com migrações;
- consultas e políticas avançadas exigem testes de integração;
- NestJS não possui integração oficial específica com Kysely, então haverá um
  `DatabaseModule` próprio e pequeno.

## Alternativas rejeitadas

### Prisma

Boa experiência de CRUD, mas cria uma camada de schema adicional e exige SQL
customizado para parte dos recursos avançados.

### TypeORM

Integração direta com NestJS, porém adota entidades e decorators que não agregam
valor proporcional neste domínio.

### Drizzle ORM

É próximo de SQL e seria viável, mas Kysely mantém mais claramente o banco como
fonte de verdade e limita sua responsabilidade à construção tipada de consultas.

### node-postgres sem query builder

Oferece controle máximo, mas perde inferência, autocomplete e verificação de
colunas em grande parte do código de acesso a dados.

