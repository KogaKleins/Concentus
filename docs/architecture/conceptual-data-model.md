# Modelo conceitual de dados

## 1. Direção arquitetural

Recomendação inicial:

- banco relacional PostgreSQL;
- PostgreSQL como fonte de verdade do esquema e da integridade;
- Kysely como query builder SQL tipado, sem ORM tradicional;
- migrações explícitas, sequenciais, revisadas e versionadas;
- armazenamento de objetos compatível com S3 para binários;
- aplicação web responsiva/mobile-first;
- tarefas assíncronas para e-mail, processamento de upload e notificações;
- autorização sempre validada no servidor;
- isolamento por `orchestra_id`, reforçado no banco.

```mermaid
flowchart LR
    UI[Aplicação web] --> API[API da Concentus]
    API --> DB[(PostgreSQL)]
    API --> OBJ[(Armazenamento de objetos)]
    API --> Q[Fila de tarefas]
    Q --> MAIL[Provedor de e-mail]
    Q --> PROC[Processamento de arquivos]
```

A stack de aplicação e persistência foi aceita no
[ADR-0005](decisions/0005-application-stack-and-sql-first-data-access.md). Provedores
de infraestrutura continuam pendentes.

## 2. Contas e organizações

```mermaid
erDiagram
    ACCOUNT ||--o{ ORCHESTRA_PROFILE : possui
    ORCHESTRA ||--o{ ORCHESTRA_PROFILE : agrega
    ORCHESTRA ||--o{ INVITATION : emite
    ACCOUNT o|--o{ INVITATION : aceita
    ORCHESTRA_PROFILE ||--o{ PROFILE_FIELD_VALUE : preenche
    ORCHESTRA ||--o{ PROFILE_FIELD_DEFINITION : configura
    PROFILE_FIELD_DEFINITION ||--o{ PROFILE_FIELD_VALUE : recebe

    ACCOUNT {
        uuid id PK
        string email UK
        string password_hash
        datetime email_verified_at
        string status
    }
    ORCHESTRA {
        uuid id PK
        string name
        string slug UK
        string status
        string logo_key
        string timezone
    }
    ORCHESTRA_PROFILE {
        uuid id PK
        uuid account_id FK
        uuid orchestra_id FK
        string display_name
        string status
        string photo_key
        text biography
        string orchestra_role
        int authority_weight
    }
    INVITATION {
        uuid id PK
        uuid orchestra_id FK
        string email
        string token_hash UK
        string status
        datetime consumed_at
        datetime revoked_at
    }
```

Restrições essenciais:

- e-mail normalizado único globalmente;
- um perfil por `conta + orquestra`;
- nome visível normalizado único por orquestra;
- token de convite armazenado como hash, nunca em texto puro;
- convite aceito somente pela conta de e-mail correspondente.

## 3. Espaços, naipes e vozes

```mermaid
erDiagram
    ORCHESTRA ||--o{ SPACE : possui
    ORCHESTRA_PROFILE ||--o{ SPACE_MEMBERSHIP : participa
    SPACE ||--o{ SPACE_MEMBERSHIP : agrega
    SPACE ||--o{ VOICE : organiza
    VOICE ||--o{ DEFAULT_VOICE_ASSIGNMENT : atribui
    ORCHESTRA_PROFILE ||--o{ DEFAULT_VOICE_ASSIGNMENT : recebe

    SPACE {
        uuid id PK
        uuid orchestra_id FK
        string type
        string name
        boolean is_global
        string status
    }
    SPACE_MEMBERSHIP {
        uuid id PK
        uuid space_id FK
        uuid profile_id FK
        string role
        datetime starts_at
        datetime ends_at
    }
    VOICE {
        uuid id PK
        uuid space_id FK
        string name
        int sort_order
        json aliases
    }
    DEFAULT_VOICE_ASSIGNMENT {
        uuid id PK
        uuid voice_id FK
        uuid profile_id FK
    }
```

