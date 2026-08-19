---
name: react-email
description: Cria templates de e-mail com React Email 6 — acessíveis, compatíveis com Gmail/Outlook/Apple Mail e visualmente cuidados. Use ao criar ou revisar e-mails transacionais, magic links, receipts, newsletters em React, ou quando o usuário pedir react-email, templates de e-mail ou HTML para clientes de mail.
---

# React Email 6

E-mail não é o browser. Flex, Grid e JS morrem no Outlook. O HTML final precisa ser tabelas, estilos inline, texto puro e um visual que sobreviva a Gmail, Outlook Windows, Apple Mail e mobile.

Pacote unificado **`react-email`** (v6). Não instale `@react-email/components` em projeto novo. Preview: `@react-email/ui`.

## Stack

```bash
npm i react-email @react-email/ui
```

```tsx
import {
  Html, Head, Body, Container, Section, Row, Column,
  Text, Heading, Button, Img, Link, Hr, Preview, Font, render,
} from "react-email"
```

- `render(<Email />)` → HTML.
- `render(<Email />, { plainText: true })` → parte texto. **Sempre envie as duas.**
- Tailwind via wrapper do React Email só se o projeto já usa; prefira style objects inline para previsibilidade.

## Anatomia

```
emails/
  _layout.tsx          # Html, Head, Preview, Body, Container, footer
  auth/magic-link.tsx
  transactional/receipt.tsx
  marketing/welcome.tsx
```

Layout único (~600px) compartilhado. Props tipadas por template. Sem data fetching dentro do e-mail — o caller passa dados já resolvidos.

## Regras inegociáveis

1. Layout com `Container` + `Section` + `Row`/`Column` — **nunca** flexbox/grid/`div` solto para estrutura.
2. Largura de conteúdo **600px** (atributo + style). Outlook ignora `max-width` sozinho.
3. Estilos **inline** (objetos `style={{}}`). Sem stylesheet externo; `<style>` só para reset mínimo / dark-mode progressivo no `Head`.
4. CTA com componente `Button` (VML/MSO incluso). Não invente `<a>` com padding de “botão CSS”.
5. Toda `Img` tem `width`, `height` e `alt` (vazio `alt=""` só se for decorativa).
6. `Html lang="pt-BR"` (ou o idioma real). `Preview` com 40–90 caracteres úteis — não “Não responda este e-mail” como preview.
7. Parte **plain text** equivalente (mesmos links e fatos).
8. Sem JavaScript, forms interativos, vídeo autoplay ou webfont como única fonte.

## Compatibilidade

| Fazer | Evitar |
|-------|--------|
| `Row` / `Column` (tabelas + MSO) | `display: flex`, Grid, `gap` |
| `width={600}` no container | Fluid 100% sem constraint |
| `Button` do pacote | `<div>` clicável, `border-radius` em `<a>` cru |
| Cores hex (`#111827`) | `oklch()`, `color-mix`, CSS variables como única fonte |
| System fonts + fallback | Só `"Inter"` / fonte hospedada |
| PNG/JPG hospedados HTTPS | SVG como conteúdo crítico, CID sem necessidade |
| Background sólido | `background-image` como informação |
| Margin via `Section`/`Text` style | Margin collapsing de browser |

Clientes mínimos a considerar: Gmail (web + app), Outlook **Windows** (Word engine), Outlook Mac, Apple Mail, iOS Mail, Yahoo. Teste real (Litmus, Email on Acid, ou envio para contas) antes de chamar de pronto.

Gmail pode **inverter cores** em dark mode. Não dependa de texto claro em fundo claro “porque o media query salva”. Fundos e textos com contraste em **ambos** os modos, ou aceite inversão com cores mid-contrast.

```tsx
<Head>
  <meta name="color-scheme" content="light dark" />
  <meta name="supported-color-schemes" content="light dark" />
</Head>
```

## Acessibilidade

- Hierarquia: um `Heading as="h1"` por e-mail; depois `h2` seções. Não pule níveis.
- Corpo ≥ **14px**, line-height ~1.5. Links sublinhados ou com texto óbvio (“Redefinir senha”), nunca “clique aqui”.
- Contraste **WCAG AA** (4.5:1 texto normal). Accent de marca não é desculpa para cinza claro em branco.
- Informação não só pela cor (erro: texto + prefixo “Erro:”, não só vermelho).
- Imagens de logo: `alt` com nome do produto. Tracking pixels: `alt=""`, 1×1, nunca conteúdo.
- Espaço de toque no CTA (~44px de altura via padding do `Button`).
- Prefira HTML semântico que o React Email já emite (`p`, `h1`, `a`, `img`) a `role` ARIA que clientes ignoram.

