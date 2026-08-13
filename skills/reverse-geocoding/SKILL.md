---
name: reverse-geocoding
description: Converte coordenadas em endereço legível (reverse geocoding) com Google Geocoding, Expo Location ou APIs de plataforma, incluindo cache, locale e formatação. Use ao mostrar endereço a partir de um pin no mapa, pickup point ou última posição GPS.
---

# Reverse Geocoding

Entrada: `{ lat, lng }` → saída: endereço estruturado + `formatted` para UI. Complementa `geolocation`. HTTP Google: `google-maps-apis`. Não confundir com geocoding forward (endereço → coordenada) nem com Places Details.

## Providers

| Provider | Quando |
|----------|--------|
| **Google Geocoding API** | Precisão/consistência white-label; key server |
| **expo-location** `reverseGeocodeAsync` | Rápido no device; qualidade varia por SO |
| **Apple / Android Geocoder** | Sem billing Google; resultados locais |

Prefira o provider já adotado pelo projeto. Em transporte (`taxi-machine`), Google costuma ser a fonte canônica no backend.

## Regras inegociáveis

- Debounce reverse quando o usuário arrasta o pin (300–500 ms).
- Cache por geohash/grid curto (ex.: ~30–100 m) para evitar spam de API.
- Locale (`language`) alinhado ao usuário/tenant.
- UI: estado loading no label do endereço; não mostre coordenada crua como destino final.
- Trate ZERO_RESULTS com fallback (“Local selecionado no mapa”).

## Modelo de saída

```text
Address {
  formatted: string
  street?, number?, district?, city?, state?, postalCode?, country?
  placeId?  // se vier de Google
  lat, lng
}
```

- Exiba `formatted` na lista; use componentes para analytics/freight zones se necessário.
- Persistência: salve lat/lng + formatted + placeId quando houver.

## Fluxo pin no mapa

```text
onRegionChangeComplete / onDragEnd
  → debounce
  → reverseGeocode(center)
  → setAddress label
```

- Não dispare a cada `onRegionChange` (só ao final).
- Seed inicial: reverse da posição atual (`geolocation`).

## Anti-padrões

- Reverse a cada frame de drag
- Concatenar campos null virando vírgulas vazias
- Misturar forward e reverse no mesmo helper sem clareza
- Expor key Google no app para reverse em massa (use backend)

## Critérios de conclusão

- Pin drag → endereço estável e legível
- Loading e zero-results tratados
- Cache/debounce reduzem calls
- Locale correto no mercado do tenant
- Estrutura tipada consumível pela UI e pelo backend
