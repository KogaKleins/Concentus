# Revisão de coesão da documentação — 2026-07-08

Status: Aplicada  
Escopo: documentação Markdown existente antes do bloco formal de segurança.

## 1. Objetivo

Esta revisão não valida apenas sintaxe Markdown. O objetivo é verificar se a
documentação conduz o leitor na direção correta, usando termos consistentes,
status atualizados e pendências explícitas.

## 2. O que foi revisado

- índice raiz e índice de documentação;
- foco atual e processo de decisão;
- documentos principais de produto;
- mapas de arquitetura frontend, backend e banco;
- roadmap e pendências;
- decisão e documento operacional sobre pg-boss;
- termos antigos ligados à orquestra específica para a qual o produto começou.

## 3. Correções aplicadas

| Área | Problema encontrado | Correção |
|---|---|---|
| Índice de documentação | última consolidação ainda apontava para 02/07/2026 | atualizado para 08/07/2026 |
| Índice de documentação | revisões de coesão não estavam navegáveis | adicionada seção de revisões |
| Roadmap | título das decisões encerradas dizia que tudo era da consolidação de 02/07/2026, embora inclua decisões posteriores | título tornou-se neutro: “Decisões já consolidadas” |
| Mapa de arquitetura | ordem de decisão refletia um momento anterior e podia sugerir que módulos/frontend/testes ainda vinham depois do banco | substituída por “ordem restante recomendada”, começando por segurança formal |
| Backend | índice chamava a decisão de “Outbox e fila” | renomeado para “Fila transacional”, alinhado ao ADR-0016 |
| Backend | índice não linkava o documento operacional de workers nem o ADR-0016 | links adicionados |

## 4. Coerência confirmada

- A documentação usa Concentus como plataforma e trata cada orquestra como tenant.
- Não foi encontrado uso de uma orquestra específica como regra do produto.
- A identidade visual é do Concentus; tenants fornecem contexto e mídia controlada.
- O fluxo de pg-boss está coerente: job dentro da transação, worker separado,
  payload mínimo e handlers idempotentes.
- Segurança formal continua corretamente marcada como próximo bloco, não como
  assunto já resolvido.

## 5. Pontos que permanecem pendentes por decisão, não por falha

- threat model formal;
- autenticação, sessão, cookies, CSRF, CORS, headers e rate limit;
- RLS e propagação de contexto de tenant;
- limites, cotas, formatos e antimalware de upload;
- provedor de e-mail e object storage;
- retenção de backups, RPO/RTO e operação de produção;
- erros HTTP, idempotência de comandos e observabilidade por módulo.

## 6. Leitura crítica

A documentação já está coesa o suficiente para continuar o planejamento, mas
ainda não está pronta para implementação de código de negócio. O próximo bloco de
segurança precisa transformar requisitos gerais em decisões testáveis, porque é
ele que vai proteger multitenancy, sessão, upload e auditoria.