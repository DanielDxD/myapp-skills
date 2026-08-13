---
name: react-native-realtime-location
description: Localização em tempo real em foreground no React Native/Expo — watchPosition, precisão, throttling, mapa ao vivo e broadcast para backend. Use ao mostrar posição atual do usuário/motorista na UI, acompanhar movimento com o app aberto ou transmitir location via websocket.
---

# React Native Real-time Location

Stream de posição com o app em **foreground** (ou active). Atualiza mapa, ETA local e envia ticks ao backend. Background contínuo: `react-native-background-location`. Coordenadas base: `geolocation`. Mapa: `google-maps-sdk`.

## Stack

- `expo-location` `watchPositionAsync` / APIs equivalentes do projeto
- Map camera follow opcional (`google-maps-sdk`)
- Canal realtime (WebSocket / SSE / MQTT) para broadcast

## Regras inegociáveis

- Permissão When-In-Use antes de watch.
- Throttle/debounce de emissão ao backend (ex.: 1–5 s ou a cada X metros).
- Pare o watch no unmount / blur de tela / fim do fluxo (`remove()`).
- Trate GPS fraco: accuracy ruim → não mover câmera de forma errática.
- UI: indicador de “buscando GPS” / precisão baixa.

## Padrão de implementação

```text
permission → getCurrentPosition (seed)
          → watchPositionAsync(opts, onUpdate)
onUpdate → filter accuracy → update local state → maybe send socket
cleanup → subscription.remove()
```

Opções típicas:

- `accuracy`: High / Balanced conforme necessidade
- `distanceInterval` / `timeInterval` para reduzir churn
- `mayShowUserSettingsDialog` (Android) quando aplicável

## Mapa ao vivo

- Seed com última posição conhecida (cache curto) para evitar mapa vazio.
- Smooth camera: anime só se deslocamento &gt; limiar; evite jitter.
- Heading/bearing quando disponível (ícone de seta do veículo).
- Não reconstrua `MapView` a cada tick — atualize marker/polyline.

## Broadcast

- Payload mínimo: `{ lat, lng, accuracy, heading?, speed?, ts }`.
- Auth no socket; room por `tripId`.
- Sequence number ou timestamp para descartar out-of-order.
- Se socket cair: buffer curto + flush; não acumule infinito.

## Anti-padrões

- Watch global no root sem cleanup
- Enviar cada callback nativo sem filtro
- Confundir realtime foreground com Always background
- Travamento de UI por setState excessivo (batch / ref + rAF)

## Critérios de conclusão

- Watch inicia/para com o ciclo da tela/fluxo
- Mapa estável sem jitter grave
- Backend recebe ticks throttled e autenticados
- Permissão negada e GPS off tratados na UI
- Sem subscriptions órfãs após navegar para longe
