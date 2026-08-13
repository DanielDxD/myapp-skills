---
name: react-native-background-location
description: Rastreamento de localização em background no React Native/Expo com permissões, task manager, foreground service Android, bateria e políticas das stores. Use ao rastrear motorista/entrega com app em background ou killed, ou configurar background location updates.
---

# React Native Background Location

Background location é privilegiado, caro em bateria e fortemente regulado. Use só quando o valor do produto exige (motorista em viagem, navegação ativa). Para UI em primeiro plano, prefira `react-native-realtime-location`.

Fundamentos de coordenadas: `geolocation`. Transporte: `taxi-machine`.

## Stack típica (Expo)

| Peça | Papel |
|------|--------|
| `expo-location` | Permissões + watch |
| `expo-task-manager` | Task em background |
| Config plugin / Info.plist / Manifest | Declarações de uso |
| Android FGS | Serviço em primeiro plano + notificação |

Em bare, equivalente nativo com as mesmas políticas.

## Regras inegociáveis

- Justificativa clara nas purpose strings (iOS) e na Play declaration (Android).
- Peça **quando** o usuário inicia o modo que exige tracking (ex.: “Iniciar viagem”), não no first launch genérico.
- Distinguir When-In-Use vs Always; só peça Always se background real for necessário.
- Notificação persistente no Android enquanto o FGS de location estiver ativo.
- Parar tracking ao fim da viagem / logout / cancelamento — sem sessões órfãs.
- Accuracy e interval mínimos suficientes (não `BestForNavigation` 24/7).

## Fluxo de permissões

1. Explique na UI o porquê (tela in-app antes do dialog do SO).
2. Solicite When-In-Use; só depois Always, se preciso.
3. Se negado: degrade (só foreground) + CTA para Ajustes.
4. Re-cheque ao resume (`AppState`) — usuário pode revogar.

## Task em background

```text
startTrip()
  → ensure permissions
  → startLocationUpdatesAsync(TASK, options)
  → updates → fila local / POST backend (batch)
endTrip()
  → stopLocationUpdatesAsync(TASK)
  → flush fila
```

- Opções: `accuracy`, `timeInterval` / `distanceInterval`, `showsBackgroundLocationIndicator` (iOS), `foregroundService` (Android).
- Idempotência no upload; backoff se offline.
- Não faça trabalho pesado na task; enfileire.

## Bateria e qualidade

- Aumente intervalo quando parado (`is_moving` / speed ≈ 0) se o produto permitir.
- Evite GPS + rede redundantes sem necessidade.
- Meça drain em device real antes de shippar defaults agressivos.

## Stores e compliance

- App Store: purpose strings específicas; evite “para melhorar experiência” genérico.
- Play: declare foreground service types / background location conforme a política vigente; prepare vídeo/demo de uso legítimo.
- Política de privacidade deve mencionar coleta contínua quando ativa.

## Anti-padrões

- Always permission no onboarding
- Continuar tracking após fim da corrida
- Ignorar falha de permissão (crash ou silêncio total)
- Logar coordenadas em analytics sem base legal
- Confiar só no Simulator para validar background

## Critérios de conclusão

- Start/stop confiáveis com app background/killed (cenários reais)
- Purpose strings e notificação Android corretas
- Upload resiliente offline
- Permissão negada tratada com degradação
- Sem tracking órfão após logout/fim de viagem
