---
name: chatgpt-api
description: Integração com a API OpenAI (ChatGPT) — Chat Completions, Responses, streaming, tools, structured outputs e erros. Use ao chamar modelos OpenAI em backends, implementar function calling ou migrar clients para o SDK openai.
---

# ChatGPT API (OpenAI)

Chaves só no servidor. Prefira o SDK oficial da linguagem do projeto. Para apps novos, alinhe ao estilo de API que o repositório já usa (**Chat Completions** ou **Responses**).

## Stack e escopo

| Peça | Papel |
|------|--------|
| **Chat Completions** | `POST /v1/chat/completions` — amplamente usado |
| **Responses API** | API mais nova unificada (se o projeto já migrou) |
| **SDK `openai`** | TS/Python/etc. |
| **Tools** | Function calling |
| **Auth** | `Authorization: Bearer <OPENAI_API_KEY>` |

Base default: `https://api.openai.com/v1`. Azure OpenAI / proxies: base URL e auth distintos — preserve o do projeto.

## Regras inegociáveis

- API key apenas em backend/workers — nunca no frontend público.
- Model ID pinado em config/env; upgrade consciente.
- Timeouts e retries com backoff só em `429` / 5xx transitórios.
- Streaming com cancelamento propagado.
- Não logue prompts/respostas completas com PII/secrets.

## Chat Completions

```ts
import OpenAI from "openai";

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const completion = await client.chat.completions.create({
  model: process.env.OPENAI_MODEL!,
  messages: [
    { role: "system", content: "You are a precise coding assistant." },
    { role: "user", content: "Summarize this diff in 3 bullets." },
  ],
  temperature: 0.2,
});

const text = completion.choices[0]?.message?.content ?? "";
```

- Roles: `system` / `user` / `assistant` / `tool`.
- Leia `finish_reason` (`stop`, `tool_calls`, `length`, …).
- Structured output: `response_format` / JSON schema quando a versão e o modelo suportarem — valide no servidor de qualquer forma.

## Tools (function calling)

1. Declare `tools` com parâmetros JSON Schema.
2. Se a resposta trouxer `tool_calls`, execute no servidor.
3. Append mensagens `role: "tool"` com `tool_call_id` e continue o loop.

- Valide argumentos antes de side effects.
- Least privilege nas tools.
- Confirmação humana / idempotência para ações destrutivas.

## Streaming

```ts
const stream = await client.chat.completions.create({
  model: process.env.OPENAI_MODEL!,
  messages,
  stream: true,
});

for await (const chunk of stream) {
  const delta = chunk.choices[0]?.delta?.content;
  if (delta) {
    // encaminhar ao client
  }
}
```

- Proxie SSE/chunks sem bufferizar tudo se a UX for stream.
- Trate conexões cortadas e half-closed.

## Responses API (quando o projeto usa)

- Use o client `responses.create` / stream equivalente do SDK.
- Não misture shapes de Completions e Responses no mesmo adapter sem camada anti-corrupção.
- Tools, texto e usage seguem o contrato Responses — leia os tipos do SDK instalado.

## Erros e limites

| Situação | Ação |
|----------|------|
| `401` / `403` | Key/projeto — não retry cego |
| `429` | Backoff + rate limits / quota |
| `5xx` | Retry limitado ou fallback |
| `length` / context | Truncar histórico com estratégia explícita |

- Monitore tokens (`usage`) e custo por feature.
- Keys e projetos separados por ambiente.

## Compatibilidade

- Ollama e outros proxies podem expor `/v1/chat/completions` — veja `ollama-api`.
- Claude/Anthropic é outra API — veja `claude-api`; não reutilize payloads cegamente.

## Anti-padrões

- Key OpenAI no bundle do browser
- Loop de tools sem teto de iterações
- Retry em erro de schema/validação
- Confiar só em “JSON mode” sem parse/validate
- Misturar Azure OpenAI e api.openai.com na mesma config sem intenção

## Critérios de conclusão

- Key e model via env/config seguros
- Completions ou Responses alinhados ao projeto
- Tool loop correto e limitado se houver tools
- Streaming cancelável quando aplicável
- 429/5xx tratados; usage observável
