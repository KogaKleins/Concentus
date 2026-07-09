# Segurança e requisitos não funcionais

## 1. Objetivos

1. Impedir vazamento entre orquestras.
2. Tornar toda decisão de acesso reproduzível e auditável.
3. Proteger contas administrativas e links sensíveis.
4. Manter arquivos privados mesmo quando armazenados fora do banco.
5. Funcionar bem em celulares e conexões móveis comuns.

## 2. Autenticação

- e-mail normalizado e verificado;
- senha armazenada com algoritmo moderno de derivação, preferencialmente
  Argon2id, com parâmetros revisáveis;
- recuperação com token curto, de uso único e com expiração;
- troca de e-mail exige verificação do novo endereço;
- sessões revogáveis e rotação segura de tokens;
- admin master usa conta separada da conta cotidiana de músico;
- senhas nunca são visíveis a maestro, admin ou desenvolvedor.
- autenticação multifator é obrigatória para o admin master;
- MFA é opcional, mas fortemente recomendado, para maestros/admins.

## 3. Convites

Embora o convite de membro não expire por decisão de produto, ele deve:

- possuir alta entropia;
- ser armazenado apenas como hash;
- estar vinculado ao e-mail convidado;
- ser consumido atomicamente uma única vez;
- poder ser revogado e reenviado;
- não revelar se outro e-mail possui conta.

## 4. Autorização

- negação por padrão;
- validação no servidor em toda leitura, download e mutação;
- cliente nunca é fonte de autoridade;
- acesso efetivo considera tenant, papel, contexto, herança, concessões, bloqueios
  e estado do recurso;
- alterações críticas usam transação para impedir publicação parcial;
- mudança de liderança invalida imediatamente permissões derivadas;
- URLs de arquivo são temporárias, assinadas e emitidas somente após autorização.

## 5. Isolamento multi-tenant

- `orchestra_id` obrigatório nas tabelas de domínio;
- chaves e restrições impedem relacionamentos cruzados;
- políticas de banco, como Row-Level Security do PostgreSQL, são recomendadas como
  segunda barreira;
- jobs assíncronos carregam explicitamente o tenant;
- cache e armazenamento usam namespace por orquestra;
- testes automatizados tentam acessar IDs pertencentes a outro tenant.

## 6. Uploads

- validar extensão, MIME real e assinatura do arquivo;
- limitar tamanho por arquivo, lote e orquestra;
- nomes originais nunca viram caminhos físicos;
- servir arquivos com cabeçalhos seguros;
- processar PDFs, imagens e áudio em ambiente isolado;
- prever varredura antimalware;
- apagar uploads incompletos ou órfãos após janela configurada pela orquestra;
- notificar o proprietário com antecedência configurável antes da limpeza; se ele
  estiver inativo, encaminhar a pendência aos maestros/admins;
- registrar hash do conteúdo para integridade e possível deduplicação futura.

> **Pendente:** limites de tamanho, formatos exatos e cotas por orquestra.

## 7. Privacidade

- e-mail visível somente para maestro/admin daquela orquestra;
- campos de perfil respeitam visibilidade por valor/campo;
- interfaces nunca retornam autor de comentário anônimo;
- comentário “anônimo” deve ser descrito como anônimo na interface e tecnicamente
  rastreável;
- logs não armazenam senha, token, conteúdo integral sensível ou URL assinada;
- dados de uma orquestra não são reutilizados por outra sem ação do usuário.

## 8. Auditoria

Eventos mínimos:

- login administrativo e falhas relevantes;
- criação, promoção, desativação e reativação de perfil;
- alterações de naipe, voz, liderança e permissões;
- criação, publicação, retirada e exclusão de recursos;
- concessão e remoção de acesso;
- aprovação/rejeição de solicitações;
- exclusão/moderação de comentário;
- exclusão definitiva de comunicado e arquivo;
- início, ação crítica e término de impersonação.

Logs são append-only para a aplicação comum, datados, pesquisáveis e separados em
escopo da orquestra e escopo técnico da plataforma.

## 9. Impersonação

- somente admin master;
- reautenticação recente;
- motivo obrigatório;
- duração curta;
- banner persistente e saída imediata disponível;
- segunda confirmação para ações mutáveis;
- registro do master e da conta representada;
- impossibilidade de impersonar sem deixar trilha técnica.
- histórico da orquestra mostra `Ação técnica da plataforma`, enquanto o vínculo
  com master e conta representada permanece no histórico técnico restrito.

## 10. Exclusão e retenção

- desativar conta ou orquestra não apaga autoria e histórico;
- exclusão permanente de arquivo remove o binário;
- log mínimo preserva identificador, ação, responsáveis, data e motivo;
- logs não devem permitir reconstruir o conteúdo excluído;
- downloads geram somente log técnico restrito e temporário; não há relatório de
  comportamento do músico para maestros na V1;
- logs técnicos de download permanecem por 90 dias por padrão; somente o admin
  master pode alterar globalmente o prazo;
- política de retenção e backup precisa respeitar exclusões definitivas após a
  janela técnica inevitável de backup.
- rascunhos de negócio não expiram automaticamente na V1; a limpeza automática
  limita-se a arquivos técnicos incompletos ou órfãos.

## 11. Disponibilidade e recuperação

- backups automatizados e testados por restauração;
- versionamento/replicação do armazenamento conforme custo definido;
- monitoramento de falhas de login, upload, processamento, e-mail e notificação;
- idempotência em publicação e envio de notificações;
- uma falha num arquivo do lote não invalida uploads bem-sucedidos;
- plano de recuperação com objetivos de perda e tempo ainda a definir.

## 12. Desempenho

- paginação em comunicados, notificações, auditoria e bibliotecas extensas;
- busca indexada por número, título e nome de material;
- miniaturas e pré-visualizações geradas assincronamente;
- downloads diretos do armazenamento somente após autorização de acesso e
  verificação de que o material permite download;
- publicação em lote processada sem bloquear a interface;
- notificações deduplicadas e agregadas antes de persistir.

## 13. Acessibilidade e experiência

- conformidade-alvo WCAG 2.2 nível AA;
- mobile-first sem depender exclusivamente de hover;
- navegação por teclado no desktop;
- contraste adequado e prioridade nunca indicada apenas por cor;
- foco visível, rótulos acessíveis e mensagens de erro acionáveis;
- tamanho de toque confortável;
- PDF e áudio com alternativa de download quando habilitada pelo responsável;
- datas e horários coerentes com o fuso da orquestra/usuário.

Navegadores formalmente suportados: major atual e anterior de Chrome, Edge e
Firefox, além da major atual e anterior do Safari no macOS e iOS. A ausência de
recurso essencial deve gerar explicação clara, não falha silenciosa.

## 14. Qualidade

Testes mínimos:

- unidade para precedência de permissões;
- integração para convite, publicação, retirada e aprovação;
- isolamento entre tenants;
- matrizes de papel x recurso x autoria;
- concorrência em convite de uso único e voto alterável;
- E2E mobile para jornadas de músico, líder e maestro;
- testes de autorização de downloads e comentários anônimos;
- testes de herança e exceção da política de download, incluindo negação a editor
  sem capacidade de gerenciar acesso.
