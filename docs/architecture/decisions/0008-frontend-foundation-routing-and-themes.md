# ADR-0008 — Fundação frontend, rotas e temas

- Estado: Aceito
- Data: 2026-07-02

## Contexto

O frontend precisa servir músicos em celulares, administradores em desktop e um
showcase de componentes. A identidade é única do Concentus, mas a URL precisa
identificar a orquestra antes do login. Modos claro e escuro devem funcionar sem
permitir temas arbitrários por tenant.

## Decisão

### Rotas

- `/` apresenta login global Concentus;
- `/{orchestra_slug}` resolve login contextual ou área autenticada do tenant;
- slugs técnicos são reservados e não podem ser atribuídos a orquestras;
- antes da autenticação, o resolver expõe somente contexto institucional seguro;
- tenant inválido ou inativo produz resposta genérica.

### Design system

- Tailwind CSS implementa tokens e estilos;
- shadcn/ui fornece código inicial para componentes, que passa a ser mantido pelo
  Concentus;
- Storybook em `apps/showcase` documenta e testa componentes;
- tokens vivem em pacote compartilhado;
- a orquestra não substitui tokens nem injeta CSS.

### Tema

- valores permitidos: `system`, `light` e `dark`;
- padrão: `system`;
- preferência autenticada pertence à conta global;
- `system` acompanha o sistema de cada dispositivo via `prefers-color-scheme`;
- `light` e `dark` explícitos sincronizam entre dispositivos;
- estado local/cookie evita flash antes da hidratação;
- troca de tema não exige permissão do sistema operacional.

### Mídia institucional

- suporte é planejado, mas slots não são definidos neste ADR;
- imagens só poderão ocupar posições previstas pelo layout;
- nenhum upload institucional concede controle de CSS ou identidade visual.

## Consequências positivas

- URL contextual funciona antes e depois do login;
- identidade visual e acessibilidade permanecem consistentes;
- componentes podem ser desenvolvidos e avaliados isoladamente;
- preferência explícita acompanha o usuário, enquanto `system` respeita cada
  dispositivo;
- temas claro e escuro são testados pelo mesmo catálogo.

## Custos e cuidados

- slugs exigem lista de nomes reservados;
- renderização inicial precisa evitar flash entre temas;
- código importado do shadcn passa a ser responsabilidade do projeto;
- Storybook e aplicação devem consumir exatamente os mesmos tokens;
- imagens institucionais permanecem bloqueadas até existir especificação de slots.

## Alternativas rejeitadas

- subdomínios por tenant na V1: exigiriam DNS e certificados wildcard;
- preferência apenas local: não preservaria escolha explícita entre dispositivos;
- tema configurado pela orquestra: contraria o ADR-0007;
- biblioteca visual fechada: dificultaria identidade própria e manutenção fina.

