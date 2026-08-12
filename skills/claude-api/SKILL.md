---
name: claude-api
description: Integração com a API Anthropic (Claude) — Messages API, streaming SSE, system prompts, tools, vision e tratamento de erros. Use ao chamar Claude em backends, implementar function calling ou migrar clients para anthropic SDK.
---

# Claude API (Anthropic)

Use a **Messages API** (`POST /v1/messages`). Chaves só no servidor; system prompt e tools são parte do contrato da feature.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Messages API** | Chat multi-turn |
| **SDK oficial** | `anthropic` (Python/TS) ou HTTP raw |
| **Streaming** | SSE (`stream: true`) |
| **Tools** | Function calling / tool use |
| **Auth** | Header `x-api-key` + `anthropic-version` |

Base: `https://api.anthropic.com`. Modelo via env (ex. `claude-sonnet-4-5` ou o ID pinado pelo projeto).

## Regras inegociáveis

- API key apenas em backend/workers — nunca no frontend público.
- Pin de model ID em config; documente upgrade consciente.
- Timeouts, retries com backoff só em erros transitórios (`429`, `529`, rede).
- Streaming: parse SSE; cancele upstream se o client abortar.
- Não logue prompts/respostas completas com PII/secrets.

## Messages

```ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

const msg = await client.messages.create({
  model: process.env.CLAUDE_MODEL!,
  max_tokens: 1024,
  system: "You are a precise coding assistant.",
  messages: [{ role: "user", content: "Summarize this diff in 3 bullets." }],
});

const text = msg.content
  .filter((b) => b.type === "text")
  .map((b) => b.text)
  .join("");
```

- `max_tokens` é obrigatório.
- `system` é string ou lista de blocos — separado do array `messages`.
- Roles de `messages`: `user` | `assistant`; conteúdo pode ser texto ou blocos (image, tool_use, tool_result).
- Leia `stop_reason` (`end_turn`, `tool_use`, `max_tokens`, etc.).

## Tool use

1. Envie `tools` com JSON Schema dos parâmetros.
2. Se `stop_reason === "tool_use"`, execute tools no servidor.
3. Devolva `tool_result` nas próximas `messages` e continue até `end_turn`.

- Valide argumentos contra schema antes de executar side effects.
- Least privilege: tools só com o que a feature precisa.
- Idempotência / confirmação humana para ações destrutivas.

## Streaming

```ts
const stream = client.messages.stream({
  model: process.env.CLAUDE_MODEL!,
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello" }],
});

for await (const event of stream) {
  // content_block_delta → texto incremental
}
const final = await stream.finalMessage();
```

- Proxie eventos ao client (SSE) sem bufferizar tudo se a UX for stream.
- Trate `error` events e conexões cortadas.

## Visão e anexos

- Imagens como blocos `image` (base64 ou URL conforme API/SDK).
- Limite tamanho/MIME; valide no servidor antes do upload à API.
- PDFs/arquivos: use o mecanismo suportado pela versão da API do projeto.

## Erros e limites

| Situação | Ação |
|----------|------|
| `401` / `403` | Config/key — não retry cego |
| `429` | Backoff + respeitar rate limits |
| `529` overloaded | Backoff ou fallback de modelo |
| `max_tokens` | Continuar ou resumir com estratégia explícita |

- Monitore `usage` (input/output tokens) por request/feature.
- Separação de filas/keys por ambiente (dev/prod).

## Anti-padrões

- Chamar Anthropic de browser com a key secreta
- Concatenar tools sem validar argumentos
- Retry infinito em erro de validação
- System prompt gigante duplicado em cada client sem versão
- Assumir que todo modelo tem os mesmos limites de contexto/tools

## Critérios de conclusão

- Key e model via env/config seguros
- Messages (+ tools se houver) com loop de tool_result correto
- Streaming cancelável quando aplicável
- Erros 429/5xx tratados; 4xx de contrato não mascarados
- Usage/custos observáveis nos caminhos críticos
