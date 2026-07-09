# Versionamento

## 1. Objetivo

Esta política separa versão do produto, documentação atual, histórico de mudanças
e justificativas arquiteturais. Ela deve ser aplicada desde a primeira release.

## 2. Fonte de verdade por necessidade

| Necessidade | Fonte |
|---|---|
| Como o produto funciona atualmente | `docs/` no branch principal |
| O que ainda não foi lançado | `CHANGELOG.md` em `Unreleased` |
| O que mudou numa versão | `CHANGELOG.md` e nota em `docs/releases/` |
| Por que uma decisão estrutural foi tomada | ADR em `docs/architecture/decisions/` |
| Estado exato de código e docs numa versão antiga | Tag Git imutável |
| Diferença linha a linha entre versões | Comparação entre tags Git |

Não serão mantidas cópias integrais como `docs/v1.0` e `docs/v1.1`. Isso duplicaria
conteúdo e permitiria divergências. Se futuramente houver um portal público que
precise mostrar versões antigas, ele será gerado a partir das tags.

## 3. Versão do produto

O Concentus usa `MAJOR.MINOR.PATCH`, adaptando
[Semantic Versioning 2.0.0](https://semver.org/) ao produto:

- **MAJOR:** mudança incompatível em API pública, contrato, integração, fluxo
  essencial ou modelo de dados que exige migração ou ação explícita;
- **MINOR:** nova funcionalidade compatível;
- **PATCH:** correção compatível, inclusive correção de segurança sem quebra de
  contrato.

Exemplos:

```text
1.0.0  primeira versão estável
1.1.0  adiciona salas temporárias sem quebrar fluxos existentes
1.1.1  corrige deduplicação de notificações
2.0.0  remove ou muda contrato público incompatível
```

## 4. Pré-releases

Antes da primeira versão estável:

```text
1.0.0-alpha.1  validação interna incompleta
1.0.0-beta.1   funcionalidades fechadas, ainda em teste
1.0.0-rc.1     candidata a produção
1.0.0          primeira versão estável
```

Uma versão publicada é imutável. Qualquer correção gera nova versão.

## 5. Tags e releases

- tags seguem `v1.0.0`, `v1.1.0` e `v1.1.1`;
- uma tag nasce somente após testes e documentação concluídos;
- uma tag nunca é movida ou recriada;
- código, migrações, documentação e changelog pertencem ao mesmo commit/tag;
- a versão da aplicação terá uma única fonte técnica quando a stack for escolhida.

## 6. Documentação viva

A documentação do branch principal descreve o produto atual ou a próxima versão
em preparação. Seu índice usa:

```text
Estado: Em definição | Em desenvolvimento | Estável
Versão-alvo: 1.0.0
Última revisão: AAAA-MM-DD
```

Não existe número independente de “versão documental”. Alterações puramente
editoriais ficam registradas no Git, mas não alteram a versão do produto.

Toda mudança funcional deve atualizar, no mesmo trabalho:

1. regra de produto;
2. fluxos e diagramas afetados;
3. modelo ou arquitetura, quando aplicável;
4. cenário de aceite;
5. seção `Unreleased` do changelog;
6. ADR, se a mudança representar decisão estrutural relevante.

## 7. Changelog

O `CHANGELOG.md` contém mudanças relevantes para humanos, não uma cópia de commits.
Segue as categorias:

- `Added`;
- `Changed`;
- `Deprecated`;
- `Removed`;
- `Fixed`;
- `Security`.

Durante o desenvolvimento, mudanças ficam em `Unreleased`. Na release:

1. definir a versão;
2. mover as entradas para uma seção com versão e data `AAAA-MM-DD`;
3. criar a nota de release;
4. atualizar a versão-alvo da documentação;
5. criar a tag Git.

## 8. Notas de release

Notas em `docs/releases/` são voltadas a usuários e administradores. Explicam
novidades, impacto, migração e limitações conhecidas. Não substituem o changelog.

## 9. ADRs

ADRs são numerados e não reescritos depois de aceitos, exceto correções editoriais.
Se uma decisão mudar, um novo ADR substitui o anterior e ambos preservam o
histórico.

Estados permitidos:

- `Proposto`;
- `Aceito`;
- `Rejeitado`;
- `Substituído por ADR-NNNN`;
- `Obsoleto`.

## 10. Migrações e API

- migrações de banco são sequenciais, append-only após compartilhadas;
- uma migração não é renomeada ou editada depois de publicada;
- versionamento de API é independente da URL do tenant;
- uma futura `/api/v1` representa contrato de API, não versão visual do produto;
- descontinuação deve ser anunciada antes de remoção incompatível.

