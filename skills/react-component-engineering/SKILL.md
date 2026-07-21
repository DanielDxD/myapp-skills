---
name: react-component-engineering
description: Engenharia de componentes React reutilizáveis com composição, estado, hooks, acessibilidade, responsividade e APIs sustentáveis. Use ao criar ou refatorar design systems, UI kits, hooks compartilhados ou componentes de biblioteca.
---

# React Component Engineering

Foque em APIs de componentes claras e estáveis. Para arquitetura de SPA completa, use a skill `reactjs-application-development`.

## Princípios

- Composição > configuração excessiva.
- Um componente, um trabalho. Extraia variantes via composição ou slots.
- Props tipadas e previsíveis; evite props booleanas explosivas.
- Estado local o mais perto possível do uso; eleve só quando compartilhado.
- Acessibilidade não é opcional.

## API de props

- Prefira `children` / slots a render-prop labirínticas, salvo quando necessário.
- Use discriminated unions para variantes mutuamente exclusivas.
- Forward refs quando o consumidor precisar do DOM.
- Exponha `className` / `style` de forma controlada, alinhada ao sistema do projeto.
- Documente comportamento de estados: loading, empty, error, disabled.

## Estado e hooks

- Derive estado quando possível; não duplique fonte da verdade.
- Encapsule side effects em hooks com cleanup.
- Não esconda data fetching complexo dentro de componentes presentacionais.
- Evite `useEffect` para sincronizar estado que poderia ser calculado.

## Acessibilidade e UX

- Controles nativos primeiro (`button`, `input`, `dialog`), com ARIA só quando preciso.
- Foco visível, ordem de tab, labels, `aria-*` corretos.
- Teclado: Enter/Space/Escape/setas conforme o padrão do widget.
- Responsividade: layout fluido, touch targets, sem overflow escondido crítico.
- Respeite `prefers-reduced-motion` em animações do componente.

## Performance do componente

- Memoize só com evidência (listas grandes, renders caros).
- Estáveis: callbacks e objetos passados a filhos memoizados.
- Evite re-render de árvores grandes por estado irrelevante no pai.
- Listas: keys estáveis; virtualize quando necessário.

## Anti-padrões

- God components com dezenas de props
- Context global para tudo
- CSS acoplado a estrutura profunda frágil
- Side effects em render
- Ignorar estados de erro/loading

## Critérios de conclusão

- API tipada, mínima e documentada pelo uso
- Acessível por teclado e leitores de tela
- Responsivo nos breakpoints do projeto
- Sem efeitos colaterais ocultos
- Testável via Testing Library pelos comportamentos públicos
