---
name: taxi-machine
description: Orquestra a criação de apps de transporte white-label (taxi/rideshare) com papéis passageiro/motorista/ops, fluxos de pedido-matching-viagem-pagamento, branding e composição de maps, location, navegação e Live Activities. Use ao construir ou evoluir um app de transporte multi-tenant ou white-label.
---

# Taxi Machine

Skill orquestradora para produtos de transporte white-label. Define domínio, fluxos e como compor as skills técnicas — não substitui backend/pagamentos específicos do projeto.

## Skills compostas

| Necessidade | Skill |
|-------------|--------|
| Navegação de telas (Expo SDK 57) | `react-native-expo-navigation` |
| UI / tema white-label | `better-mobile-interface-react-native` |
| Mapa nativo | `google-maps-sdk` |
| Directions / Places / Geocoding HTTP | `google-maps-apis` |
| Trajeto no mapa | `react-native-routes` |
| GPS pontual | `geolocation` |
| Endereço a partir de coordenada | `reverse-geocoding` |
| Posição ao vivo (app aberto) | `react-native-realtime-location` |
| Tracking motorista em background | `react-native-background-location` |
| Status na Lock Screen / ongoing | `react-native-live-activities` |
| App RN geral | `react-native-development` |

## White-label

Config por tenant (remoto ou build-time):

```text
TenantConfig {
  brand: { name, logo, accent, colorScheme }
  legal: { privacyUrl, termsUrl, supportPhone }
  maps: { provider: google, regionBias }
  features: { scheduledRides, chat, tips, multiStop }
  payments: { provider, methods[] }
}
```

- Tokens de UI derivados de `accent` + neutros (`better-mobile-interface-react-native`).
- Sem hardcode do nome do cliente nas screens — use config.
- Features via flags; não compile fluxos mortos se o tenant não usa.

## Papéis

| Papel | Responsabilidades principais |
|-------|------------------------------|
| **Passageiro** | Origem/destino, estimar preço, solicitar, acompanhar, pagar, avaliar |
| **Motorista** | Online/offline, aceitar, navegar até pax/destino, status da viagem, ganhos |
| **Ops** (se no escopo) | Monitorar viagens, suporte, fraude básica |

Apps separados ou flavors/targets por papel — preserve o padrão do monorepo.

## Máquina de estados da viagem

```text
idle → estimating → requested → matching → accepted
    → driver_arriving → waiting_pax → in_trip → completed
    → (canceled | expired | failed) a partir de requested..waiting_pax
```

Regras:

- Transições só no backend (fonte da verdade); client otimista com reconciliação.
- IDs estáveis `tripId`; idempotência em create/cancel/pay.
- Timeouts: matching sem motorista → expired; waiting_pax → no-show policy do produto.
- Canceled por quem e com motivo tipado.

## Fluxos obrigatórios

### Passageiro

1. Autocomplete origem/destino (`google-maps-apis` Places) + pin no mapa.
2. Reverse geocode do pin (`reverse-geocoding`).
3. Estimativa (preço/ETA) antes do request.
4. Matching + Live Activity / push.
5. Mapa ao vivo do motorista (`react-native-realtime-location` no lado motorista + socket).
6. Rota motorista→pax e pax→destino (`react-native-routes`).
7. Pagamento e recibo; avaliação.

### Motorista

1. Toggle online + permissões location (When-In-Use; Always se background).
2. Oferta de corrida com timeout; aceitar/recusar.
3. Navegação / rota até pickup e dropoff.
4. Background tracking só **durante** viagem ativa (`react-native-background-location`).
5. Status: chegou / iniciou / finalizou.
6. Live Activity / ongoing notification espelhando fase.

## Mapa e localização

- SDK no device (`google-maps-sdk`); rotas e geocode via APIs com key restrita.
- Keys: iOS/Android bundle restrictions; server key só no backend.
- Região default do tenant (city bias) para Places.
- Privacy: retenção de trail GPS alinhada à política; não logar trajetos em analytics crus.

## Pagamentos e compliance (alto nível)

- Preauth/capture conforme o provider do projeto.
- Recibo e disputa: estados claros no client.
- Categorias de app nas stores: mobilidade; declare background location com uso legítimo.
- Termos e privacidade linkados no onboarding.

## Estrutura de app sugerida

```text
apps/passenger/
apps/driver/
packages/domain-trip/     # tipos e transições
packages/ui-theme/        # tokens white-label
packages/maps/
```

Ajuste ao monorepo existente.

## Anti-padrões

- GPS Always ligado com motorista offline
- Preço só no client sem validação server
- Mapa sem estados de loading/erro de permissão
- Branding hardcoded impedindo segundo tenant
- Pular estimativa e ir direto ao charge

## Critérios de conclusão

- Estados da viagem cobertos nos dois apps (ou papéis)
- Config white-label aplica marca sem rebuild de lógica
- Maps + routes + location + live activity compostos corretamente
- Permissões pedidas no momento certo
- Cancel/expire/pagamento com feedback claro
- Fluxo feliz passageiro e motorista testado em device
