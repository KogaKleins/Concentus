# Processo de decisões embasadas

Status: Aceito  
Última revisão: 2026-07-08

## 1. Princípio

Decisão estrutural não deve nascer escondida no código nem depender apenas de
memória, preferência ou pressa.

Ela deve ter:

- problema claro;
- contexto do Concentus;
- alternativas consideradas;
- embasamento proporcional ao risco;
- decisão aprovada;
- consequências e cuidados;
- local onde será mantida viva.

## 2. Níveis de embasamento

| Nível | Quando usar | Fontes esperadas |
|---|---|---|
| Leve | Decisão reversível e pequena | Experiência do projeto e coerência interna |
| Médio | Afeta arquitetura, dados ou operação | Documentação oficial, padrões conhecidos e comparação de alternativas |
| Forte | Segurança, privacidade, banco, autenticação, custos relevantes ou risco de perda de dados | Fontes primárias, documentação oficial, padrões de segurança e ADR obrigatório |

## 3. Fontes preferidas

Ordem de preferência:

1. documentação oficial da tecnologia;
2. padrões reconhecidos, como W3C, OWASP, IETF ou documentação de cloud/arquitetura;
3. artigos técnicos de autores ou organizações reconhecidas;
4. experiência local documentada;
5. opinião informal apenas como hipótese, nunca como prova final.

## 4. Quando criar ADR

Criar ADR quando a decisão afetar:

- stack ou runtime;
- segurança;
- autenticação e sessão;
- banco de dados, RLS ou migrações;
- propriedade de dados;
- filas, workers e efeitos assíncronos;
- fronteiras entre módulos;
- padrões de teste;
- compatibilidade, acessibilidade ou operação.

## 5. Segurança

Decisões de segurança exigem nível forte de embasamento. Devem responder:

1. que ameaça reduzimos;
2. que ameaça permanece;
3. que teste comprova a mitigação;
4. que log ou alerta permite perceber abuso;
5. que custo operacional aceitamos;
6. qual decisão será revisada antes de produção.

## 6. Regra de ouro

O assistente pode sugerir e pesquisar, mas sugestão não vira regra sem aprovação.
Decisão aceita pode mudar depois, mas não deve ser apagada para esconder a mudança:
novo ADR substitui o anterior.
