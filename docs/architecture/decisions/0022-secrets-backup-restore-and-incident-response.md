# ADR-0022 — Segredos, backup, restore e resposta a incidentes

- Estado: Aceito
- Data: 2026-07-09

## Contexto

O Concentus lidará com contas, permissões, materiais privados, comunicados,
arquivos, logs técnicos e dados de múltiplas orquestras. Mesmo com autorização,
RLS, rate limit e upload seguro, a plataforma ainda pode falhar por segredos
vazados, backup inexistente, restore não testado, logs insuficientes ou resposta
improvisada a incidente.

Na V1, a infraestrutura ainda pode ser simples e até rodar em servidor próprio,
mas os controles operacionais precisam nascer claros. Uma plataforma pequena não
precisa começar com um SOC completo; precisa, porém, conseguir responder a três
perguntas sem pânico:

1. qual segredo vazou e como rotacionar;
2. qual backup restaura o serviço e em quanto tempo;
3. quem faz o quê durante um incidente.

## Decisão

### Segredos

- Nenhum segredo real será commitado no repositório.
- `.env.example` documenta nomes e formato, nunca valores reais.
- `.env.local` é permitido apenas para desenvolvimento local e deve ser ignorado
  pelo Git.
- Produção usa fonte de segredos operacional, como secret manager, cofre de
  senhas com deploy controlado, Vault, SOPS/age ou equivalente documentado.
- Segredos de desenvolvimento, homologação e produção são separados.
- Produção não reutiliza credenciais de desenvolvimento.
- A aplicação não escreve segredos em logs, payloads de job, erros, screenshots ou
  documentação.
- CI/CD executa verificação de segredo antes de aceitar código.

### Inventário mínimo de segredos

O inventário operacional deve listar, no mínimo:

- credenciais PostgreSQL por papel;
- segredo/HMAC de sessão;
- chaves de assinatura ou criptografia;
- credenciais de object storage;
- credenciais SMTP/provedor de e-mail;
- credencial do scanner antimalware se houver;
- tokens de deploy/CI;
- credenciais de backup;
- chaves de criptografia dos backups;
- credenciais de monitoramento.

Cada segredo deve possuir dono, ambiente, finalidade, local de armazenamento,
procedimento de rotação, impacto de vazamento e última rotação conhecida.

### Rotação e revogação

- Segredo vazado é revogado, não apenas removido do código.
- Rotação deve ser ensaiada para segredos críticos antes da produção.
- Credenciais de banco e storage devem permitir rotação sem parada longa quando
  possível.
- Segredos de longa duração são aceitos na V1 apenas quando inevitáveis e
  documentados.
- Tokens temporários e credenciais de escopo curto são preferidos.
- Recuperação emergencial de segredo exige registro técnico.

### Logs e evidências

- Logs de segurança são estruturados, pesquisáveis e correlacionáveis por
  `request_id`.
- Logs não armazenam senha, token, segredo, código MFA, URL assinada, conteúdo de
  arquivo ou comentário completo.
- Logs de auditoria de negócio e logs técnicos restritos são separados.
- Eventos de segurança relevantes são preservados contra alteração pela aplicação
  comum.
- Relógio dos servidores deve estar sincronizado.
- Durante incidente, logs e artefatos relevantes são preservados antes de limpeza.

### Backup

- Backup automatizado é obrigatório antes de produção.
- Backup sem restore testado não conta como controle fechado.
- PostgreSQL de produção usa backup físico/base backup com WAL archiving para
  permitir point-in-time recovery.
- Dumps lógicos podem existir como camada auxiliar de portabilidade, mas não
  substituem PITR.
- Object storage possui backup/versionamento/replicação conforme infraestrutura
  final.
- Configurações operacionais e manifests de backup também são protegidos.
- Segredos e chaves de backup não ficam armazenados no mesmo local dos backups
  criptografados.
- Backups são criptografados antes de sair do ambiente confiável ou protegidos por
  mecanismo equivalente do provedor.

### RPO e RTO iniciais

Alvos iniciais da V1:

| Escopo | RPO | RTO |
|---|---:|---:|
| PostgreSQL | 15 minutos | 4 horas |
| Object storage | 24 horas | 8 horas |
| Configuração e segredos | 24 horas | 4 horas |
| Plataforma completa | 24 horas | 8 horas |

Se esses alvos não forem atingidos antes de produção, o risco precisa ser aceito
explicitamente e registrado.

### Restore

- Restore é testado em ambiente isolado antes da produção.
- Restore é revalidado pelo menos mensalmente ou antes de release candidata.
- Teste de restore deve provar login, isolamento de tenant, leitura de biblioteca,
  acesso a arquivo autorizado e ausência de envio real de e-mail para usuários.
