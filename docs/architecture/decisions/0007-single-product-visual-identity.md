# ADR-0007 — Identidade visual única do produto

- Estado: Aceito
- Data: 2026-07-02
- Substitui: ADR-0001

## Contexto

O ADR-0001 permitia que cada orquestra tivesse cores e identidade visual próprias.
Essa direção faria o Concentus se comportar como plataforma white-label e
enfraqueceria consistência, acessibilidade e reconhecimento do produto.

A orquestra deve ser identificável, mas não redesenhar a aplicação.

## Decisão

- Concentus possui uma única identidade visual;
- o design system oferece modos claro e escuro próprios do produto;
- orquestras não alteram paleta, tipografia, espaçamento, sombras, ícones,
  componentes, navegação ou layouts;
- cada tenant pode exibir nome, símbolo e URL como contexto institucional;
- o produto pode oferecer slots controlados para capa, imagem de fundo ou mídia
  institucional;
- slots controlam dimensões, recorte, overlay, contraste, fallback e peso;
- tenant sem imagem usa o padrão Concentus;
- login contextual continua sendo uma tela Concentus, enriquecida com contexto da
  orquestra, e não uma aplicação rebranded.

## Consequências positivas

- experiência consistente entre orquestras;
- componentes e acessibilidade testados uma única vez;
- manutenção menor nos modos claro e escuro;
- identidade reconhecível do produto;
- screenshots, suporte e documentação permanecem coerentes.

## Custos e limites

- orquestras possuem menos liberdade estética;
- mídias institucionais exigem regras de contraste e processamento;
- temas precisam equilibrar personalidade própria e neutralidade para diferentes
  instituições.

## Migração da decisão anterior

Referências atuais a cores ou temas configuráveis por tenant devem ser removidas.
Nome, símbolo, slug e mídias institucionais controladas permanecem válidos.

