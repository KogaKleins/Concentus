# ADR-0017 — Baseline de segurança e threat model inicial

- Estado: Aceito
- Data: 2026-07-09

## Contexto

O Concentus é uma plataforma multi-orquestra com autenticação, perfis, arquivos
privados, permissões hierárquicas, impersonação técnica, jobs assíncronos e
conteúdo sensível por tenant. Segurança não pode ser tratada apenas como checklist
final, porque decisões de sessão, RLS, upload, auditoria e autorização afetam o
modelo de dados e os testes desde o início.

Também não queremos inventar um processo caseiro sem referência. As decisões de
segurança precisam ser embasadas e testáveis.

## Decisão

- OWASP ASVS 5.0.0 será usado como catálogo de referência para requisitos de
  segurança da V1, sem promessa de certificação formal;
- o projeto manterá um threat model vivo usando DFD e STRIDE;
- cada ameaça relevante deve registrar mitigação, teste e sinal de observabilidade;
- a V1 usará sessão opaca server-side em cookie seguro, não JWT em `localStorage`;
- cookies de sessão serão `HttpOnly`, `Secure` e `SameSite`, com escopo restrito;
- sessões serão revogáveis por dispositivo e rotacionadas em eventos sensíveis;
- requisições mutáveis protegidas por cookie exigirão defesa CSRF explícita;
- CSRF usará token no padrão sincronizado ou equivalente server-side, validação de
  `Origin`/`Referer` e Fetch Metadata quando disponível;
- senhas usarão Argon2id, parâmetros revisáveis e política alinhada a NIST/OWASP;
- MFA será obrigatório para o admin master e opcional, porém recomendado, para
  maestros/admins;
- TOTP com códigos de recuperação será o caminho inicial de MFA; passkeys/WebAuthn
  ficam previstos para evolução;
- RLS será segunda barreira de isolamento, nunca substituto da autorização da API;
- roles da aplicação não terão `BYPASSRLS`;
- workers e jobs falham fechados sem contexto explícito de tenant;
- upload seguro será tratado como superfície crítica, com allowlist, validação de
  tipo real, limites, armazenamento privado e varredura/isolamento detalhados no
  ADR-0021.

## Riscos P0 iniciais

1. vazamento entre orquestras;
2. bypass de permissão por identificador conhecido;
3. sequestro, fixação ou replay de sessão;
4. CSRF ou XSS em ação administrativa;
5. upload malicioso ou abusivo;
6. impersonação mal auditada;
7. worker/job atuando no tenant errado;
8. token de convite, recuperação ou URL assinada vazando;
9. auditoria crítica perdida, alterada ou atribuída ao ator errado.

## Embasamento

- OWASP ASVS 5.0.0 fornece uma base de requisitos verificáveis para aplicações
  web.
- OWASP Threat Modeling orienta modelar o sistema, identificar ameaças, mitigá-las
  e validar controles.
- OWASP Session Management recomenda identificadores de sessão fortes, cookies
  seguros, escopo restrito e proteção contra fixação/sequestro.
- OWASP CSRF Prevention recomenda tokens em operações mutáveis para aplicações
  com cookies, além de defesas em profundidade como origem, SameSite e Fetch
  Metadata.
- NIST SP 800-63B e OWASP Authentication sustentam senhas longas, evitar troca
  periódica arbitrária e adoção de MFA.
- PostgreSQL RLS aplica políticas por linha, mas superusers, owners e roles com
  `BYPASSRLS` podem contornar o mecanismo; por isso ele é segunda barreira.
- OWASP File Upload recomenda allowlist, validação de tipo real, nome gerado,
  limites, autorização, armazenamento fora do webroot e antivírus/sandbox quando
  aplicável.

Referências:

- https://owasp.org/www-project-application-security-verification-standard/
- https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- https://pages.nist.gov/800-63-4/sp800-63b.html
- https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
- https://www.postgresql.org/docs/current/ddl-rowsecurity.html

## Consequências positivas

- segurança passa a guiar arquitetura antes do código;
- decisões críticas ganham fonte primária, teste e observabilidade;
- a escolha por sessão server-side preserva revogação e controle operacional;
- RLS reduz impacto de erro de consulta, desde que o contexto de tenant esteja
  correto;
- upload, impersonação e jobs entram no threat model desde o começo.

## Custos e cuidados

- sessão server-side exige armazenamento, rotação e limpeza de sessões;
- CSRF adiciona contrato de frontend/API para mutações;
- RLS exige disciplina de transação, roles e testes reais em PostgreSQL;
- MFA exige fluxo de recuperação seguro, especialmente para o admin master;
- threat model precisa ser revisado quando novas superfícies surgirem;
- ASVS como referência não significa conformidade automática.

## Alternativas rejeitadas

- segurança apenas no fim da V1: tarde demais para sessão, RLS e upload;
- JWT em `localStorage`: aumenta exposição a XSS e dificulta revogação imediata;
- confiar apenas em `SameSite` contra CSRF: insuficiente como defesa única;
- confiar apenas na autorização da API sem RLS: aumenta impacto de erro de query;
- confiar apenas em RLS sem autorização da API: não cobre regra de negócio,
  hierarquia e estados;
- permitir upload amplo e bloquear extensões ruins: denylist é frágil e incompleta.