- Restore nunca sobrescreve produção sem runbook e confirmação explícita.
- Ambientes de desenvolvimento não recebem dados reais de produção sem
  anonimização ou justificativa registrada.

### Retenção de backups

- Janela PITR inicial do PostgreSQL: 14 dias.
- Retenção rolling de backups criptografados de produção: 30 dias.
- Retenção rolling de versões/cópias do object storage: 30 dias.
- Backups mensais de longa duração não ficam habilitados por padrão na V1; exigem
  decisão de privacidade e retenção própria.
- Exclusão definitiva remove dado do ambiente ativo imediatamente, mas o dado pode
  permanecer em backups criptografados até a expiração natural da janela de 30
  dias.
- Backups não são editados individualmente para remover uma linha/arquivo, salvo
  exigência legal ou incidente específico com procedimento documentado.

### Incidentes

O Concentus adotará ciclo inspirado no NIST:

1. preparação;
2. detecção e análise;
3. contenção;
4. erradicação;
5. recuperação;
6. pós-incidente.

Todo incidente relevante deve ter:

- identificador;
- severidade;
- responsável;
- linha do tempo;
- decisões tomadas;
- evidências preservadas;
- impacto estimado;
- ações de contenção;
- ações de recuperação;
- revisão pós-incidente.

### Severidades

| Severidade | Exemplo | Resposta |
|---|---|---|
| SEV-1 | vazamento entre orquestras, perda de dados, segredo de produção vazado | resposta imediata, contenção e comunicação coordenada |
| SEV-2 | conta administrativa comprometida, backup falhando, malware publicado | resposta urgente no mesmo dia |
| SEV-3 | abuso localizado, falha parcial de worker, alerta de scanner | triagem e correção planejada |
| SEV-4 | evento suspeito sem impacto confirmado | monitorar e registrar |

### Comunicação

- Comunicação externa de incidente é coordenada, factual e registrada.
- Maestros/admins afetados recebem comunicação quando houver impacto real ou
  obrigação operacional.
- Notificação legal ou regulatória depende do caso e deve ser avaliada com apoio
  jurídico quando aplicável.
- Não prometer causa, escopo ou prazo antes de análise suficiente.

### Produção

Antes da primeira produção real, precisam estar prontos:

1. inventário de segredos;
2. política de rotação;
3. backup automatizado;
4. restore testado;
5. alvos RPO/RTO aceitos;
6. alertas mínimos;
7. runbook de incidente;
8. caminho de break-glass do admin master;
9. separação de ambientes;
10. secret scanning no pipeline.

## Consequências positivas

- Reduz risco de vazamento operacional por segredo perdido.
- Torna recuperação uma prática testada, não uma esperança.
- Define expectativa realista de perda e tempo de recuperação.
- Ajuda a preservar evidência sem vazar dados sensíveis.
- Evita improviso em incidente envolvendo contas, arquivos ou tenants.

## Custos e cuidados

- Exige disciplina operacional antes de existir código de negócio.
- Backup com PITR aumenta armazenamento e monitoramento.
- Restore mensal consome tempo, mas revela falhas cedo.
- Segredos em servidor próprio exigem cuidado equivalente a serviço gerenciado.
- Comunicação de incidente exige cautela para não gerar ruído ou prometer demais.

## Alternativas rejeitadas

- Usar apenas `.env` manual em produção: frágil e difícil de auditar.
- Backup manual ocasional: não atende produção.
- Apenas `pg_dump` como recuperação principal: útil, mas insuficiente para RPO de
  minutos e point-in-time recovery.
- Replicação como substituto de backup: replica corrupção, exclusão e erro humano.
- Guardar chave de backup junto do backup: elimina o ganho da criptografia.
- Resolver incidente “quando acontecer”: incompatível com plataforma multi-tenant.

## Testes obrigatórios

Antes de produção, testes e ensaios devem provar:

1. secret scanning falha ao detectar segredo de teste commitado;
2. aplicação não inicia em produção sem segredo obrigatório;
3. logs mascaram tokens, senhas e URLs assinadas;
4. backup PostgreSQL é gerado automaticamente;
5. WAL archiving está funcionando e monitorado;
6. restore isolado atinge RPO/RTO definidos ou registra exceção;
7. arquivo do object storage restaurado corresponde ao metadado do banco;
8. rotação de credencial crítica tem runbook testado;
9. incidente SEV-1 simulado gera linha do tempo e ações;
10. admin master possui caminho de break-glass auditável.

## Referências

- https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- https://csrc.nist.gov/pubs/sp/800/61/r2/final
- https://csrc.nist.gov/pubs/sp/800/92/final
- https://www.postgresql.org/docs/current/backup.html
- https://www.postgresql.org/docs/current/continuous-archiving.html
- https://www.postgresql.org/docs/current/app-pgdump.html
