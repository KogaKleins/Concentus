# Documentação do Concentus

Esta pasta é a fonte central de verdade do produto. Decisões futuras devem ser
registradas aqui antes ou junto da implementação.

| Campo | Valor |
|---|---|
| Estado | Em definição |
| Versão-alvo do produto | 1.0.0 |
| Última consolidação | 02/07/2026 |
| Escopo documentado | Produto atual em desenvolvimento |

## Produto

Documentação viva de como o produto deve funcionar agora.

| Documento | Conteúdo |
|---|---|
| [Visão e escopo](product/vision-and-scope.md) | Objetivo, princípios, escopo da V1 e limites |
| [Papéis e permissões](product/roles-and-permissions.md) | Hierarquia, propriedade, delegação e aprovações |
| [Usuários, convites e perfis](product/users-invitations-and-profiles.md) | Identidade global, cadastro, perfil e ciclo de vida |
| [Orquestras, espaços, naipes e vozes](product/orchestras-spaces-sections-and-voices.md) | Multitenancy, salas, formação e atribuições musicais |
| [Bibliotecas, obras e materiais](product/libraries-works-and-materials.md) | Organização, upload, publicação e distribuição |
| [Comunicados e notificações](product/announcements-and-notifications.md) | Comunicados, comentários, enquetes e alertas |
| [Roadmap e pendências](product/roadmap-and-open-decisions.md) | Fases, itens adiados e decisões ainda abertas |

## Arquitetura e segurança

| Documento | Conteúdo |
|---|---|
| [Modelo conceitual de dados](architecture/conceptual-data-model.md) | Entidades, relacionamentos, isolamento e arquivos |
| [Segurança e requisitos não funcionais](architecture/security-and-non-functional-requirements.md) | Segurança, privacidade, auditoria e qualidade |
| [Mapa da arquitetura](architecture/README.md) | Estado das decisões e próximos artefatos técnicos |
| [Arquitetura do banco](architecture/database/README.md) | Convenções, modelo lógico e dicionário de dados |
| [Arquitetura frontend](architecture/frontend/README.md) | Rotas, shells, design system e showcase |
| [Arquitetura backend](architecture/backend/README.md) | Módulos, ownership, dependências e transações |
| [Estratégia de testes](architecture/testing-strategy.md) | Camadas, banco real, navegadores e gates de qualidade |
| [Decisões arquiteturais](architecture/decisions/README.md) | Por que decisões estruturais foram tomadas |

## Experiência, referência e histórico

| Documento | Conteúdo |
|---|---|
| [Fluxos e telas](ux/flows-and-screens.md) | Jornadas, navegação mobile e wireframes conceituais |
| [Glossário](reference/glossary.md) | Vocabulário oficial do domínio |
| [Versionamento](reference/versioning.md) | Releases, documentação, changelog e tags |
| [Notas de release](releases/README.md) | Comunicação amigável de versões publicadas |

## Convenções

- **Definido:** regra confirmada durante o planejamento.
- **Recomendação:** direção arquitetural adotada, sujeita a validação técnica.
- **Pendente:** decisão que ainda precisa de resposta explícita.
- Identificadores como `PER-01` e `MAT-12` tornam regras rastreáveis em código,
  testes e histórias de usuário.
- Diagramas usam Mermaid e devem continuar válidos em renderizadores Markdown
  compatíveis.

## Princípios centrais

1. Configurável sem abandonar integridade estrutural.
2. Isolamento rigoroso entre orquestras.
3. Permissões explícitas, herdáveis e auditáveis.
4. Mobile-first para o músico; produtividade em lote para administradores.
5. Uma única fonte de verdade para cada obra, sem duplicar arquivos por público.
6. Rascunho antes de publicação e histórico para ações relevantes.

## Separação de marca

- **Concentus:** única identidade visual da aplicação, nos modos claro e escuro.
- **Orquestra:** tenant armazenado no banco, identificado por nome, símbolo e URL.
- **Personalização controlada:** imagens podem ocupar slots previstos pelo produto.
- **Sem rebranding:** tenant não altera cores, tipografia, componentes ou layout.
- A documentação nunca assume que uma orquestra específica existe.

## Disciplina de mudança

Ao alterar uma regra:

1. atualizar o documento do domínio;
2. atualizar diagramas e fluxos afetados;
3. revisar o modelo conceitual;
4. ajustar cenário de aceite ou adicionar um novo;
5. atualizar `CHANGELOG.md`;
6. registrar um ADR se a decisão for estrutural;
7. registrar como pendência se a decisão ainda não estiver fechada.

Na dúvida entre documentos, a regra mais específica do domínio prevalece; uma
contradição deve ser corrigida, nunca resolvida silenciosamente no código.
