---
name: geolocation
description: Obtém coordenadas geográficas (GPS/rede) com permissões, accuracy, erros e fallbacks em mobile e web. Use ao ler a posição atual do usuário, checar serviços de localização ou implementar getCurrentPosition de forma segura.
---

# Geolocation

Obter **uma** posição (ou snapshot) com permissão explícita. Streams contínuos: `react-native-realtime-location`. Background: `react-native-background-location`. Reverse address: `reverse-geocoding`.

## Fontes

| Ambiente | API típica |
|----------|------------|
| Expo RN | `expo-location` `getCurrentPositionAsync` |
| RN bare | `@react-native-community/geolocation` ou equivalente |
| Web | `navigator.geolocation` |

## Regras inegociáveis

- Solicite permissão antes; trate denied/restricted.
- Timeout e `maximumAge` conscientes (não travar a UI para sempre).
- Propague accuracy; descarte leituras piores que o limiar do produto quando crítico.
- Fallback UX se GPS off / sem sinal (entrada manual, último known, mapa arrastável).
- Não bloqueie o first paint sem necessidade — async + skeleton.

## Fluxo

```text
check services enabled
  → request permission
  → getCurrentPosition({ accuracy, timeout, maximumAge })
  → { lat, lng, accuracy, altitude?, heading?, speed?, timestamp }
```

- iOS: When-In-Use suficiente para snapshot.
- Android: fine vs coarse conforme o uso; fine para pin de pickup.

## Erros tipados

- `permission_denied`
- `services_disabled`
- `timeout`
- `unavailable`

Mapeie para copy acionável (“Ative a localização” / “Abrir Ajustes”).

## Privacidade

- Minimize retenção; não envie posição sem contexto de feature.
- Documente uso na política quando persistir no backend.

## Anti-padrões

- Assumir sucesso sem checar permission
- `enableHighAccuracy: true` sempre sem necessidade (bateria)
- Swallow de erro com `0,0` no Golfo da Guiné
- Pedir Always só para um get pontual

## Critérios de conclusão

- Snapshot confiável com permissão concedida
- Todos os erros principais com UX
- Accuracy disponível para consumidores (mapa/matching)
- Funciona em iOS e Android (e web se no escopo)
