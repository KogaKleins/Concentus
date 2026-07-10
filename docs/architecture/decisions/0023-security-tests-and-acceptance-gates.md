# ADR-0023 — Testes de segurança e critérios de aceite

- Estado: Aceito
- Data: 2026-07-09

## Contexto

Os ADRs 0017 a 0022 definiram baseline de segurança, threat model, sessão,
CSRF, RLS, rate limit, upload seguro, antimalware, segredos, backup e resposta a
incidentes. Essas decisões só terão valor prático se forem verificáveis antes de
merge, release e produção.

O Concentus já adotou Vitest, PostgreSQL real, Playwright, Storybook e gates de
qualidade. Este ADR fecha como essas ferramentas serão usadas para provar os
controles de segurança e quais evidências bloqueiam ou liberam avanço.

## Decisão

### Regra central

Controle de segurança sem teste, evidência ou exceção formal não é considerado
fechado.

Um recurso sensível só está pronto quando:

1. possui regra documentada;
2. possui teste positivo;
3. possui teste negativo;
4. possui teste de fronteira quando aplicável;
5. gera log ou métrica segura quando falha;
6. aparece no gate correto de PR, release ou produção.

### Fontes de referência

- ASVS 5.0.0 será usado como catálogo de requisitos, sem certificação formal na V1.
- NIST SSDF será usado como referência de práticas de desenvolvimento seguro ao
  transformar controles em gates e evidências.
- OWASP WSTG será usado como guia para testes manuais e exploratórios.
- Cheat sheets de autorização da OWASP serão referência para testes de regressão
  e automação de autorização.
- ZAP baseline/API scan será adotado como varredura automatizada complementar.
- Testes próprios continuam obrigatórios; scanner não substitui teste de regra de
  negócio.

### Camadas obrigatórias

| Camada | Prova de segurança |
|---|---|
| Unitária | regra pura, precedência, normalização e bloqueios locais |
| Integração API | autenticação, autorização, CSRF, rate limit e respostas |
| Integração DB | RLS, constraints, papéis, transações e contexto de tenant |
| Worker | jobs com tenant, falha fechada, idempotência e efeitos assíncronos |
| E2E | jornadas reais por papel e navegador |
| DAST | headers, cookies, CSRF ausente, exposição acidental e API scan |
| Operacional | restore, rotação, incidente simulado e secret scanning |

### Gates

#### Pull request

Obrigatório:

- lint e tipos;
- unitários;
- cobertura mínima e ratchet;
- integração PostgreSQL real para áreas tocadas;
- testes de autorização quando regra de acesso for alterada;
- E2E crítico no Chromium quando fluxo sensível for alterado;
- secret scanning;
- ausência de `TODO` de segurança novo sem rastreio explícito.

#### Branch principal

Obrigatório:

- todos os gates de PR;
- integração completa de segurança;
- E2E crítico completo;
- matriz de navegadores definida no ADR-0014 quando aplicável;
- relatório de cobertura;
- artefatos de falha retidos conforme política de CI.

#### Release candidata

Obrigatório:

- todos os gates da branch principal;
- ZAP baseline scan;
- ZAP API scan quando houver OpenAPI disponível;
- checklist manual WSTG reduzido;
- restore testado ou evidência recente válida;
- verificação de headers/cookies/cache em build de produção;
- validação de configuração segura de ambiente;
- relatório de exceções conhecidas.

#### Produção

Obrigatório antes da primeira produção real:

- nenhum SEV-1/SEV-2 aberto;
- nenhum teste P0 pendente;
- threat model revisado;
- runbook mínimo de incidente pronto;
- backup automatizado e restore testado;
- inventário de segredos criado;
- scanner antimalware configurado ou risco formalmente aceito para ambiente não
  produtivo;
- logs técnicos sem vazamento de segredo em amostra verificada.

### Exceções

Exceção de segurança precisa conter:

- controle afetado;
- risco aceito;
- impacto provável;
- ambiente permitido;
- prazo de expiração;
- responsável;
- mitigação temporária;
- teste que será reativado ou criado.

