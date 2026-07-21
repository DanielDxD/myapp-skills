---
name: web-performance
description: Otimiza Core Web Vitals, bundles, imagens, fontes, rendering e caching com metas verificáveis. Use ao investigar LCP/INP/CLS, reduzir JS, melhorar Lighthouse ou perfilar renders lentos em apps web.
---

# Web Performance

Meça antes de otimizar. Meta: experiência rápida em dispositivo mediano, não só no desktop potente.

## Métricas

- **LCP** — maior conteúdo visível rápido
- **INP** — interação responsiva
- **CLS** — layout estável
- Secundárias: TTFB, bundle size, hydration cost

Defina baseline (Lab + Field se disponível) e meta numérica.

## Rendering e JS

- Reduza JS no caminho crítico; code-split rotas e features pesadas.
- Server-first quando a stack permitir (RSC); hidrate só islands necessárias.
- Evite waterfalls de data fetching.
- Debounce/throttle handlers caros; quebre long tasks.
- Remova dependências pesadas substituíveis (moment → date-fns/native, lodash full → imports pontuais).

## Imagens e fontes

- Imagens: tamanho correto, formatos modernos, lazy abaixo da dobra, dimensões explícitas.
- Hero/LCP image: prioridade alta, sem lazy.
- Fontes: subset, `display=swap`/`optional`, preload só do critical face.
- Evite layout shift de fonte e mídia.

## Caching e rede

- Cache HTTP/CDN com política clara para estáticos.
- Prefetch/preload com parcimônia.
- Comprima assets; evite payloads JSON inflados.
- Streaming/Suspense para feedback cedo.

## Profiling

1. Reproduza o problema (rota, device throttle).
2. Use Lighthouse, Performance panel, React Profiler, bundle analyzer.
3. Aplique a menor mudança de maior impacto.
4. Re-meça e registre o delta.

## Anti-padrões

- Otimizar microbenchmarks irrelevantes
- `loading="lazy"` na LCP image
- Animar layout thrashing
- Carregar analytics/chat no critical path sem consent/necessidade
- Ignorar mobile

## Critérios de conclusão

- Baseline e pós-otimização medidos
- Regressões de CLS/LCP/INP evitadas ou justificadas
- Bundle do caminho crítico menor ou justificado
- Imagens/fontes sem shift óbvio
- Mudanças não quebram funcionalidade
