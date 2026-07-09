# Convenções do banco de dados

## 1. Princípios

1. Clareza vence abreviação.
2. Integridade relevante é garantida pelo PostgreSQL.
3. O tenant aparece explicitamente nos dados e nas relações.
4. Configurabilidade não justifica JSON sem estrutura.
5. Migrações são imutáveis depois de compartilhadas.
6. Uma tabela deve representar um conceito reconhecível.
7. Índices nascem de consultas e constraints reais.

## 2. Versão e extensões

- baseline: PostgreSQL 18 ou superior dentro da major suportada;
- toda extensão precisa de ADR ou justificativa na migração;
- extensões não são instaladas manualmente fora das migrações;
- dependência de extensão deve aparecer no inventário operacional.

## 3. Idioma e forma dos nomes

| Elemento | Regra | Exemplo |
|---|---|---|
| Schema | inglês, singular, `snake_case` | `communication` |
| Tabela | inglês, plural, `snake_case` | `orchestra_profiles` |
| Coluna | inglês, `snake_case` | `display_name` |
| Chave primária | `id` | `id` |
| Chave estrangeira | entidade singular + `_id` | `orchestra_id` |
| Booleano | pergunta positiva | `is_active` |
| Instante | sufixo `_at` | `published_at` |
| Data civil | sufixo `_date` ou nome inequívoco | `birth_date` |
| Contagem | sufixo `_count` | `attempt_count` |
| Código humano | `_code` ou nome de domínio | `catalog_number` |

Proibido:

- nomes como `data`, `info`, `value`, `type1` ou `aux` sem contexto;
- abreviações não registradas no glossário;
- identificadores SQL com maiúsculas que exijam aspas;
- nomes de tabela que repitam o schema sem agregar clareza;
- palavras reservadas quando houver alternativa clara.

## 4. Schemas

| Schema | Responsabilidade |
|---|---|
| `identity` | Conta global, credenciais, sessões, MFA e recuperação |
| `tenancy` | Orquestras, perfis, convites, espaços, membros e vozes |
| `content` | Bibliotecas, obras, materiais, arquivos e concessões |
| `communication` | Comunicados, interações, enquetes e notificações |
| `audit` | Eventos imutáveis e sessões técnicas de impersonação |

`public` não receberá tabelas de negócio. Seu uso fica limitado ao necessário para
extensões e infraestrutura técnica controlada.

## 5. Chaves e identificadores

- chave primária padrão: `id uuid PRIMARY KEY DEFAULT uuidv7()`;
- UUID é interno e não substitui número de catálogo, slug ou código visível;
- códigos humanos são colunas separadas e recebem regra de unicidade no escopo
  correto;
- UUIDv7 não substitui `created_at`;
- IDs nunca são reciclados;
- tabelas associativas podem usar chave composta quando a própria relação for sua
  identidade e não precisar ser referenciada externamente;
- tabelas associativas com ciclo de vida, autoria ou auditoria recebem `id` próprio.

O PostgreSQL 18 fornece `uuidv7()` nativo e temporalmente ordenável. A aplicação
não deve gerar um ID diferente sem motivo documentado.

## 6. Multi-tenancy

- toda tabela pertencente a uma orquestra contém `orchestra_id NOT NULL`;
- `orchestra_id` aparece próximo de `id` na definição;
- relações entre dados tenant-scoped incluem o tenant na integridade referencial;
- cada tabela tenant-scoped possui `UNIQUE (orchestra_id, id)` quando necessário
  para suportar FK composta;
- FK composta usa `(orchestra_id, target_id)` e referencia
  `(orchestra_id, id)`;
- toda consulta de aplicação opera dentro de contexto de tenant;
- RLS é segunda barreira, não substituto das cláusulas e relações corretas;
- dados globais, como `identity.accounts`, não recebem `orchestra_id`.

Exemplo conceitual:

```sql
FOREIGN KEY (orchestra_id, library_id)
  REFERENCES content.libraries (orchestra_id, id)
  ON DELETE RESTRICT
```

## 7. Tipos

| Necessidade | Tipo preferido |
|---|---|
| Identificador | `uuid` |
| Texto geral | `text` |
| Texto com limite de negócio real | `varchar(n)` + justificativa |
| Instante | `timestamptz` |
| Data sem horário | `date` |
| Duração | `interval` ou inteiro com unidade explícita |
| Contador | `integer` ou `bigint` conforme escala |
| Tamanho de arquivo | `bigint` em bytes |
| Dinheiro futuro | `numeric`, nunca ponto flutuante |
| Flag binária genuína | `boolean NOT NULL` |
| Metadado flexível excepcional | `jsonb` documentado |

### Texto

Usar `text` por padrão. Limite de interface não precisa virar `varchar` sem regra
de negócio. Normalizações pesquisáveis devem possuir estratégia explícita, como
coluna normalizada ou índice funcional.