Exceção sem prazo não é aceita. Exceção para produção envolvendo vazamento
cross-tenant, bypass de autorização, segredo exposto ou backup não restaurável
exige decisão explícita do admin master/desenvolvedor responsável.

### Evidências

Cada release candidata deve produzir um pacote mínimo de evidência:

- commit/tag avaliado;
- data;
- ambiente;
- resultado dos gates;
- resumo de falhas e exceções;
- cobertura;
- resultado de ZAP quando aplicável;
- resultado de restore ou referência ao ensaio recente;
- checklist manual assinado pelo responsável.

Esses artefatos ficam no histórico técnico da release, não em documentação de
produto voltada ao usuário final.

### Escopo crítico de segurança

O escopo crítico inclui:

- autenticação, MFA, sessão e cookies;
- CSRF, CORS e headers;
- tenant resolution, RLS e contexto transacional;
- autorização hierárquica, pesos e liderança;
- convites e recuperação de senha;
- impersonação e auditoria;
- rate limit e abuso;
- upload, antimalware e serving de arquivos;
- downloads e URLs assinadas;
- comunicados anônimos e privacidade;
- pg-boss e workers;
- segredos, logs seguros, backup e restore.

### Testes que devem falhar

Para controles de segurança, teste negativo é obrigatório. Exemplos:

- usuário da orquestra A tenta acessar UUID da B;
- líder tenta alterar decisão bloqueada pelo maestro;
- requisição mutável sem CSRF;
- cookie sem `HttpOnly` ou `Secure`;
- upload `.exe` renomeado para `.pdf`;
- job sem `orchestra_id`;
- token de convite reutilizado;
- download de material sem permissão;
- segredo de teste commitado;
- restore sem objeto correspondente ao metadado.

## Consequências positivas

- Segurança passa a ter evidência objetiva.
- Mudanças futuras terão regressão automatizada.
- Critérios de produção ficam claros antes de código.
- Scanners complementam, mas não substituem testes de negócio.
- O pacote de segurança da documentação fica encerrável.

## Custos e cuidados

- A suíte de testes crescerá e precisará de particionamento.
- Alguns gates, como ZAP e restore, não rodam em toda PR.
- Testes de autorização exigem factories e matrizes bem desenhadas.
- Falso positivo de scanner precisa triagem, não desativação silenciosa.
- Evidência de release precisa retenção e disciplina operacional.

## Alternativas rejeitadas

- Deixar segurança para revisão manual final: caro, tardio e pouco repetível.
- Depender apenas de scanner DAST: não entende regras de naipe, biblioteca,
  maestro, líder e tenant.
- Depender apenas de testes unitários: não prova RLS, headers, cookies, workers ou
  deploy.
- Aceitar exceção sem prazo: transforma risco temporário em dívida permanente.
- Exigir pentest externo obrigatório para a V1 privada: pode ser útil, mas não
  substitui gates internos; fica recomendado antes de abertura pública ampla.

## Testes obrigatórios

Antes da primeira produção real, deve haver cobertura automatizada ou evidência
manual rastreável para:

1. todos os itens P0 do threat model;
2. todos os testes obrigatórios dos ADRs 0018 a 0022;
3. secret scanning;
4. restore testado;
5. ZAP baseline;
6. ZAP API scan quando houver OpenAPI;
7. checklist manual reduzido WSTG;
8. autorização por matriz de papéis;
9. isolamento DB com RLS;
10. logs sem segredos em amostra.

## Referências

- https://owasp.org/www-project-application-security-verification-standard/
- https://csrc.nist.gov/pubs/sp/800/218/final
- https://owasp.org/www-project-web-security-testing-guide/
- https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Testing_Automation_Cheat_Sheet.html
- https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Regression_Testing_Cheat_Sheet.html
- https://www.zaproxy.org/docs/docker/baseline-scan/
- https://www.zaproxy.org/docs/docker/api-scan/
- https://playwright.dev/docs/test-assertions
- https://vitest.dev/guide/coverage
