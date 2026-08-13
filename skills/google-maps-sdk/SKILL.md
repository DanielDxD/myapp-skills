---
name: google-maps-sdk
description: Integra Google Maps SDK no app mobile via react-native-maps ou Expo Maps — MapView, markers, polylines, câmera, styling e API keys iOS/Android. Use ao renderizar mapas nativos, pins, rotas desenhadas ou câmera seguindo o usuário.
---

# Google Maps SDK

SDK nativo no device para **mostrar** mapa e overlays. Dados de rota/lugar vêm de `google-maps-apis`. Em RN/Expo, o wrapper usual é `react-native-maps` (provider Google) ou o módulo Maps do Expo adotado pelo projeto.

## Setup

- Ative **Maps SDK for iOS** e **Maps SDK for Android** no GCP.
- Keys com restrição de bundle ID / SHA-1.
- `AppDelegate` / `AndroidManifest` / config plugin Expo conforme o template do projeto.
- Provider Google no Android; no iOS Apple Maps é default do `react-native-maps` — force Google se o produto exigir paridade.

## Regras inegociáveis

- Um `MapView` estável; não remonte a cada tick de GPS.
- Overlays (markers, polylines) com `key` estáveis.
- Loading / fallback se key inválida ou Play Services ausente.
- Dark mode: map style JSON ou estilo do sistema — alinhar a `better-mobile-interface-react-native`.
- Acessibilidade: descrições em pins críticos; não dependa só da cor.

## Capacidades

| Feature | Uso |
|---------|-----|
| `MapView` | Base |
| `Marker` | Origem, destino, motorista, POIs |
| `Polyline` | Trajeto (`react-native-routes`) |
| Camera / `animateToRegion` | Fit bounds, follow user |
| Circles / polygons | Hotspots, geofence visual |
| User location blue dot | Quando permissão ok |

## Câmera e performance

- `fitToCoordinates` com padding para origem+destino+veículo.
- Throttle follow-user (`react-native-realtime-location`).
- Evite milhares de markers sem clustering.
- Imagens de marker leves; cache.

## Estilo

- JSON styling Google para light/dark ou marca white-label (`taxi-machine`).
- Controles: compass, my-location, toolbar — ligue só o necessário.
- Gestos: rotate/pitch se fizer sentido para navegação.

## Anti-padrões

- Recriar MapView no render do parent state
- Key de Maps SDK sem restrição de app
- Polylines gigantes sem simplificação
- Assumir Google provider no iOS sem configurar
- Ignorar erro de billing/SDK nos devices de QA

## Critérios de conclusão

- Mapa renderiza iOS e Android com keys corretas
- Markers/polyline do fluxo principal ok
- Câmera/fit bounds adequados
- Estilo light/dark alinhado ao app
- Sem remount churn sob location updates
