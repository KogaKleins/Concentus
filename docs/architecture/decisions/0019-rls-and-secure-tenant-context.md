# ADR-0019 — RLS e contexto seguro de tenant

- Estado: Aceito
- Data: 2026-07-09

## Contexto

O Concentus é uma plataforma multi-orquestra. A aplicação deve impedir que dados
de uma orquestra vazem para outra mesmo quando um endpoint recebe um UUID válido
de outro tenant, quando uma consulta administrativa é escrita incorretamente ou
quando um worker processa um job assíncrono.

A autorização da API continua sendo a primeira barreira, pois ela conhece regras
de produto como liderança, peso administrativo, bloqueio do maestro, concessões e
estado de publicação. Mesmo assim, isolamento entre tenants é risco estrutural e
merece uma segunda barreira no banco.

O PostgreSQL oferece Row-Level Security (RLS), políticas por comando e o padrão
default-deny quando RLS está habilitado sem política aplicável. Também permite
configuração transacional por `SET LOCAL` ou `set_config(..., true)`, útil para
propagar `orchestra_id` sem depender de estado permanente da conexão.

## Decisão

### Modelo multi-tenant

- A V1 usará banco compartilhado e tabelas compartilhadas.
- Não criaremos um schema ou banco separado por orquestra na V1.
- Tabelas de domínio pertencentes a uma orquestra terão `orchestra_id uuid not null`.
- Tabelas globais, como conta global, credenciais e sessões, não terão
  `orchestra_id` obrigatório, mas terão políticas próprias de privacidade e acesso.
- Tabelas de vínculo, como perfil/membro por orquestra, carregarão `orchestra_id`
  quando isso for necessário para isolamento, FK composta e consultas claras.
- FKs compostas devem impedir relacionamento cruzado entre linhas de orquestras
  diferentes sempre que o relacionamento for tenant-scoped.

### Fonte do tenant

- O cliente nunca é fonte de autoridade para escolher o tenant.
- O slug da URL identifica uma orquestra candidata.
- A sessão autenticada identifica a conta global.
- A API valida que a conta possui perfil ativo naquela orquestra antes de abrir
  contexto autenticado de tenant.
- `orchestra_id` enviado em body, query ou header não é confiável; quando existir
  por conveniência, deve ser ignorado, derivado no servidor ou validado contra o
  contexto resolvido.
- A autorização de produto permanece no backend; RLS não substitui regras de
  permissão contextual.

### Papéis de banco

- A aplicação usará papel de banco sem `BYPASSRLS` e sem privilégios de owner das
  tabelas de negócio.
- Migrações usarão papel separado de owner/migration.
- Worker usará papel de banco sem `BYPASSRLS`; pode ser o mesmo papel da API ou
  um papel próprio com grants mínimos.
- Nenhum processo normal da aplicação conecta como superuser.
- Operações técnicas globais exigem caminho separado, papel ou função explícita,
  auditoria técnica e justificativa.

### Propagação de contexto

Toda operação que leia ou escreva dados tenant-scoped deve ocorrer dentro de uma
transação com contexto definido.

O contexto mínimo é:

- `app.orchestra_id`;
- `app.account_id`;
- `app.membership_id`, quando existir no tenant;
- `app.request_id`;
- `app.actor_kind`, como `user`, `worker`, `impersonation` ou `platform`;
- `app.impersonated_by_account_id`, quando houver impersonação.

O contexto será definido com `SET LOCAL` ou `set_config(nome, valor, true)`.
Valores transacionais impedem vazamento de contexto entre requisições quando a
conexão volta ao pool.

### Padrão de acesso

- Todo acesso tenant-scoped passa por um wrapper, por exemplo
  `withTenantContext(ctx, callback)`.
- Esse wrapper abre transação Kysely, define o contexto PostgreSQL e entrega ao
  módulo um `TenantDb` já contextualizado.
- Código de negócio tenant-scoped não recebe o `db` global diretamente.
- Consultas sem tenant usam caminhos explícitos, como `SystemDb`, limitados a
  autenticação, bootstrap público, tarefas técnicas e administração da plataforma.
- Jobs pg-boss criados por operações tenant-scoped carregam `orchestra_id` no
  payload mínimo e são criados na mesma transação da operação principal.

