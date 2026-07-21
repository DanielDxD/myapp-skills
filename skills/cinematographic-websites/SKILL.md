---
name: cinematographic-websites
description: Projeta, implementa e revisa sites cinematográficos com direção de arte autoral, narrativa por scroll e movimento deliberado usando Lenis, GSAP e ScrollTrigger. Use sempre que o usuário pedir um site, landing page, portfólio, hero, seção ou experiência visual cinematográfica, editorial, imersiva ou film-like.
---

# Cinematographic Websites

Trate a página como uma sequência dirigida, não como um conjunto de efeitos. Cada movimento deve controlar atenção, revelar informação ou estabelecer atmosfera.

## Regras inegociáveis

- Use Lenis como única camada de smooth scroll.
- Use GSAP para todo movimento e ScrollTrigger para toda animação vinculada ao scroll.
- Não misture Framer Motion, bibliotecas de reveal, smooth scroll em CSS ou outro motor de animação.
- Entradas e saídas duram no mínimo `0.6s`. Prefira `0.8–1.4s` para reveals importantes.
- Use apenas eases da família `power3` ou `expo`. O padrão de entrada é `power3.out`.
- Em sequências editoriais, use `stagger: 0.2`.
- Movimento cinematográfico é lento com intenção, nunca lento por falta de resposta.
- Preserve a stack, os padrões e a arquitetura existentes. Instale somente dependências ausentes.
- Respeite `prefers-reduced-motion`; acessibilidade é a única exceção às regras de movimento.

## Antes de implementar

1. Inspecione a estrutura, a stack, os estilos e as animações existentes.
2. Defina em poucas linhas:
   - a ideia central e a emoção da experiência;
   - o papel narrativo de cada seção;
   - uma assinatura visual memorável;
   - o ritmo: tensão, revelação, sustentação e transição.
3. Escolha tipografia, escala, enquadramento, cor e movimento a partir do assunto real. Evite estética genérica de “site premium”.
4. Organize a página como cenas. Cada cena deve ter um ponto focal e uma razão para existir.
5. Só então implemente.

## Sistema de movimento

### Responsabilidades

- Lenis controla a sensação e a progressão do scroll.
- GSAP controla timelines, transforms, opacidade, máscaras e estados.
- ScrollTrigger conecta as timelines à posição do scroll, incluindo `pin`, `scrub` e callbacks.
- Registre plugins uma única vez e inicialize tudo apenas no cliente quando a stack tiver SSR.
- Sincronize Lenis com o ticker do GSAP e notifique `ScrollTrigger.update` a cada scroll.
- Recalcule triggers depois que fontes, mídia ou layout assíncrono alterarem dimensões.
- Faça cleanup de timelines, triggers, listeners e da instância Lenis no ciclo de vida apropriado.

### Ritmo

- Crie uma pausa perceptível antes do primeiro reveal importante. Tensão vem da expectativa, não de loaders artificiais.
- Use sobreposição entre animações para continuidade; evite uma fila mecânica de efeitos isolados.
- Entrada padrão: `power3.out`.
- Movimento amplo ou mudança de cena: `expo.inOut` ou `power3.inOut`.
- Saída: nunca abaixo de `0.6s`.
- Sequência de elementos: `0.2s` entre itens, salvo quando o conteúdo exigir leitura mais lenta.
- Em scrub, faça o deslocamento parecer massa e inércia de câmera, sem saltos.
- Não use bounce, elastic, back, eases extravagantes ou parallax em tudo.

### Propriedades

- Anime prioritariamente `transform`, `opacity`, `clip-path` e máscaras.
- Evite animar propriedades que recalculam layout, como `top`, `left`, `width` e `height`.
- Use profundidade com escala, translação, desfoque sutil e diferença de velocidade; não dependa apenas de fade-up.
- Aplique `will-change` apenas durante animações relevantes, não globalmente.
- Garanta que o estado final seja correto mesmo se JavaScript falhar ou a animação for desativada.

## Hero cinematográfico

“Cria um hero fullscreen com vídeo em loop no background, texto grande em reveal com GSAP e um scroll indicator que some ao descer. Tipografia tem que criar tensão antes da primeira palavra aparecer.”

Ao aplicar:

- Faça o hero ocupar a viewport real, considerando barras móveis.
- Use vídeo decorativo com `autoplay`, `muted`, `loop` e `playsinline`, além de poster e fallback visual.
- Preserve legibilidade com enquadramento e contraste, não com overlays genéricos pesados.
- Revele o título por linhas ou palavras usando máscaras, sem quebrar o texto semântico para leitores de tela.
- Segure o primeiro frame por tempo suficiente para criar tensão e então revele em uma timeline coordenada.
- Faça o scroll indicator desaparecer progressivamente após o usuário iniciar o scroll.
- Não reproduza áudio automaticamente.

