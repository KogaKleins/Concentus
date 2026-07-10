# Revisão minuciosa do pacote de segurança

Status: Aplicada  
Data: 2026-07-09  
Escopo: documentos e ADRs do pacote de segurança da V1

## 1. Objetivo

Esta revisão fez o pente fino do pacote de segurança antes de mudar o foco para
modelo lógico relacional, migrações, seeds e dicionário de dados.

O objetivo não foi “achar um jeito de aprovar” o pacote. Foi procurar falhas de
lógica, ambiguidades, decisões frágeis, referências insuficientes e frases que
poderiam guiar a implementação para o caminho errado.

## 2. Artefatos revisados

- `docs/architecture/security/README.md`
- `docs/architecture/security/threat-model.md`
- `docs/architecture/security/session-cookies-and-csrf.md`
- `docs/architecture/security/rls-and-tenant-context.md`
- `docs/architecture/security/rate-limit-and-abuse.md`
- `docs/architecture/security/secure-uploads-and-antimalware.md`
- `docs/architecture/security/secrets-backup-and-incident-response.md`
- `docs/architecture/security/security-tests-and-acceptance.md`
- ADRs 0017 a 0023
- documentos de apoio afetados: foco atual, estratégia de testes, requisitos não
  funcionais, índice geral e roadmap.

## 3. Método

1. Conferir o pacote contra fontes primárias e referências reconhecidas.
2. Revisar coerência entre threat model, ADRs, documentos operacionais e foco
   atual.
3. Procurar estados desatualizados, termos futuros que já viraram decisão e
   pendências que pareciam regra fechada.
4. Validar se cada risco P0 tinha mitigação, teste mínimo e observabilidade.
5. Separar o que está fechado em nível arquitetural do que ainda depende de
   implementação, CI, infraestrutura e ensaio real.

## 4. Fontes usadas como embasamento

| Área | Referências principais |
|---|---|
| Baseline e verificação | OWASP ASVS 5.0.0; NIST SSDF SP 800-218; OWASP WSTG |
| Threat modeling | OWASP Threat Modeling Cheat Sheet; OWASP Abuse Case Cheat Sheet |
| Multi-tenancy | OWASP Multi-Tenant Application Security; OWASP API Security BOLA/resource consumption; PostgreSQL RLS |
| Autenticação | NIST SP 800-63B; OWASP Authentication; Password Storage; Session Management; CSRF Prevention |
| Uploads | OWASP File Upload; OWASP Unrestricted File Upload; ClamAV; EICAR; RFC 6266 |
| Abuso e rate limit | OWASP Credential Stuffing; Forgot Password; Denial of Service; RFC 6585; RFC 9110 |
| Operação | OWASP Secrets Management; OWASP Logging; NIST SP 800-61; NIST SP 800-92; PostgreSQL Backup/PITR |
| Testes | ZAP Baseline/API Scan; Playwright; Vitest |

## 5. Correções aplicadas

| Tipo | Correção | Motivo |
|---|---|---|
| Estado documental | `security/README.md` passou de “Em definição formal” para “Fechado em nível arquitetural para V1” | O pacote já estava completo como arquitetura; manter “em definição” confundiria o leitor |
| Foco atual | removida a orientação de não detalhar tabelas, pois o próximo bloco é exatamente banco/modelagem | Evitar contradição direta com o próximo passo |
| Temporalidade | `futura RLS` foi trocado por `RLS definida/obrigatória` onde a decisão já estava aceita | Evitar leitura de que RLS ainda é opcional |
| Threat model | corrigido escopo para incluir rate limit, antimalware, segredos, backup, incidentes e RLS obrigatória | O threat model precisava espelhar o pacote completo |
| Pontuação/sintaxe | corrigidas listas com ponto final antes de novos itens e pontuação inconsistente | Reduzir ruído de leitura e ambiguidade |
| Referências | adicionadas referências de NIST SSDF, OWASP Multi-Tenant Security, OWASP Abuse Case e EICAR | Embasar melhor gates, multi-tenancy, abuso e teste antimalware |
| Índices | índice geral agora descreve a arquitetura de segurança como controles e critérios de aceite, não “próximos controles” | Evitar estado antigo no índice |
| Uploads | “janela configurada pela orquestra” virou “janela operacional documentada” | Evitar prometer uma configuração de produto ainda não modelada |

## 6. Análise de coerência

### 6.1. Sessão, cookies e CSRF

Coerente. A decisão por sessão opaca server-side, cookie `__Host-`, `HttpOnly`,
`Secure`, `SameSite=Lax`, rotação, revogação e CSRF explícito está alinhada com
ASVS, OWASP Session Management, OWASP CSRF e MDN.

Ponto importante: `SameSite` continua corretamente tratado como defesa adicional,
não como substituto do token CSRF.

### 6.2. RLS e tenant context

Coerente. A modelagem usa RLS como segunda barreira, não como substituto da
autorização de produto. Isso evita dois erros clássicos:

1. confiar somente na API e deixar uma query esquecida vazar dados;
2. colocar toda regra de negócio no banco e criar políticas complexas demais.

