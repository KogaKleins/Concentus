# Fluxos e experiência mobile

## 1. Navegação principal

```text
┌─────────────────────────────────┐
│ Início                          │
│                                 │
│ Urgentes e fixados              │
│ Comunicados globais             │
│ Meus naipes                     │
│ Salas temporárias               │
│                                 │
├──────┬──────────┬───────┬───────┤
│Início│Biblioteca│ Salas │Alertas│ Perfil
└──────┴──────────┴───────┴───────┘
```

A barra inferior prioriza o músico. A administração aparece por ações contextuais
e por uma área de gestão separada, não como mais um item permanente para todos.

## 2. Início sem enxurrada

```text
INÍCIO

[ Urgentes e fixados — máximo inicial visível ]

Comunicados globais
  3 mais recentes                     Ver todos

Meus naipes
  Trompetes               2 não lidos  >
  Clarinetes              1 não lido   >

Salas temporárias
  Concerto de Natal       3 novidades  >
```

- não existe feed único infinito na página inicial;
- global, naipes e salas temporárias aparecem separados;
- cada seção mostra resumo e leva ao histórico completo;
- materiais não aparecem na página inicial; ficam na Biblioteca;
- comunicados expirados somem para músicos.

## 3. Biblioteca do músico

```text
MINHA BIBLIOTECA

[ Buscar por número, título ou material ]

Materiais globais                         >
Trompete                                  >
Clarinete                                 >
Salas temporárias                         >
Compartilhados comigo                     >
```

Dentro do naipe:

```text
TROMPETE

Repertório oficial                        >
Materiais de estudo                       >
  Respiração                              >
  Embocadura                              >
```

Uma concessão individual aparece em `Compartilhados comigo`, com origem visível.
O mesmo objeto não é duplicado em resultados de busca.

## 4. Detalhe da obra

```text
55 — O Gato Branco
Repertório oficial • atualizado em 02/07/2026

Notas da obra
“Atenção à dinâmica no compasso 42.”

Seus materiais
┌─────────────────────────────────────┐
│ Trompete — 1ª voz              PDF │
│ [Abrir] [Baixar]                    │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│ Trompete — 2ª voz              PDF │
│ [Abrir] [Baixar]                    │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│ Áudio de referência            MP3 │
│ [▶ Reproduzir] [Baixar]             │
└─────────────────────────────────────┘

Última atualização
“PDF corrigido no compasso 42.”
```

PDF abre no visualizador interno e áudio toca na página. O botão de download
aparece somente quando o autor ou gestor autorizado o habilitou para o material.

## 5. Fluxo de convite

```mermaid
sequenceDiagram
    actor Maestro
    participant App as Concentus
    actor Musico as Músico
    Maestro->>App: Informa e-mail, naipes, salas e papel
    App-->>Musico: Envia convite
    Musico->>App: Abre link
    alt Conta nova
        Musico->>App: Valida e-mail e cria senha
    else Conta existente
        Musico->>App: Autentica
    end
    Musico->>App: Preenche perfil obrigatório
    App->>App: Consome convite uma única vez
    App-->>Musico: Abre orquestra correta
```

## 6. Criação e publicação de obra

```mermaid
flowchart TD
    A[Escolher biblioteca] --> B[Informar número e título]
    B --> C[Copiar formação padrão]
    C --> D[Excluir ou reincluir partes]
    D --> E[Upload em lote]
    E --> F[Associar automaticamente por aliases]
    F --> G[Revisar não associados, títulos e ordem]
    G --> H[Revisar destinatários e exceções]
    H --> I[Salvar como rascunho]
    I --> J{Publicar como?}
    J -->|Selecionados| K[Publicar partes escolhidas]
    J -->|Todos os prontos| L[Validar e publicar lote]
    K --> M[Registrar lote e notificar]
    L --> M
```

### Mesa de distribuição administrativa

```text
OBRA 55 — O GATO BRANCO

Parte                 Arquivo       Destino       Estado
Trompete — 1ª voz     tpt1.pdf      4 músicos     Publicado
Trompete — 2ª voz     tpt2.pdf      3 músicos     Rascunho
Clarinete — 1ª voz    clar1.pdf     5 músicos     Pronto
Clarinete — 2ª voz    —             4 músicos     Faltando
Tuba                   —             —             Excluído

[ Upload em lote ]  [ Salvar ]  [ Publicar todos os prontos ]
```

## 7. Ajuste de voz por obra

```mermaid
flowchart TD
    A[Abrir plano da obra] --> B[Selecionar naipe e voz]
    B --> C{Maestro já decidiu?}
    C -->|Sim| D[Líder visualiza bloqueio]
    C -->|Não| E{Ator é maestro ou líder do naipe?}
    E -->|Não| F[Negar alteração]
    E -->|Sim| G[Adicionar, trocar ou retirar músicos]
    G --> H[Salvar fotografia da obra]
```

## 8. Comunicado

```mermaid
flowchart TD
    A[Criar comunicado] --> B[Selecionar público]
    B --> C[Definir prioridade e anexos]
    C --> D[Ativar ciência, comentários, reações ou enquete]
    D --> E[Definir fixação, agenda e expiração]
    E --> F{Publicação}
    F -->|Agora| G[Publicar e notificar]
    F -->|Depois| H[Agendar]
    H --> G
    G --> I[Receber interações]
    I --> J[Encerrar, expirar ou excluir]
```

## 9. Solicitação de mudança

```mermaid
sequenceDiagram
    actor Lider as Líder/editor
    participant App as Concentus
    actor Maestro
    Lider->>App: Propõe substituição e envia novo arquivo
    App-->>Maestro: Notifica solicitação
    Maestro->>App: Compara atual, proposta e justificativa
    alt Aprovar
        Maestro->>App: Aprova
        App->>App: Substitui, publica e registra histórico
        App-->>Lider: Solicitação aprovada
    else Rejeitar
        Maestro->>App: Rejeita com motivo opcional
        App-->>Lider: Solicitação rejeitada
    end
```

## 10. Impersonação

```mermaid
sequenceDiagram
    actor Master as Admin master
    participant App as Concentus
    actor User as Conta representada
    Master->>App: Informa motivo e reautentica
    App->>App: Cria sessão curta e auditada
    App-->>Master: Exibe faixa "Visualizando como usuário"
    Master->>App: Solicita ação real
    App-->>Master: Exige confirmação adicional
    Master->>App: Confirma
    App->>App: Executa e registra ambos no log técnico
    App->>App: Registra "Ação técnica da plataforma" na orquestra
    Master->>App: Encerra impersonação
```

## 11. Estados vazios e erros obrigatórios

Cada área deve explicar o próximo passo:

- biblioteca vazia: informar que nenhum material foi liberado;
- upload não associado: explicar como escolher naipe e voz;
- obra sem parte para o usuário: não exibir a obra na biblioteca pessoal;
- convite consumido/revogado: informar o estado sem revelar dados da conta;
- acesso removido: remover conteúdo e manter notificação explicativa;
- falha de upload: preservar demais arquivos e permitir tentar novamente;
- sessão em outra orquestra: manter URL e contexto visual inequívocos.