### Políticas RLS

- Toda tabela tenant-scoped terá RLS habilitado.
- Quando possível, `FORCE ROW LEVEL SECURITY` será usado para reduzir risco de
  bypass acidental por owner.
- Ausência de contexto deve falhar fechada: não retorna linhas e não permite
  insert/update/delete.
- Políticas usam funções auxiliares estáveis no schema `app`, por exemplo
  `app.current_orchestra_id()`.
- `USING` controla linhas visíveis ou modificáveis.
- `WITH CHECK` controla linhas que podem ser inseridas ou gravadas.
- Atualizar `orchestra_id` de uma linha tenant-scoped é proibido, salvo migração
  técnica documentada.
- Políticas devem preferir comparar colunas da própria linha com contexto atual.
- Política que consulta outras tabelas precisa ser justificada, testada contra
  concorrência e documentada, pois políticas complexas podem criar riscos sutis.

### Admin master e impersonação

- Impersonação não bypassa RLS.
- Ao impersonar, o admin master atua dentro do tenant representado, com contexto
  explícito e trilha técnica privada.
- O histórico operacional da orquestra mostra ação técnica da plataforma quando
  aplicável, sem culpar a conta representada.
- Consultas globais do admin master não usam o caminho normal de requisição de
  tenant; elas ficam em painel técnico, com autorização, auditoria e contexto
  `platform`.

### Workers

- Worker não confia cegamente no payload do job.
- Job tenant-scoped sem `orchestra_id` falha fechado.
- Worker carrega o estado atual do recurso dentro de `withTenantContext`.
- Se o recurso não pertence ao tenant do job, o job falha e gera evento técnico.
- Jobs globais, como limpeza técnica ampla, usam contexto `platform`, consultas
  próprias e escopo mínimo.

## Consequências positivas

- Bug de filtro na aplicação tende a ser barrado pelo banco.
- Risco de Broken Object Level Authorization diminui em leituras e mutações.
- Jobs assíncronos passam a ter contrato claro de tenant.
- Testes conseguem provar isolamento no nível da API e no nível do PostgreSQL.
- O modelo continua escalável sem introduzir complexidade de banco por tenant na V1.

## Custos e cuidados

- Todo acesso tenant-scoped precisa passar por transação contextualizada.
- RLS exige disciplina em migrations, grants, testes e observabilidade.
- Queries administrativas cruzadas precisam de camada de leitura própria.
- Políticas mal desenhadas podem mascarar bugs como “0 linhas afetadas”.
- Políticas complexas com subconsultas podem ter custo e risco de concorrência.

## Alternativas rejeitadas

- Apenas filtrar por `orchestra_id` na aplicação: frágil contra um único endpoint
  esquecido ou consulta manual incorreta.
- Um banco ou schema por orquestra na V1: melhora isolamento físico, mas aumenta
  custo operacional, migrações, backups e suporte antes de haver escala real.
- `SET` de sessão em vez de contexto transacional: arriscado com pool de conexões.
- Papel de aplicação com `BYPASSRLS`: elimina a principal razão de adotar RLS.
- Usar RLS para substituir toda autorização de produto: misturaria regras de
  negócio complexas no banco e dificultaria evolução.

## Testes obrigatórios

Antes de produção, os testes devem provar:

1. sem contexto de tenant, tabela tenant-scoped não retorna dados;
2. conta da orquestra A não lê recurso da orquestra B por UUID conhecido;
3. insert com `orchestra_id` diferente do contexto falha;
4. update que tenta trocar `orchestra_id` falha;
5. delete/update cruzado afeta zero linhas ou falha de forma segura;
6. conexão reutilizada não herda contexto do tenant anterior;
7. worker sem `orchestra_id` no job falha fechado;
8. worker com `orchestra_id` divergente do recurso falha fechado;
9. papel da aplicação não possui `BYPASSRLS`;
10. tabelas tenant-scoped novas falham no gate se não tiverem RLS/documentação.

## Referências

- https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html
- https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
- https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- https://www.postgresql.org/docs/current/sql-createpolicy.html
- https://www.postgresql.org/docs/current/functions-admin.html
- https://www.postgresql.org/docs/current/sql-set.html
- https://kysely.dev/docs/examples/transactions/simple-transaction