### JSONB

`jsonb` não será usado para:

- relações entre entidades;
- campos que precisam de FK;
- valores pesquisados ou ordenados rotineiramente;
- esconder variações de schema;
- substituir tabelas configuráveis.

Cada uso deve listar estrutura esperada, motivo e estratégia de evolução.

## 8. Datas, horas e fuso

- instantes usam `timestamptz` e são tratados em UTC;
- exibição converte para o fuso IANA da orquestra;
- fuso é armazenado como nome, por exemplo `America/Sao_Paulo`, nunca somente
  deslocamento fixo;
- datas civis como nascimento usam `date`;
- não usar `timestamp without time zone` para eventos reais;
- nomes padrão: `created_at`, `updated_at`, `published_at`, `expires_at`,
  `revoked_at` e `deleted_at`;
- `created_at` não é inferido do UUID;
- precisão padrão de aplicação: microssegundos do PostgreSQL, serialização ISO 8601.

## 9. Estados e valores configuráveis

- ciclo de vida usa uma coluna `status` explícita;
- não criar combinações como `is_active`, `is_deleted`, `is_published` e
  `is_draft` para representar um único estado;
- estados técnicos fechados usam `text` com `CHECK` e união TypeScript;
- valores configuráveis pela orquestra usam tabelas próprias;
- PostgreSQL ENUM só será usado após justificativa específica;
- transições válidas pertencem à regra de negócio e recebem testes.

## 10. Nullabilidade e defaults

- `NOT NULL` é padrão quando ausência não possui significado de negócio;
- `NULL` significa desconhecido, inexistente ou não aplicável, nunca string vazia;
- defaults devem ser determinísticos e não esconder decisão do usuário;
- booleanos recebem default somente quando o significado for inequívoco;
- timestamps de criação podem usar `DEFAULT now()`;
- `updated_at` deve ser atualizado de forma uniforme pela camada de persistência ou
  mecanismo central documentado, nunca manualmente de maneira dispersa.

## 11. Constraints

Prefixos:

| Tipo | Prefixo | Exemplo |
|---|---|---|
| Primary key | `pk_` | `pk_works` |
| Foreign key | `fk_` | `fk_works_library` |
| Unique | `uq_` | `uq_works_library_catalog_number` |
| Check | `ck_` | `ck_materials_status` |
| Exclusion | `ex_` | `ex_scheduled_ranges` |

Regras:

- toda FK possui comportamento de exclusão explícito;
- `ON DELETE RESTRICT` ou `NO ACTION` é o padrão;
- `CASCADE` somente para dependente inseparável e documentado;
- `SET NULL` somente quando a relação histórica opcional continuar inteligível;
- unicidade considera o escopo correto do tenant;
- regras de formato simples e invariantes locais usam `CHECK`;
- regra entre linhas/tabelas usa FK, UNIQUE, EXCLUDE ou transação apropriada, não
  `CHECK` enganoso.

## 12. Índices

Prefixo `idx_`, seguido da tabela e finalidade:

```text
idx_works_orchestra_library
idx_notifications_profile_unread
idx_audit_events_orchestra_occurred_at
```

- PostgreSQL não cria automaticamente índice útil para toda FK; avaliar e criar;
- índices compostos seguem a ordem dos filtros e ordenação reais;
- índices parciais são preferidos para filas pequenas, como notificações não lidas;
- índice duplicado por PK/UNIQUE é proibido;
- todo índice não óbvio registra consulta que sustenta;
- índices são revistos com `EXPLAIN (ANALYZE, BUFFERS)` usando volume representativo.

## 13. Exclusão e histórico

- desativação e remoção de acesso não equivalem a exclusão física;
- `deleted_at` só existe onde recuperação/retensão for requisito;
- arquivos aprovados para exclusão são removidos do object storage;
- autoria histórica referencia perfil desativado, não usuário apagado;
- logs de auditoria são append-only para a aplicação comum;
- cascatas nunca podem apagar auditoria relevante.

## 14. Documentação dentro do banco

Toda migração que cria tabela inclui:

```sql
COMMENT ON TABLE content.works IS
  'Obra catalogada dentro de uma biblioteca de uma orquestra.';

COMMENT ON COLUMN content.works.catalog_number IS
  'Identificador humano único dentro da biblioteca; não é chave técnica.';
```

Comentários explicam significado, unidade, escopo e sensibilidade; não repetem
apenas o nome da coluna.

## 15. Consultas e transações

- sempre parametrizar valores;
- selecionar colunas necessárias em vez de `SELECT *` no código de produção;
- transação pertence ao caso de uso, não a cada repository isoladamente;
- publicação em lote, convite de uso único e troca de voto exigem atomicidade;
- operações repetíveis por worker devem ser idempotentes;
- queries complexas recebem nome, teste e comentário sobre intenção;
- SQL nativo é preferível a uma composição menos legível.

