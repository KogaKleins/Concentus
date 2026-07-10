# ADR-0018 — Autenticação, sessões, cookies e CSRF

- Estado: Aceito
- Data: 2026-07-09

## Contexto

O Concentus usará autenticação por e-mail e senha, com MFA obrigatório para o
admin master e opcional para maestros/admins. A aplicação web será uma PWA e usará
cookie para sessão. Isso torna revogação e experiência mobile melhores, mas exige
cuidado explícito contra fixação de sessão, roubo de cookie, CSRF, CORS aberto e
headers HTTP fracos.

O ADR-0017 já definiu o baseline: sessão opaca server-side, CSRF explícito,
Argon2id, ASVS como referência e RLS como segunda barreira.

## Decisão

### Senhas e credenciais

- login primário usa e-mail verificado e senha;
- senha mínima da V1: 15 caracteres para todas as contas;
- senhas aceitam frases longas, espaços e gerenciadores de senha;
- não haverá regra obrigatória de maiúscula, número e símbolo;
- não haverá troca periódica arbitrária de senha;
- troca será exigida em caso de suspeita, vazamento, reset ou mudança relevante de
  política;
- senha será armazenada com Argon2id;
- parâmetros de Argon2id partem do mínimo recomendado pela OWASP e serão
  calibrados no ambiente real para manter verificação abaixo de 1 segundo;
- hashes antigos serão reprocessados no próximo login quando o work factor mudar;
- senhas comuns, fracas ou conhecidamente comprometidas serão bloqueadas por
  lista local ou serviço documentado que preserve privacidade.

### MFA

- admin master não acessa painel técnico sem MFA ativo;
- maestros/admins podem habilitar MFA e serão incentivados a fazê-lo;
- TOTP será o primeiro método suportado;
- códigos de recuperação são gerados uma única vez, exibidos ao usuário e
  armazenados somente como hash;
- perda de MFA do admin master exige procedimento técnico documentado, nunca
  bypass silencioso;
- passkeys/WebAuthn ficam planejados como evolução, não dependência da V1.

### Sessão

- a sessão web principal será opaca, server-side e revogável;
- o cookie contém somente token aleatório forte, sem claims, papel, tenant ou e-mail;
- o banco armazena apenas hash/HMAC do token de sessão, não o token em claro;
- o cookie de sessão usará prefixo `__Host-`, `HttpOnly`, `Secure`, `SameSite=Lax`,
  `Path=/` e nenhum atributo `Domain`;
- sessão é criada somente após senha e MFA exigido serem concluídos;
- desafio pré-MFA usa estado temporário curto e não concede acesso autenticado;
- ID de sessão gira após login, conclusão de MFA, elevação de privilégio,
  recuperação de senha e eventos sensíveis;
- usuário pode visualizar e revogar sessões ativas;
- alteração de senha revoga outras sessões por padrão;
- desativação de conta ou perfil revoga sessões afetadas imediatamente.

### Tempos iniciais

| Contexto | Ociosidade | Expiração absoluta | Observação |
|---|---:|---:|---|
| Músico, líder e maestro/admin em uso comum | 14 dias | 30 dias | favorece uso mobile recorrente |
| Janela de ação sensível | 15 minutos | Não aplicável | exige reautenticação recente |
| Admin master | 2 horas | 12 horas | sem sessão persistente longa |
| Impersonação | 30 minutos | 30 minutos | sempre com motivo, banner e encerramento explícito |
| Desafio pré-MFA ou recuperação | 10 minutos | 10 minutos | não concede sessão completa |

Esses valores são defaults da V1 e podem ser revisados por ADR antes da produção,
com base em teste de uso real.

### CSRF

- toda requisição mutável autenticada por cookie exige token CSRF;
- métodos mutáveis incluem `POST`, `PUT`, `PATCH` e `DELETE`;
- `GET`, `HEAD` e `OPTIONS` não podem alterar estado;
- o token segue padrão sincronizado server-side, vinculado à sessão;
- o token é entregue por bootstrap autenticado ou endpoint seguro, nunca por URL;
- o frontend envia o token em header customizado, como `X-CSRF-Token`;
- falha de token, token ausente ou token de outra sessão resulta em `403`;
- `Origin` deve bater com a origem esperada nas requisições mutáveis;
- `Referer` é fallback quando `Origin` estiver ausente e for confiável;
- Fetch Metadata será usado para rejeitar mutações cross-site quando disponível;
- `SameSite=Lax` é defesa adicional, não substituto do token CSRF;
- login, logout, recuperação de senha, aceite de convite e emissão de credencial
  de upload também seguem a política de CSRF quando houver sessão/cookie aplicável.

### CORS

- a API de negócio é same-origin por padrão;
- CORS fica desabilitado para credenciais fora das origens explicitamente
  permitidas;
- nunca usar `Access-Control-Allow-Origin: *` com credenciais;
- ambientes de desenvolvimento podem ter allowlist explícita e separada;
- qualquer futura origem pública exige ADR ou decisão operacional documentada.

### Headers HTTP

A V1 adotará, no mínimo:

| Header | Decisão inicial |
|---|---|
| `Strict-Transport-Security` | produção com HTTPS estável; evoluir para `includeSubDomains` e `preload` após validação do domínio |
| `Content-Security-Policy` | obrigatório; usar nonces/hashes para scripts e `frame-ancestors 'none'` |
| `X-Frame-Options` | `DENY` como fallback para clickjacking |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | negar recursos não usados, como câmera, microfone, geolocalização e pagamento |
| `Cache-Control` | `no-store` para respostas autenticadas sensíveis; assets versionados usam cache longo |

## Consequências positivas

- sessões podem ser revogadas e auditadas por dispositivo;
- cookie `HttpOnly` reduz exposição direta a JavaScript em caso de XSS;
- prefixo `__Host-` reduz risco de escopo amplo ou subdomínio injetar cookie;
- CSRF fica coberto por camadas complementares;
- senha forte por frase fica mais usável que regra de composição artificial;
- MFA protege a conta técnica mais poderosa desde o início.

## Custos e cuidados

- sessão server-side exige tabela, limpeza e observabilidade;
- CSRF cria contrato obrigatório entre frontend e API;
- CSP com Next.js exige disciplina de scripts, estilos e integrações;
- timeouts podem precisar ajuste após teste real com músicos;
- recuperação de MFA do admin master precisa ser desenhada antes de produção;
- listas de senhas comprometidas precisam equilibrar privacidade e cobertura.

## Alternativas rejeitadas

- JWT em `localStorage`: aumenta exposição em XSS e complica revogação imediata;
- cookie sem CSRF explícito: deixaria mutações expostas a cross-site requests;
- `SameSite` como única defesa: defesa útil, mas não suficiente sozinha;
- senha curta com regra de composição: pior experiência e falsa sensação de força;
- MFA obrigatório para todos na V1: seguro, mas atrito alto para músicos no começo;
- CORS aberto para facilitar desenvolvimento: perigoso e difícil de auditar depois.

## Referências

- https://owasp.org/www-project-application-security-verification-standard/
- https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html
- https://pages.nist.gov/800-63-4/sp800-63b.html
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie