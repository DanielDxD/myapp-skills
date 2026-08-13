---
name: react-native-live-activities
description: Implementa Live Activities no iOS (ActivityKit) e atualizações ongoing no Android a partir de React Native/Expo — start/update/end, push, permissões e limites de background. Use ao mostrar progresso ao vivo (viagem, entrega, pedido) na Lock Screen, Dynamic Island ou notificação persistente.
---

# React Native Live Activities

Live Activities comunicam estado **em andamento** fora do app (Lock Screen / Dynamic Island no iOS). No Android, use foreground notification / ongoing progress equivalente. Não substituem push genérico nem tracking GPS completo.

Transporte: `taxi-machine`. Localização: `react-native-realtime-location` / `react-native-background-location`.

## Escopo por plataforma

| Plataforma | Mecanismo |
|------------|-----------|
| **iOS** | ActivityKit Live Activity (+ push-to-update quando remoto) |
| **Android** | Ongoing notification / live updates (FGS se política exigir) |

- iOS 16.1+ para Live Activities; teste Dynamic Island em dispositivos suportados.
- Expo: use o módulo/config plugin que o projeto já adotar (`expo-live-activity` ou bridge nativa documentada). Bare: target Widget Extension + ActivityKit.

## Regras inegociáveis

- Conteúdo **atual** e útil (ETA, status, ação curta) — não marketing.
- Ciclo claro: `start` → `update` (throttled) → `end` / `dismiss`.
- Sem PII sensível na superfície da activity (mask telefone, etc.).
- Deep link de volta ao fluxo correto no app.
- Respeite quotas de update; batch/throttle (ex.: a cada N s ou mudança de estado).
- Declarações de permissão e background modes só com uso real.

## Modelo de dados

```text
TripLiveState {
  phase: searching | assigned | arriving | in_progress | completed
  title, subtitle
  etaMinutes?
  progress? // 0..1
  deepLink
}
```

- Mapeie estados de domínio → UI compacta (poucos campos).
- Atualize só quando `phase`, ETA ou progresso mudarem de forma perceptível.

## iOS (ActivityKit)

- Attribute type estável; não quebre o schema entre updates.
- UI da extension: layouts compactos (lock / minimal / expanded).
- Push updates: token da activity + APNs; autentique o backend.
- Encerre ao completar/cancelar; stale date coerente.
- Teste kill do app: updates remotos ainda devem funcionar se push estiver configurado.

## Android

- Canal de notificação dedicado (`trip_progress`), importância adequada.
- Ongoing enquanto a viagem/pedido estiver ativo; limpe ao terminar.
- Se tracking contínuo: alinhe com foreground service e políticas Play (`react-native-background-location`).
- Ações da notificação: abrir app / cancelar (com confirmação no app).

## Integração RN

- API JS tipada: `startLiveActivity(state)`, `updateLiveActivity(id, patch)`, `endLiveActivity(id)`.
- Fonte da verdade = estado da viagem no client/backend; a activity é projeção.
- Feature-detect: no-op graceful se a plataforma/versão não suportar.

## Anti-padrões

- Activity eterna sem `end`
- Spam de updates a cada GPS tick
- Duplicar lógica de negócio na Widget Extension
- Começar activity sem permissão/autorização do usuário quando exigido
- Mostrar endereço completo + dados sensíveis desnecessários

## Critérios de conclusão

- Start/update/end testados no fluxo principal
- Deep link correto
- Throttle de updates definido
- iOS Lock Screen / Island (ou fallback) e Android ongoing verificados
- Sem leaks de activity após viagem encerrada
