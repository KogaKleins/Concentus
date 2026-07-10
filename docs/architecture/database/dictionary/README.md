# Dicionário de dados

## 1. Finalidade

O dicionário explica o significado e o uso de cada tabela e coluna. Ele não é uma
listagem automática sem contexto: registra semântica, privacidade, ciclo de vida,
integridade, RLS, índices e auditoria.

## 2. Índice por domínio

| Domínio | Documento | Estado |
|---|---|---|
| Identidade | [identity.md](identity.md) | Proposto para P2 |
| Tenancy | [tenancy.md](tenancy.md) | Proposto para P2 |
| Autorização e concessões | [authorization-and-access.md](authorization-and-access.md) | Proposto para P2 |
| Conteúdo | [content.md](content.md) | Proposto para P2 |
| Comunicação | [communication.md](communication.md) | Proposto para P2 |
| Auditoria | `audit.md` | A modelar |

O documento de autorização é transversal: suas tabelas vivem no schema `content`
porque protegem recursos de conteúdo, mas a regra pertence ao módulo de
Autorização. Os demais arquivos serão criados quando o domínio for modelado. O
formato obrigatório está em [Modelo de ficha](table-template.md).

## 3. Como localizar um dado

1. consultar o [Glossário](../../../reference/glossary.md) para o termo de negócio;
2. localizar o domínio no índice acima;
3. localizar a tabela pelo propósito, não apenas pelo nome;
4. consultar relações, RLS e exemplos da ficha;
5. localizar a migração de origem indicada pela ficha;
6. confirmar o estado físico pelos comentários e catálogo do PostgreSQL.

## 4. Regra de atualização

Uma mudança de tabela só está completa quando atualiza:

- migração;
- ficha do dicionário;
- `COMMENT ON`;
- tipos Kysely;
- diagrama afetado;
- testes;
- changelog, quando relevante.
