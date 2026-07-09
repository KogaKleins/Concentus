# Design system e showcase

## 1. Identidade

O design system pertence ao Concentus. Tenant é contexto, não tema.

```text
Concentus
├── tema claro
├── tema escuro
├── tipografia
├── espaçamento
├── ícones
├── componentes
└── motion

Orquestra
├── nome
├── símbolo
└── mídias em slots permitidos
```

## 2. Modos claro e escuro

Preferência prevista:

- `system`: acompanha o sistema operacional;
- `light`: força modo claro;
- `dark`: força modo escuro.

A preferência pertence à conta global, não à orquestra:

- padrão inicial: `system`;
- `system`: cada dispositivo acompanha seu próprio sistema operacional;
- `light` ou `dark`: escolha explícita sincronizada entre dispositivos;
- antes da autenticação: usar preferência local conhecida ou `system`;
- após autenticação: reconciliar com a preferência da conta;
- cookie/local storage pode espelhar o valor para evitar flash visual, mas o banco
  mantém a preferência autenticada de referência.

`prefers-color-scheme` é uma media query do navegador e não solicita permissão ao
usuário. Ambos os modos precisam preservar contraste, foco, legibilidade de
partituras e estados sem depender somente de cor.

## 3. Tokens

Categorias mínimas:

- cores semânticas;
- tipografia;
- espaçamento;
- raios;
- sombras;
- camadas/z-index;
- motion;
- breakpoints;
- tamanhos de toque;
- largura de conteúdo.

Componentes usam nomes semânticos como `surface`, `foreground`, `primary`,
`danger` e `focus`, nunca uma cor física como contrato público.

## 4. Mídia institucional

Cada slot define:

- formatos aceitos;
- proporção e resolução;
- peso máximo;
- recorte responsivo;
- overlay obrigatório quando necessário;
- fallback Concentus;
- texto alternativo ou indicação decorativa;
- processamento e remoção segura.

Não existe CSS, fonte ou cor arbitrária fornecida pelo tenant.

## 5. Showcase

Storybook documentará:

- foundations e tokens;
- componentes primitivos;
- componentes compostos;
- padrões de formulário;
- feedback, loading e erros;
- navegação mobile e desktop;
- temas claro e escuro;
- slots com e sem mídia institucional;
- estados por papel sem implementar autorização no componente;
- testes de interação e acessibilidade.

## 6. Ferramentas aceitas

- Storybook mantém o showcase e os estados documentados;
- Tailwind CSS implementa utilitários e tokens;
- shadcn/ui fornece código inicial, não uma identidade pronta;
- componentes copiados passam a pertencer ao Concentus e só recebem atualizações
  upstream mediante revisão explícita;
- tokens ficam em pacote próprio compartilhado por `web` e `showcase`.

A decisão está registrada no
[ADR-0008](../decisions/0008-frontend-foundation-routing-and-themes.md).

## 7. Imagens institucionais pendentes

O mecanismo é previsto, mas nenhum slot será modelado antes do layout das páginas.
Cada slot futuro exigirá finalidade, responsável, dimensões, fallback, contraste,
limite de peso, comportamento responsivo e fluxo de moderação definidos.
