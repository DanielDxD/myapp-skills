---
name: react-native-routes
description: Trajetos no mapa em React Native — Directions API, decode de polyline, Polyline no MapView, rotas alternativas, ETA e re-roteamento. Use ao desenhar rota origem/destino, pickup do motorista ou navegação visual no app.
---

# React Native Routes

Rotas de **mapa** (trajeto), não rotas de tela. Navegação de screens: `react-native-expo-navigation`. Directions HTTP: `google-maps-apis`. Desenho: `google-maps-sdk`.

## Pipeline

```text
origin + destination (+ waypoints)
  → Directions API (mode, alternatives)
  → pick route (default / user choice)
  → decode overview_polyline
  → <Polyline coordinates={...} />
  → fitToCoordinates(padding)
  → show ETA / distance
```

## Regras inegociáveis

- Chamar Directions no **backend** quando a key for secreta ou o volume for alto; mobile só com key restrita se o projeto permitir.
- Decode correto (precision 5 Google Encoded Polyline Algorithm).
- Atualizar rota em desvio significativo ou mudança de destino — não a cada GPS tick.
- Estados: loading rota, erro, zero results, rota ativa.
- Alternates: listar duração/distância antes de trocar polyline.

## Modos

- `driving` — default transporte
- `walking` / `bicycling` / `transit` — se o produto oferecer
- Evite mode errado para ETA de carro

## Polyline no mapa

- Cor do token `accent` / traffic-agnostic; espessura legível (≈ 4–6 pt).
- Z-index acima do base map; abaixo de markers de veículo se necessário.
- Simplifique geometria se demais pontos (>1–2k) afetarem FPS.
- `fitToCoordinates` com padding para header/bottom sheet.

## ETA e re-roteamento

- Mostre `duration` / `duration_in_traffic` quando disponível e permitido.
- Re-route se distância à polyline &gt; limiar (ex.: 50–100 m) por N segundos.
- Throttle re-requests (mín. intervalo) para custo e estabilidade.
- Em viagem (`taxi-machine`): rota motorista→pax e depois pax→destino como duas fases.

## Waypoints e multi-stop

- Ordem otimizada só se a API/`optimize:true` e o produto permitirem.
- Limite de waypoints conforme cota Google.
- UI clara da ordem das paradas.

## Anti-padrões

- Redesenhar Directions a cada `watchPosition` callback
- Polyline hardcoded de demo em produção
- Ignorar `status` != OK
- Fit bounds sem padding (UI cobrindo a rota)
- Misturar coordenadas destino tipadas como string endereço sem geocode

## Critérios de conclusão

- Rota desenhada A→B com ETA/distance
- Loading/erro cobertos
- Decode e fit bounds corretos em iOS/Android
- Re-route com throttle no fluxo de desvio (se no escopo)
- Integração limpa com MapView sem remount
