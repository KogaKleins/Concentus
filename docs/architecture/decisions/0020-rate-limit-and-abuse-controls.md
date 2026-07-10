# ADR-0020 — Rate limit e controle de abuso

- Estado: Aceito
- Data: 2026-07-09

## Contexto

O Concentus terá superfícies que podem ser abusadas mesmo sem uma falha clássica
de autorização: login, MFA, recuperação de senha, convites, uploads, downloads,
buscas, comentários, SSE, jobs assíncronos e envio de e-mails.

Esses abusos podem gerar tomada de conta, enumeração de usuários, negação de
serviço, aumento de custo operacional, spam interno, esgotamento de storage e
sobrecarga de workers. Como a plataforma começa pequena, não faz sentido adicionar
Redis apenas para rate limiting na V1; ao mesmo tempo, ignorar o tema agora criaria
uma dívida perigosa.

## Decisão

### Estratégia em camadas

A V1 adotará controles de abuso em camadas:

1. limites grossos no proxy/edge por IP, tamanho de requisição e conexão;
2. limites aplicacionais por ação, conta, e-mail normalizado, sessão, tenant,
   recurso e IP/prefixo;
3. limites de concorrência para uploads, processamento e SSE;
4. quotas de consumo para recursos caros, como e-mail, storage, download e jobs;
5. observabilidade de eventos suspeitos sem expor dados sensíveis.

Nenhum limite isolado será tratado como defesa suficiente. IP, por exemplo, é um
sinal útil, mas não é identidade confiável nem deve ser a única chave de bloqueio.

### Armazenamento dos contadores

- Na V1, os contadores aplicacionais persistentes ficarão no PostgreSQL.
- Não adicionaremos Redis na V1 apenas para rate limit.
- O proxy/edge poderá ter limites próprios em memória ou configuração local.
- Se houver múltiplas instâncias e o PostgreSQL virar gargalo, Redis ou serviço
  dedicado de rate limit exigirá novo ADR.
- Chaves de abuso devem minimizar dados pessoais: e-mail normalizado vira hash;
  IP pode ser armazenado como hash ou prefixo quando a precisão completa não for
  necessária.

### Tipos de limitador

- Token bucket ou GCRA para limites de rajada e uso normal.
- Janela deslizante para falhas de autenticação e abuso suspeito.
- Cooldown progressivo para login, MFA e recuperação de senha.
- Semáforo de concorrência para uploads, SSE, downloads longos e workers.
- Quota diária/mensal para recursos de custo acumulado.

### Autenticação

- Login sempre retorna mensagem genérica para falha.
- Falhas são limitadas por combinação de e-mail normalizado, conta quando
  conhecida, IP/prefixo e sessão/dispositivo quando aplicável.
- Não haverá bloqueio permanente automático de conta por erro de senha.
- Após falhas sucessivas, o sistema aplica atraso progressivo e cooldown curto.
- O teto duro de falhas consecutivas por autenticador segue o limite superior do
  NIST: nunca permitir mais de 100 tentativas consecutivas sem revalidação,
  recuperação ou intervenção segura.
- Sucesso de autenticação zera contadores de falha daquele autenticador quando
  apropriado.

### MFA

- Desafio MFA tem limite próprio de tentativas.
- Após tentativas excessivas, o desafio é invalidado e novo login completo pode
  ser exigido.
- Falha de senha correta seguida de falha MFA é evento sensível e pode gerar aviso
  ao usuário, com cuidado para não criar enxurrada de notificações.

### Recuperação de senha e convites

- Pedido de recuperação sempre responde de forma uniforme, exista ou não a conta.
- Envio de e-mail de recuperação é limitado por e-mail, IP/prefixo e janela de
  tempo.
- Convites e reenvios são limitados por orquestra, e-mail e maestro/admin.
- Consumo de convite é limitado por token, IP/prefixo e e-mail convidado.
- Tokens continuam sendo de uso único, armazenados como hash e sem exposição em
  logs.

### APIs autenticadas

- Toda rota autenticada passa por limite geral por conta/sessão.
- Rotas mutáveis possuem limite mais restrito que leituras.
- Listagens e buscas têm limite de página, ordenação permitida e tamanho máximo
  de filtros.
- Endpoints administrativos caros têm limites próprios, ainda que o usuário seja
  maestro/admin.
- O cliente não controla livremente tamanho de página, profundidade de resposta ou
  quantidade de operações em lote.

### Downloads

- Download só é considerado depois da autorização do material.
- Acesso a arquivo possui limite por conta, tenant, material e IP/prefixo.
- O objetivo é bloquear scraping e vazamento em massa, não vigiar rotina normal do
  músico.
- Como já decidido, a V1 não mostra ao maestro relatório de quem baixou; eventos
  permanecem técnicos, restritos e temporários.

### Uploads e processamento

- Upload usa limite por arquivo, lote, conta, tenant e concorrência.
- O usuário pode navegar na plataforma durante upload em lote, mas o upload ainda
  consome uma vaga de concorrência.