`SPACE.type` é estrutural (`GLOBAL`, `SECTION`, `TEMPORARY`), embora os nomes
mostrados ao usuário sejam configuráveis. Aliases de voz podem usar uma tabela
filha em vez de JSON na implementação final; a decisão depende da necessidade de
busca, unicidade e manutenção.

## 4. Recursos, bibliotecas e obras

Para reutilizar autoria, estado, hierarquia e concessões sem transformar todos os
dados em uma tabela genérica, recomenda-se um registro estrutural `RESOURCE_NODE`
e tabelas especializadas para os detalhes de cada tipo.

```mermaid
erDiagram
    ORCHESTRA ||--o{ RESOURCE_NODE : isola
    ORCHESTRA_PROFILE ||--o{ RESOURCE_NODE : cria
    RESOURCE_NODE o|--o{ RESOURCE_NODE : contem
    RESOURCE_NODE ||--o| LIBRARY : detalha
    RESOURCE_NODE ||--o| FOLDER : detalha
    RESOURCE_NODE ||--o| WORK : detalha
    RESOURCE_NODE ||--o| MATERIAL : detalha
    WORK ||--o{ DISTRIBUTION_SLOT : espera
    DISTRIBUTION_SLOT ||--o{ WORK_VOICE_ASSIGNMENT : distribui
    MATERIAL ||--o{ STORED_FILE : anexa
    RESOURCE_NODE ||--o{ ACCESS_GRANT : autoriza
    RESOURCE_NODE ||--o{ CHANGE_REQUEST : recebe
    WORK ||--o{ PUBLICATION_BATCH : publica
    PUBLICATION_BATCH ||--o{ PUBLICATION_ITEM : inclui

    RESOURCE_NODE {
        uuid id PK
        uuid orchestra_id FK
        uuid parent_id FK
        uuid owner_profile_id FK
        string resource_type
        string state
        int sort_order
    }
    LIBRARY {
        uuid resource_id PK
        string default_download_policy
    }
    WORK {
        uuid resource_id PK
        uuid library_id FK
        string catalog_number
        string title
        text notes
    }
    MATERIAL {
        uuid resource_id PK
        uuid work_id FK
        string display_title
        string material_type
        string download_policy
        datetime published_at
        datetime updated_at
    }
    STORED_FILE {
        uuid id PK
        uuid material_id FK
        string storage_key UK
        string original_name
        string mime_type
        bigint size_bytes
        string processing_status
    }
    DISTRIBUTION_SLOT {
        uuid id PK
        uuid work_id FK
        uuid source_space_id FK
        uuid source_voice_id FK
        string status
        boolean maestro_locked
    }
    ACCESS_GRANT {
        uuid id PK
        uuid resource_id FK
        string subject_type
        uuid subject_id
        string capability
        string source
    }
```

`LIBRARY.default_download_policy` define `allow` ou `deny`.
`MATERIAL.download_policy` define `inherit`, `allow` ou `deny`. A política efetiva
é resolvida no servidor; alterar o material exige autoridade de gerenciamento de
acesso, não apenas edição.

Restrições importantes:

- número de catálogo único por `biblioteca + número normalizado`;
- todo nó filho pertence à mesma orquestra do pai;
- um acesso nunca referencia sujeito de outra orquestra;
- `maestro_locked` impede líder de substituir decisão explícita;
- `WORK_VOICE_ASSIGNMENT` é uma fotografia; não depende ao vivo da voz padrão;
- publicações referenciam materiais e destinatários efetivos no momento do lote.

## 5. Comunicação

```mermaid
erDiagram
    ORCHESTRA ||--o{ ANNOUNCEMENT : possui
    ORCHESTRA_PROFILE ||--o{ ANNOUNCEMENT : publica
    ANNOUNCEMENT ||--o{ ANNOUNCEMENT_TARGET : destina
    ANNOUNCEMENT ||--o{ COMMENT : recebe
    ANNOUNCEMENT ||--o{ REACTION : recebe
    ANNOUNCEMENT ||--o| POLL : pode_ter
    POLL ||--o{ POLL_OPTION : oferece
    POLL_OPTION ||--o{ POLL_VOTE : recebe
    ORCHESTRA_PROFILE ||--o{ POLL_VOTE : vota
    ANNOUNCEMENT ||--o{ ACKNOWLEDGEMENT : confirma
    ORCHESTRA_PROFILE ||--o{ ACKNOWLEDGEMENT : declara
    ORCHESTRA ||--o{ PRIORITY_LEVEL : configura
    ORCHESTRA ||--o{ NOTIFICATION_TEMPLATE : configura
    ORCHESTRA_PROFILE ||--o{ NOTIFICATION : recebe

    ANNOUNCEMENT {
        uuid id PK
        uuid orchestra_id FK
        uuid author_profile_id FK
        uuid priority_id FK
        string state
        datetime scheduled_at
        datetime expires_at
        datetime pinned_until
        boolean comments_enabled
        boolean anonymous_comments
        boolean acknowledgement_required
    }
    COMMENT {
        uuid id PK
        uuid announcement_id FK
        uuid author_profile_id FK
        text body
        boolean anonymous_in_ui
        datetime edited_at
        datetime deleted_at
    }
    POLL_VOTE {
        uuid id PK
        uuid option_id FK
        uuid profile_id FK
        datetime updated_at
    }
    NOTIFICATION {
        uuid id PK
        uuid profile_id FK
        string event_type
        string rendered_title
        text rendered_body
        datetime read_at
    }
```

Regras de unicidade:

- uma confirmação por `comunicado + perfil`;
- um voto ativo por `enquete + perfil`, alterável enquanto aberta;
- notificações deduplicadas por destinatário e evento/lote;
- comentário anônimo nunca omite `author_profile_id` no armazenamento, apenas na
  projeção retornada à interface.

## 6. Auditoria e impersonação

```mermaid
erDiagram
    ACCOUNT ||--o{ IMPERSONATION_SESSION : inicia
    ACCOUNT ||--o{ IMPERSONATION_SESSION : representa
    IMPERSONATION_SESSION ||--o{ PLATFORM_AUDIT_EVENT : gera
    ORCHESTRA ||--o{ ORCHESTRA_AUDIT_EVENT : registra
    ORCHESTRA_PROFILE o|--o{ ORCHESTRA_AUDIT_EVENT : executa

    IMPERSONATION_SESSION {
        uuid id PK
        uuid master_account_id FK
        uuid represented_account_id FK
        string reason
        datetime started_at
        datetime expires_at
        datetime ended_at
    }
    ORCHESTRA_AUDIT_EVENT {
        uuid id PK
        uuid orchestra_id FK
        uuid actor_profile_id FK
        string action
        string entity_type
        uuid entity_id
        json safe_metadata
        datetime occurred_at
    }
```

O histórico da orquestra e o histórico técnico da plataforma são separados. O
detalhe de uma sessão de impersonação fica restrito ao master. No histórico
operacional, a ação aparece como `Ação técnica da plataforma`; somente o log
técnico relaciona master, conta representada e sessão.

## 7. Armazenamento e exclusão

O banco não armazena PDFs e áudios. Guarda apenas metadados e uma chave opaca do
objeto físico.

```mermaid
sequenceDiagram
    participant UI as Aplicação
    participant API as API
    participant DB as PostgreSQL
    participant S3 as Object storage
    UI->>API: Solicita upload
    API->>DB: Cria arquivo como "enviando"
    API-->>UI: Credencial temporária de upload
    UI->>S3: Envia binário
    S3-->>API: Confirmação/processamento
    API->>DB: Marca upload concluído ou falha
```

Na exclusão permanente, o objeto físico é removido. O log não conserva conteúdo
nem anexo, apenas metadados mínimos necessários à responsabilização.

## 8. Índices conceituais mínimos

- perfis por orquestra e estado;
- espaços e membros por orquestra;
- obras por biblioteca, número e título;
- materiais por obra e estado;
- slots por obra, naipe, voz e estado;
- concessões por recurso e sujeito;
- comunicados por público, estado, prioridade e data;
- notificações por destinatário e leitura;
- eventos de auditoria por orquestra, entidade e data.
