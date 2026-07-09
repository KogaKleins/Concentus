# ADR-0003 — Autorização hierárquica

- Estado: Aceito
- Data: 2026-07-02

## Contexto

Maestros, líderes e membros precisam colaborar sem permitir que uma autoridade
menor ultrapasse decisões superiores. Também pode haver maestros/admins com pesos
diferentes.

## Decisão

- capacidades de recurso são separadas de papéis;
- maior peso administra conteúdo de menor peso;
- pesos iguais coadministram;
- menor peso solicita alteração ao maior;
- somente o admin master altera pesos;
- editar não concede automaticamente gerenciar acesso;
- decisões do maestro numa obra prevalecem sobre líder e padrões.

## Consequências

- autorização exige comparação explícita de contexto, peso, autoria e concessões;
- a matriz é previsível, mas precisa de testes combinatórios;
- solicitações preservam colaboração sem romper a hierarquia.