## Pin scroll cinematográfico

“Cria uma seção com pin scroll usando ScrollTrigger. Elementos entram em sequência com timing de 0.2s entre cada um. Usa ease power3.out. O scroll tem que parecer controlado por câmera.”

Ao aplicar:

- Use uma timeline principal para a cena, com um único ScrollTrigger sempre que possível.
- Fixe apenas o tempo necessário para apresentar a narrativa; pin excessivo faz a página parecer travada.
- Dimensione a distância de scroll pelo número e pela duração dos beats narrativos.
- Use `scrub` suavizado para criar peso de câmera, sem remover o controle do usuário.
- Anime os elementos em sequência com `stagger: 0.2` e `ease: "power3.out"`.
- Mantenha continuidade visual entre entrada, sustentação e saída.
- Revise em mobile: reduza deslocamentos, adapte ou remova o pin quando a viewport não comportar a cena.

## Direção de arte

- O hero é a tese visual da experiência.
- Tipografia é protagonista: escolha famílias com personalidade, escala expressiva e contraste de papéis.
- Use conteúdo real ou específico ao assunto; lorem ipsum e slogans vagos destroem a imersão.
- Escolha um gesto visual dominante e mantenha o restante disciplinado.
- Use assimetria, espaço negativo, crop, ritmo editorial e sobreposição com propósito.
- Não transforme “cinematográfico” automaticamente em fundo preto, fonte serifada e grão.
- Não use cards, pills, gradientes ou glow como preenchimento visual sem relação com o conceito.
- Preserve navegação, hierarquia, foco visível e leitura confortável em todas as larguras.

## Responsividade e acessibilidade

- Em `prefers-reduced-motion: reduce`, desative smooth scroll, pin prolongado, scrub e grandes deslocamentos; mantenha o conteúdo visível e a navegação imediata.
- Nunca dependa apenas da animação para comunicar estado ou significado.
- Garanta teclado, foco visível, contraste e ordem de leitura coerente.
- Pause vídeo quando estiver fora da viewport ou quando a página ficar oculta, se a stack permitir.
- Ofereça poster/fallback para economia de dados, falha de autoplay e dispositivos limitados.
- Não aplique smooth scroll a controles internos, modais ou áreas que precisam de scroll nativo.

## Performance

- Comprima vídeos e imagens, forneça formatos e tamanhos adequados e evite preload agressivo de mídia não crítica.
- Evite múltiplos vídeos simultâneos, filtros pesados em grandes áreas e layers promovidas sem necessidade.
- Agrupe leituras e escritas de layout. Não consulte geometria a cada frame se ScrollTrigger pode calcular.
- Teste com CPU reduzida e em viewport móvel; desktop potente não é referência suficiente.
- Não sacrifique LCP, interação ou estabilidade visual por uma introdução.

## Fluxo de revisão

“Revisa todas as animações do projeto. Qualquer coisa que entrar ou sair em menos de 0.6s, aumenta. Qualquer ease que não seja power3 ou expo, substitui. Sites cinematográficos são lentos com intenção.”

Ao revisar:

1. Catalogue timelines, tweens, transições CSS e bibliotecas de movimento.
2. Remova motores concorrentes e concentre movimento em GSAP.
3. Migre comportamento de scroll para Lenis + ScrollTrigger.
4. Aumente toda entrada ou saída abaixo de `0.6s`.
5. Substitua qualquer ease fora de `power3` e `expo`.
6. Normalize sequências editoriais para `stagger: 0.2`.
7. Procure flashes de conteúdo, triggers duplicados, leaks e cleanup ausente.
8. Verifique reduced motion, teclado, mobile e estados sem JavaScript.
9. Revise o ritmo completo da página, não apenas cada tween isoladamente.
10. Valide visualmente o início, o meio e o fim de cada cena em diferentes viewports.

## Critérios de conclusão

- A experiência tem uma ideia visual específica, não um preset “cinematic”.
- Scroll e timelines estão sincronizados sem jitter.
- Nenhuma entrada ou saída dura menos de `0.6s`.
- Todos os eases pertencem a `power3` ou `expo`.
- Sequências usam intervalo de `0.2s` quando aplicável.
- Hero, pin e transições funcionam em desktop e mobile.
- Reduced motion apresenta todo o conteúdo sem bloqueios.
- Não há triggers órfãos, listeners duplicados ou animações após unmount.
- Vídeo tem poster/fallback e não compromete legibilidade ou carregamento.
- O resultado foi inspecionado em movimento; código correto sem revisão visual não basta.
