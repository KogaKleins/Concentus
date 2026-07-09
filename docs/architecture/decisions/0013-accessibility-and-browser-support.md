# ADR-0013 — Acessibilidade e suporte a navegadores

- Estado: Aceito
- Data: 2026-07-03

## Contexto

O Concentus será usado principalmente em celulares por músicos e em desktop por
maestros. Uma declaração vaga de que a interface será “acessível e responsiva”
não define critérios verificáveis nem estabelece quais ambientes precisam ser
testados.

## Decisão

### Acessibilidade

- o frontend tem como requisito formal WCAG 2.2 nível AA;
- nível AA é critério de projeto, revisão, teste e aceite desde os componentes;
- Storybook documenta estados acessíveis, incluindo teclado, foco, erro, loading,
  contraste, tema claro e tema escuro;
- testes automatizados ajudam a detectar regressões, mas não substituem avaliação
  manual com teclado, zoom e tecnologia assistiva;
- nenhuma informação depende somente de cor, hover, som ou movimento;
- o sistema respeita preferências como redução de movimento;
- o projeto não declara conformidade pública sem avaliação adequada do produto;
- a acessibilidade do visualizador e dos controles pertence ao Concentus; o
  conteúdo interno de PDFs e documentos enviados pertence aos seus autores e não
  pode ser automaticamente garantido pela plataforma.

### Navegadores suportados

- Chrome, Edge e Firefox: versão principal atual e anterior;
- Safari no macOS e iOS: versão principal atual e anterior;
- a matriz é móvel: a cada nova versão estável, a janela suportada avança;
- testes mobile prioritários usam Chrome no Android e Safari no iOS;
- PWA instalável é validada apenas onde o navegador oferecer os recursos
  necessários;
- navegadores fora da matriz podem funcionar, mas não recebem garantia completa;
- ausência de recurso essencial produz aviso claro ou bloqueio explicativo;
- detecção de capacidade é preferida a depender somente do nome do navegador.

## Consequências positivas

- acessibilidade possui meta objetiva e verificável;
- componentes já nascem com estados de teclado e leitor de tela;
- a equipe sabe onde reproduzir e priorizar defeitos de compatibilidade;
- a política acompanha navegadores evergreen sem congelar números de versão;
- limitações de documentos enviados não são confundidas com falhas da interface.

## Custos e cuidados

- testes manuais precisam fazer parte de cada fluxo crítico;
- Safari/iOS e Chrome/Android exigem dispositivos ou ambientes de teste reais;
- componentes de terceiros precisam ser auditados antes de adoção;
- PDFs inacessíveis podem exigir orientação futura aos autores;
- atingir critérios técnicos não elimina a necessidade de testes com usuários.

## Alternativas rejeitadas

- acessibilidade apenas após a V1: tornaria correções estruturais mais caras;
- somente testes automatizados: ferramentas não avaliam toda a experiência;
- suportar qualquer navegador indefinidamente: inviabilizaria teste e segurança;
- bloquear todo navegador fora da matriz: impediria uso que ainda pode funcionar;
- prometer acessibilidade do conteúdo de todo PDF enviado: não é controlável pela
  aplicação.
