# ADR-0002 — Conta global e perfis por orquestra

- Estado: Aceito
- Data: 2026-07-02

## Contexto

Uma pessoa pode integrar várias orquestras. Credenciais duplicadas prejudicariam a
experiência, enquanto um perfil único compartilharia dados indevidamente.

## Decisão

- e-mail e senha pertencem a uma conta global;
- cada associação cria um perfil isolado dentro da orquestra;
- dados e permissões de uma orquestra não são copiados para outra;
- convite para conta existente adiciona somente a nova associação.

## Consequências

- usuário mantém uma credencial e alterna o contexto e a URL;
- recuperação de senha é global e não fica sob controle de uma orquestra;
- autorização sempre combina conta, perfil e tenant atual.

