# Roadmap e pendências

## 1. Estratégia de entrega

A comunicação interativa faz parte da V1 por ser uma dor central. Recursos
avançados de plataforma e conveniência permanecem posteriores, mas o banco deve
evitar bloqueá-los.

```mermaid
flowchart LR
    F0[F0 Documentação] --> F1[F1 Fundação multi-tenant]
    F1 --> F2[F2 Identidade e organização]
    F2 --> F3[F3 Bibliotecas e distribuição]
    F3 --> F4[F4 Comunicação]
    F4 --> F5[F5 Segurança e aceite]
    F5 --> V1[V1 em produção]
```

## 2. Fases recomendadas

### F0 — Especificação

- validar estes documentos;
- transformar regras numeradas em critérios de aceite;
- escolher stack, provedores e ambiente;
- produzir esquema lógico/SQL e contrato inicial da API;
- criar protótipo navegável mobile.

### F1 — Fundação multi-tenant

- contas globais e perfis por orquestra;
- isolamento e autorização de base;
- admin master, criação e desativação de orquestras;
- sala global automática;
- armazenamento privado e auditoria inicial.

### F2 — Identidade e organização

- convite por e-mail;
- recuperação de senha;
- perfis e campos personalizados;
- naipes, espaços, vozes e lideranças;
- formação padrão;
- painel administrativo de membros.

### F3 — Bibliotecas e distribuição

- bibliotecas, pastas, obras e materiais;
- upload simples e em lote;
- associação por aliases;
- plano de distribuição e status;
- visualização PDF, áudio e downloads;
- concessões, herança e solicitações;
- publicação agrupada e histórico.

### F4 — Comunicação

- comunicados segmentados;
- prioridade, fixação, agenda e expiração;
- ciência, comentários, anonimato e moderação;
- reações e enquetes;
- modelos e notificações internas;
- painel de pendências e respostas.

### F5 — Segurança e aceite

- testes completos de isolamento;
- hardening de upload e sessão;
- revisão de acessibilidade;
- backup/restauração;
- desempenho mobile;
- cenários de aceite com usuários reais da orquestra.

## 3. Posterior à V1

- acesso offline;
- favoritos;
- marcação “estudado”;
- respostas encadeadas;
- modelos de formação reutilizáveis;
- importação de perfil entre orquestras;
- agendamento de materiais;
- assistente inicial;
- métricas avançadas;
- interação entre orquestras;
- aplicação móvel nativa, se necessária;
- contas temporárias e ferramentas avançadas de desenvolvimento;
- impersonação completa, caso não entre na primeira entrega técnica.

## 4. Decisões abertas

Estas pendências não invalidam o modelo atual, mas precisam ser encerradas antes
das áreas correspondentes serem implementadas.

| ID | Decisão | Momento limite |
|---|---|---|
| PEN-02 | Provedor de e-mail e armazenamento de objetos | Antes da F1 |
| PEN-04 | Limites por arquivo/lote e cotas de armazenamento | Antes de uploads |
| PEN-05 | Formatos permitidos exatos e política antimalware | Antes de uploads |
| PEN-06 | Redirecionamento quando o slug da orquestra mudar | Antes de editar URL |
| PEN-08 | Retenção de backups após exclusão definitiva | Antes de produção |
| PEN-13 | Objetivos de recuperação e disponibilidade | Antes de produção |
| PEN-16 | Slots, limites e moderação de imagens institucionais por orquestra | Antes da personalização de mídia |

## 5. Decisões encerradas na consolidação de 02/07/2026

