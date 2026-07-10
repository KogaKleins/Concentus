# Revisão final pré-segurança — 2026-07-08

Status: Aplicada  
Escopo: todos os documentos Markdown atuais do Concentus antes do bloco formal de segurança.

## 1. Critério da revisão

Esta revisão procurou problemas que poderiam induzir implementação errada:

- decisões antigas tratadas como atuais;
- termos ambíguos ou nomes de papel inconsistentes;
- diagramas contradizendo texto normativo;
- roadmap apontando uma ordem errada de trabalho;
- links locais quebrados;
- blocos Markdown mal fechados;
- numeração de regras difícil de seguir;
- pendências reais escondidas como se já estivessem resolvidas.

## 2. Correções aplicadas

| Área | Problema | Correção |
|---|---|---|
| Papéis | `admin master/dev` parecia dois papéis ou um nome informal | papel oficial padronizado como `Admin master`; a descrição explica que é conta técnica de plataforma |
| Papéis | regras `PER` apareciam visualmente fora de ordem | regras foram renumeradas de forma sequencial dentro do documento |
| Impersonação | algumas frases podiam sugerir que o histórico da orquestra culparia o usuário representado | ações reais agora registram os dois atores apenas no histórico técnico restrito; a orquestra vê `Ação técnica da plataforma` |
| Roadmap | F5 parecia colocar segurança somente no fim | esclarecido que segurança formal ocorre na F0; F5 é hardening, aceite e operação |
| Roadmap | stack aparecia como escolha pendente | ajustado para provedores e ambiente, pois a stack já foi decidida |
| Roadmap | impersonação parecia posterior à V1 apesar de já estar especificada | mantida a impersonação auditada na V1; adiadas apenas ferramentas avançadas além dela |
| Backend | diagrama criava job depois do commit | diagrama corrigido: job nasce antes do commit, na mesma transação |
| ADR-0016 | tabela dizia `Job após commit`, ambíguo | tabela agora explicita: job nasce na transação e executa após commit |
| Backend | link duplicado no índice | duplicidade removida |
| Segurança | fuso da orquestra/usuário contradizia a V1 | corrigido para fuso da orquestra na V1; fuso por usuário fica posterior |
| Changelog | decisões recentes não estavam registradas | adicionados pg-boss, worker e revisões de documentação |
| Navegação | README raiz não apontava primeiro para o foco atual | `Foco atual` virou a primeira leitura recomendada |

## 3. Coerência confirmada

- Concentus permanece plataforma; orquestras são tenants isolados e não redefinem a identidade visual.
- A URL por slug resolve contexto público antes da autenticação sem expor dados internos.
- Conta global e perfil por orquestra continuam separados.
- Convite é por e-mail, uso único, sem expiração e revogável.
- Bibliotecas, rascunhos, compartilhamento e política de download estão alinhados com a hierarquia.
- SSE não escreve dados; apenas avisa telas para buscar o estado canônico.
- PWA não cacheia PDFs, áudios, imagens privadas nem respostas autenticadas da API.
- pg-boss está alinhado à regra transacional: estado principal, auditoria crítica e job confirmam juntos.
- Testes continuam exigindo PostgreSQL real para persistência, RLS, constraints e transações.

## 4. Pendências reais que continuam abertas

Estas pendências são intencionais e não devem ser implementadas por suposição:

1. threat model formal;
2. sessão, cookies, CSRF, CORS, headers e rate limit;
3. RLS e propagação segura de tenant context;
4. provedor de e-mail e object storage;
5. limites, cotas, formatos permitidos e antimalware de upload;
6. retenção de backups, RPO/RTO e restauração testada;
7. erros HTTP, idempotência de comandos e observabilidade por módulo;
8. política de CI, paralelismo e retenção de artefatos;
9. slots finais de mídia institucional por orquestra.

## 5. Resultado

A documentação está coesa para avançar ao bloco formal de segurança. Ainda não é
momento de implementar código de negócio, porque o isolamento, sessão, upload e
RLS precisam virar decisões testáveis primeiro.