## Design

E-mail bonito é ritmo e restrição, não CSS moderno.

- **Um** accent de marca (botão + links) com contraste AA. Fundo do canvas `#f4f4f5`, card `#ffffff`, texto `#111827`, muted `#52525b`.
- Hierarquia: h1 24–28px / body 16px / footer 12–13px muted.
- Espaço vertical consistente (24/32px entre blocos). Sem seções coladas.
- Um CTA primário. Secundário vira link de texto.
- Largura 600, padding interno 24–32px. Mobile: colunas viram stack via `Row`/`Column` (não media queries complexas).
- Logo pequeno no topo (altura ~32–40px), não hero de 800px.
- Sem gradients, glassmorphism, sombras pesadas, bordas arredondadas excessivas (Outlook some com elas).
- Footer: quem envia, por quê, link de preferências/unsubscribe se marketing (obrigatório em muita jurisdição).

Padrões:

| Tipo | Conteúdo |
|------|----------|
| **Auth** | Quem pediu, botão de ação, validade, aviso “se não foi você ignore”, URL crua abaixo do botão |
| **Transacional** | Fato (pedido #, valor), tabela simples `Row`/`Column`, próximo passo |
| **Welcome / e-commerce** | Uma mensagem, um CTA, um produto — não newsletter de 12 blocos |

## Esqueleto

```tsx
import { Html, Head, Preview, Body, Container, Section, Text, Heading, Button, Link, Img, Hr } from "react-email"

const font = "system-ui, -apple-system, 'Segoe UI', sans-serif"

export function MagicLinkEmail({ url, expiresMinutes }: { url: string; expiresMinutes: number }) {
  return (
    <Html lang="pt-BR">
      <Head />
      <Preview>Seu link de acesso vale por {expiresMinutes} minutos</Preview>
      <Body style={{ margin: 0, backgroundColor: "#f4f4f5", fontFamily: font }}>
        <Container style={{ maxWidth: 600, width: "100%", margin: "32px auto", backgroundColor: "#ffffff" }}>
          <Section style={{ padding: "32px 32px 0" }}>
            <Img src="https://example.com/logo.png" width="120" height="32" alt="Acme" />
          </Section>
          <Section style={{ padding: "24px 32px" }}>
            <Heading as="h1" style={{ fontSize: 24, color: "#111827", fontWeight: 600 }}>
              Entrar na Acme
            </Heading>
            <Text style={{ fontSize: 16, lineHeight: "24px", color: "#3f3f46" }}>
              Use o botão abaixo para acessar sua conta. O link expira em {expiresMinutes} minutos.
            </Text>
            <Button href={url} style={{ backgroundColor: "#18181b", color: "#fafafa", padding: "12px 20px" }}>
              Acessar conta
            </Button>
            <Text style={{ fontSize: 13, lineHeight: "20px", color: "#52525b" }}>
              Se o botão não funcionar, copie:{" "}
              <Link href={url} style={{ color: "#18181b", textDecoration: "underline" }}>{url}</Link>
            </Text>
          </Section>
          <Hr />
          <Section style={{ padding: "0 32px 32px" }}>
            <Text style={{ fontSize: 12, color: "#71717a" }}>
              Se você não pediu este e-mail, ignore. Acme nunca pede senha por mensagem.
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  )
}
```

## Envio

- HTML + `text/plain`. List-Unsubscribe em marketing.
- Imagens em CDN HTTPS estável; não embuta base64 pesado.
- Peso total baixo (imagens abaixo de ~100KB cada quando possível).
- Preview local (`react-email` / UI) **não** substitui Outlook Windows.

## Anti-padrões

- Import de `@react-email/components` em app v6
- Flex/Grid, `gap`, `position: absolute`
- CTA sem `Button`
- Preview vazio ou igual ao assunto
- Sem plain text
- Texto branco em fundo branco “no dark mode”
- `alt` faltando ou `alt="image"`
- “Clique aqui”
- Gradient, vídeo, mapa de imagem como navegação única

## Integração com outras skills

- Copy: `website-copywriting`
- Auth flows (servidor gera o link): `authentication-authorization`, `phoenix`, `ruby-on-rails`
- UI web **não** se aplica 1:1 — não copie Tailwind de app para e-mail sem inlining

## Critérios de conclusão

- Imports de `react-email` (v6)
- Layout 600px em componentes de tabela; um `Button` de CTA
- `lang`, `Preview`, `alt`, contraste AA, corpo ≥14px
- HTML + plain text
- Visual sóbrio (ritmo, um accent, hierarquia) — não “landing page”
- Pelo menos o caminho Outlook + Gmail considerado (documentado ou testado)