O uso de `set_config(..., true)`/`SET LOCAL` está correto porque o contexto fica
preso à transação e não vaza pelo pool de conexões.

Ponto de atenção para implementação: políticas que consultarem outras tabelas
precisarão de cuidado especial, porque o próprio PostgreSQL documenta riscos de
concorrência e vazamento sutil em políticas complexas.

### 6.3. Rate limit e abuso

Coerente. A decisão de começar com PostgreSQL em vez de Redis é aceitável para a
escala inicial, desde que o pacote mantenha a observação já documentada: se houver
múltiplas instâncias ou gargalo, Redis/serviço dedicado exige novo ADR.

O modelo evita os dois extremos ruins: nenhum limite na V1 ou CAPTCHA obrigatório
em todo login.

### 6.4. Upload seguro e antimalware

Coerente. O pacote segue defesa em profundidade: allowlist, validação de extensão,
MIME apenas como sinal fraco, magic bytes, hash, quarentena, antimalware,
processamento em worker e storage privado.

Ponto importante: ClamAV é recomendado como implementação inicial, mas a arquitetura
mantém interface abstrata para troca futura. Isso é bom: evita acoplamento
prematuro sem adiar o controle obrigatório.

### 6.5. Segredos, backup, restore e incidentes

Coerente. O pacote deixa claro que backup sem restore testado não é controle
fechado. Essa frase é central e deve permanecer como princípio de implementação.

RPO/RTO estão razoáveis como alvo inicial, mas dependem de servidor/provedor real.
Por isso ficaram corretamente condicionados a ensaio e aceite explícito antes da
produção.

### 6.6. Testes e gates

Coerente. O pacote não confunde scanner com prova de regra de negócio. ZAP entra
como complemento, enquanto autorização, RLS, tenant, CSRF e upload hostil exigem
testes próprios.

O NIST SSDF reforça a direção de integrar práticas seguras ao ciclo de
desenvolvimento, não tratar segurança como auditoria tardia.

## 7. Buracos ou riscos residuais reais

Nenhum bloqueador lógico foi encontrado após as correções. Ainda existem pendências,
mas elas são de implementação ou infraestrutura, não de regra desconhecida:

1. provedor de object storage;
2. infraestrutura concreta do scanner antimalware;
3. ferramenta de secret management;
4. ferramenta de backup PostgreSQL;
5. provedor de CI e retenção de artefatos;
6. ferramenta de secret scanning;
7. política final de CSP quando o frontend existir;
8. procedimento real de break-glass/MFA do admin master;
9. calibração de limites após ensaio com usuários reais.

Essas pendências não impedem o próximo bloco de banco. Elas devem reaparecer antes
da fundação operacional, uploads reais, CI e produção.

## 8. Testes mentais de falha

| Cenário | Resultado esperado pela documentação | Avaliação |
|---|---|---|
| API esquece filtro de tenant | RLS bloqueia ou retorna zero linhas | Coberto |
| RLS sem contexto | falha fechada | Coberto |
| Worker recebe job sem tenant | falha fechada e dead letter sem segredo | Coberto |
| Scanner fora em produção | arquivo não promove | Coberto |
| Maestro tenta ver relatório de downloads | não existe na V1; logs são técnicos | Coberto |
| Upload malicioso renomeado | assinatura/magic bytes e scanner rejeitam | Coberto |
| Backup existe, mas restore nunca foi feito | produção bloqueada | Coberto |
| ZAP passa, mas regra de negócio falha | testes próprios continuam obrigatórios | Coberto |
| Exceção de segurança sem prazo | não aceita | Coberto |
| Admin master impersona e altera algo | histórico operacional não culpa o usuário representado; log técnico restrito preserva vínculo | Coberto |

## 9. Conclusão

O pacote de segurança está coeso para a fase atual: documentação e arquitetura
antes de código.

Ele não prova que a plataforma implementada será segura. O que ele faz corretamente
é definir controles, limites, testes e evidências que a implementação futura terá
de satisfazer.

Próximo foco recomendado permanece:

1. modelo lógico relacional;
2. migrações;
3. seeds;
4. dicionário de dados;
5. depois, contrato inicial da API.

## 10. Referências

- https://owasp.org/www-project-application-security-verification-standard/
- https://csrc.nist.gov/pubs/sp/800/218/final
- https://owasp.org/www-project-web-security-testing-guide/
- https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html
- https://pages.nist.gov/800-63-4/sp800-63b.html
- https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload
- https://docs.clamav.net/manual/Usage/Scanning.html
- https://www.eicar.org/download-anti-malware-testfile/
- https://www.rfc-editor.org/rfc/rfc6585.html#section-4
- https://www.rfc-editor.org/rfc/rfc9110.html#name-retry-after
- https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
- https://csrc.nist.gov/pubs/sp/800/61/r2/final
- https://csrc.nist.gov/pubs/sp/800/92/final
- https://www.postgresql.org/docs/current/continuous-archiving.html
- https://www.zaproxy.org/docs/docker/baseline-scan/
- https://www.zaproxy.org/docs/docker/api-scan/