| ID anterior | Decisão confirmada |
|---|---|
| PEN-03 | MFA obrigatório para master; opcional e recomendado para maestros/admins |
| PEN-07 | Histórico da orquestra usa `Ação técnica da plataforma` |
| PEN-09 | Campo recém-obrigatório bloqueia navegação no próximo acesso até preenchimento |
| PEN-10 | Músico solicita saída; maestro/admin confirma; último admin não sai |
| PEN-11 | Sala temporária aceita responsáveis limitados ao próprio espaço |
| PEN-12 | V1 utiliza o fuso único da orquestra |
| PEN-14 | Músico edita/exclui comentário próprio enquanto interações estiverem abertas |
| PEN-15 | Mesmo peso coadministra; maior peso administra menor; menor solicita |
| PEN-01 | Next.js, NestJS, TypeScript, PostgreSQL e Kysely SQL-first; implantação inicial em monólito modular |
| FRONT-01 | URL contextual `/{orchestra_slug}`, Storybook, Tailwind, shadcn/ui e temas `system/light/dark` |
| FRONT-02 | OpenAPI, Server Components, TanStack Query, filtros na URL, SSE e painel global de uploads |
| FRONT-03 | Rascunhos privados por padrão, compartilhamento explícito e revisão otimista contra conflitos |
| FRONT-04 | React Hook Form, Zod 4 compartilhado e autosave após 1.500 ms de inatividade |
| FRONT-05 | PWA cacheia apenas assets estáticos; instalação opcional; atualização nunca interrompe trabalho |
| FRONT-06 | Download herda padrão da biblioteca, aceita exceção por material e não gera relatório ao maestro |
| SEC-01 | Log técnico de download permanece 90 dias por padrão, configurável somente pelo master |
| FRONT-07 | WCAG 2.2 AA e suporte às duas majors mais recentes dos navegadores definidos |
| QA-01 | Vitest, Storybook e Playwright; PostgreSQL real; gates diferentes em PR e release |
| QA-02 | E2E crítico cobre identidade, isolamento, publicação, hierarquia e comunicados |
| QA-03 | Testcontainers por padrão e cobertura 80/75%, com 90% de branches em segurança e ratchet |
| BACK-01 | Nove módulos iniciais, tabela com um proprietário e efeitos não críticos após commit |

## 6. Cenários essenciais de aceite da V1

### Isolamento

1. Dadas duas orquestras, um usuário da A não encontra nem acessa recurso da B,
   mesmo conhecendo seu identificador.

### Convite existente

1. Dada uma conta já validada, ao aceitar convite de outra orquestra, o usuário
   autentica sem criar nova senha e recebe apenas as associações do convite.

### Distribuição padrão

1. Dada uma obra nova, o sistema copia a formação atual.
2. Ao enviar arquivos nomeados por naipe/voz, sugere associações.
3. Ao publicar todos os prontos, ignora faltantes e excluídos.
4. Cada músico recebe uma notificação com somente suas partes.

### Precedência do maestro

1. Dada uma atribuição bloqueada pelo maestro, o líder vê a decisão, mas não pode
   substituí-la.
2. Sem decisão do maestro, o líder consegue ajustar apenas seu naipe.

### Hierarquia de conteúdo

1. Líder edita e exclui o que criou.
2. Para conteúdo criado pelo maestro, envia solicitação com proposta.
3. A aprovação aplica a mudança e notifica; a rejeição mantém o original.

4. Dois maestros/admins de mesmo peso alteram diretamente seus conteúdos.
5. Peso menor solicita alteração em conteúdo de peso maior.
6. Peso maior administra conteúdo de peso menor.

### Comunicado urgente

1. Maestro publica comunicado urgente com ciência obrigatória e enquete.
2. Músico confirma ciência, vota e pode alterar o voto enquanto aberto.
3. Maestro consulta pendentes e pode encerrar a enquete antecipadamente.

### Anonimato

1. Comentário anônimo não revela autor em nenhuma resposta da API de negócio.
2. Maestro consegue moderar sem descobrir a identidade.
3. O vínculo técnico permanece protegido e acessível somente em intervenção de
   plataforma autorizada.

## 7. Próximos artefatos recomendados

1. Matriz detalhada de permissões por ação.
2. Esquema lógico do PostgreSQL e migração inicial.
3. Contrato OpenAPI.
4. Protótipo mobile navegável.
5. Backlog em histórias verticais com critérios de aceite.
6. Registro de decisões arquiteturais (ADRs).