- Processamento assíncrono terá limite de jobs simultâneos por tenant e por tipo
  de tarefa.
- Limites finais de tamanho, tipos de arquivo e antimalware são definidos no
  ADR-0021; CDR permanece como evolução possível.

### Comunicados e interações

- Comentários, votos, curtidas/descurtidas e criação de comunicados possuem
  limites por conta, comunicado, tenant e janela de tempo.
- Líderes e maestros podem ter limites maiores, mas não ilimitados.
- Moderação continua sendo regra de produto; rate limit é apenas barreira contra
  spam e abuso automatizado.

### SSE e notificações

- SSE possui limite de conexões por conta, tenant e sessão/dispositivo.
- O cliente deve aplicar backoff ao reconectar.
- Notificações são deduplicadas e agrupadas antes de persistir e enviar via SSE.
- Fan-out grande deve passar por worker e quota de jobs, não por loop síncrono da
  requisição.

### Respostas HTTP

- Para rotas autenticadas comuns, usar `429 Too Many Requests` e `Retry-After`.
- Para login, recuperação de senha e convites, preservar resposta genérica e
  tempo uniforme sempre que revelar o limite possa ajudar enumeração.
- Respostas `429` não devem ser armazenadas por cache.
- Headers detalhados de limite, como limite restante, não serão expostos na V1 em
  endpoints sensíveis.

### CAPTCHA e fingerprinting

- CAPTCHA não será obrigatório por padrão na V1.
- CAPTCHA ou desafio adaptativo poderá ser usado futuramente para tráfego suspeito,
  com cuidado de acessibilidade.
- Fingerprinting invasivo de dispositivo não será adotado na V1.
- Sinais passivos básicos, como IP/prefixo, User-Agent e padrão de requisição,
  podem alimentar detecção de abuso com retenção limitada.

### Configuração e governança

- Limites serão configuráveis por ambiente e operação, mas não hardcoded.
- Maestro/admin da orquestra não poderá desligar proteções globais de segurança.
- Admin master poderá ajustar limites globais mediante registro operacional.
- Mudanças relevantes de limite antes de produção devem ser registradas em ADR ou
  decisão operacional.

## Consequências positivas

- Reduz risco de brute force, password spraying e credential stuffing.
- Diminui abuso de e-mail, uploads, downloads e jobs.
- Evita adicionar Redis prematuramente.
- Mantém proteção mensurável e testável desde o desenho da arquitetura.
- Preserva a experiência normal do músico ao evitar bloqueios permanentes
  agressivos.

## Custos e cuidados

- PostgreSQL receberá escritas extras de contadores de abuso.
- Limites mal calibrados podem gerar falso positivo em ensaios, eventos ou uploads
  grandes.
- Limites precisam ser observados em produção antes de ficarem rígidos demais.
- Algumas respostas genéricas dificultam suporte, exigindo boa auditoria técnica.
- Futuro scale-out pode exigir Redis, serviço gerenciado ou proteção adicional na
  borda.

## Alternativas rejeitadas

- Sem rate limit na V1: risco alto demais para autenticação, e-mail e arquivos.
- Bloqueio permanente de conta por falhas: fácil de abusar como negação de serviço
  contra usuários legítimos.
- Rate limit apenas por IP: frágil contra proxies, NAT, redes compartilhadas e
  ataques distribuídos.
- CAPTCHA obrigatório em todo login: atrito alto, impacto de acessibilidade e
  proteção incompleta.
- Redis obrigatório desde o início: adiciona operação antes de haver escala que
  justifique.
- Mostrar ao maestro relatório de downloads: não é necessário na V1 e mudaria a
  natureza de privacidade já acordada.

## Testes obrigatórios

Antes de produção, testes devem provar:

1. login falho aplica cooldown sem revelar se e-mail existe;
2. login bem-sucedido limpa contadores aplicáveis;
3. MFA invalida desafio após excesso de tentativas;
4. recuperação de senha responde uniformemente para conta existente e inexistente;
5. envio excessivo de e-mail é bloqueado ou atrasado;
6. API autenticada comum retorna `429` com `Retry-After` quando apropriado;
7. rota sensível não expõe headers que facilitem enumeração;
8. upload em lote respeita concorrência;
9. download em massa é limitado sem criar relatório comportamental para maestro;
10. SSE exige backoff e respeita limite de conexões;
11. job fan-out respeita quota por tenant;
12. logs de abuso não gravam tokens, senhas, URLs assinadas ou conteúdo sensível.

## Referências

- https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/
- https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Credential_Stuffing_Prevention_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Denial_of_Service_Cheat_Sheet.html
- https://pages.nist.gov/800-63-4/sp800-63b.html
- https://www.rfc-editor.org/rfc/rfc6585.html#section-4
- https://www.rfc-editor.org/rfc/rfc9110.html#name-retry-after
