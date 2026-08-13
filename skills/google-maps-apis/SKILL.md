---
name: google-maps-apis
description: Integra Google Maps Platform via HTTP — Directions, Distance Matrix, Geocoding, Places, Roads — com API keys, billing, quotas, restrições e contratos tipados. Use ao chamar APIs Google no backend ou client, estimar rotas/ETAs, autocomplete de endereços ou geocoding.
---

# Google Maps APIs

APIs HTTP da Google Maps Platform para dados de mapa/lugar/rota. Renderização nativa no app: `google-maps-sdk`. Trajetos no RN: `react-native-routes`. Geocode reverso: `reverse-geocoding`.

## Produtos comuns

| API | Uso |
|-----|-----|
| **Directions** | Rota A→B, steps, polyline, duration/distance |
| **Distance Matrix** | Matriz de ETAs (matching, batch) |
| **Geocoding** | Endereço ↔ coordenada |
| **Places** | Autocomplete, Place Details, nearby |
| **Roads** | Snap to road (trail de GPS) |

Use só as APIs necessárias — billing é por request.

## Regras inegociáveis

- API keys com **restrição** (HTTP referrer, IP server, bundle iOS/Android).
- Key de servidor **nunca** no app mobile; proxy via backend quando o produto exigir segredo.
- Ative apenas APIs usadas no Google Cloud Console; monitore quotas/alertas.
- Trate `OVER_QUERY_LIMIT`, `ZERO_RESULTS`, `NOT_FOUND`, `REQUEST_DENIED` de forma tipada.
- Cache respostas estáveis (geocode de endereço fixo) com TTL consciente e respeito aos ToS.
- Não logue keys nem PII de endereço em excesso.

## Auth e billing

- Projeto GCP + billing account.
- Separar keys: `mobile-sdk`, `server-directions`, `server-places` quando possível.
- Budget alerts no Cloud.
- Em white-label (`taxi-machine`): key/config por tenant ou projeto GCP dedicado.

## Contratos

- Tipar responses no client/server (lat, lng, `encodedPolyline`, `placeId`, `formattedAddress`).
- Prefira `placeId` a endereços soltos quando o usuário escolheu um Place.
- Language/region params para locale do usuário/tenant.
- Units: metric/imperial conforme mercado.

## Directions (resumo)

```text
origin, destination, mode=driving|walking|transit
waypoints? alternatives?
→ routes[].overview_polyline, legs[].duration, distance
```

- Decode polyline no client (`react-native-routes`) ou envie pontos já decodificados se preferir.
- Re-request em desvio significativo / modo muda.

## Places Autocomplete

- Session tokens para faturamento correto de autocomplete → details.
- `locationBias` / restriction pelo município do tenant.
- Debounce input (200–300 ms); mínimo de caracteres.

## Anti-padrões

- Key `AIza...` sem restrição commitada no git
- Autocomplete a cada keypress sem debounce/session
- Distance Matrix N×N explosivo no client
- Confiar em ZERO_RESULTS como erro fatal de rede
- Misturar Geocoding e Places Details sem necessidade (custo)

## Critérios de conclusão

- Keys restritas e APIs certas habilitadas
- Erros Google mapeados para UX/domain errors
- Session tokens em Places quando aplicável
- Directions/Matrix/Geocode cobrindo o fluxo do produto
- Sem key secreta no binário mobile
