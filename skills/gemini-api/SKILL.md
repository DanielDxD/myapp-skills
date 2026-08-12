---
name: gemini-api
description: Integração com a API Google Gemini — generateContent, streaming, system instructions, tools/function calling, vision e tratamento de erros. Use ao chamar Gemini em backends, implementar function calling ou migrar clients para o SDK Google Gen AI.
---

# Gemini API (Google)

Use a API de conteúdo generativo (`generateContent` / chat sessions). Chaves só no servidor; model ID e system instruction fazem parte do contrato da feature.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **generateContent** | Turnos de texto / multimodal |
| **SDK oficial** | `@google/genai` (TS) / `google-genai` (Python) — ou o client já no repo |
| **Streaming** | `generateContentStream` |
| **Tools** | Function calling / code execution / grounding (conforme produto) |
| **Auth** | API key (`GEMINI_API_KEY` / `GOOGLE_API_KEY`) ou ADC no Vertex |

Apps novos: API Google AI (ai.google.dev) com API key, salvo o projeto já estar em **Vertex AI** — nesse caso preserve endpoint, projeto e auth ADC/service account.

## Regras inegociáveis

- API key / credenciais apenas em backend/workers — nunca no frontend público.
- Pin de model ID em config/env (ex. família `gemini-2.5-flash` / o ID que o projeto padronizar).
- Timeouts e retries com backoff só em `429` / 5xx / erros transitórios de rede.
- Streaming: consuma o iterator; cancele upstream se o client abortar.
- Não logue prompts/respostas completas com PII/secrets.

## generateContent

```ts
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const response = await ai.models.generateContent({
  model: process.env.GEMINI_MODEL!,
  contents: "Summarize this diff in 3 bullets.",
  config: {
    systemInstruction: "You are a precise coding assistant.",
    temperature: 0.2,
    maxOutputTokens: 1024,
  },
});

const text = response.text;
```

- `contents`: string, lista de turns (`role` + `parts`) ou multimodal (texto + inline data / file URIs).
- Multi-turn: alterne `user` / `model` (não `assistant`).
- Leia finish/safety: bloqueios de safety podem vir sem texto útil — trate como erro de domínio.
- Se o projeto usar o SDK legado `@google/generative-ai`, preserve essa API; não migre major sem pedido.

## Chat session

```ts
const chat = ai.chats.create({
  model: process.env.GEMINI_MODEL!,
  config: { systemInstruction: "Be concise." },
});

const reply = await chat.sendMessage({ message: "Hello" });
```

- Use session quando o histórico for gerenciado pelo SDK; senão envie `contents` explícitos (mais fácil de testar/persistir).
- Persista histórico no seu store se a conversa atravessar requests HTTP.

## Tool / function calling

1. Declare function declarations (nome, descrição, parâmetros JSON Schema).
2. Se a resposta trouxer `functionCall`, execute no servidor.
3. Devolva `functionResponse` nas parts e continue até texto final.

- Valide argumentos contra schema antes de side effects.
- Least privilege: só tools necessárias à feature.
- Idempotência / confirmação humana para ações destrutivas.
- Limite iterações do loop tool↔model.

## Streaming

```ts
const stream = await ai.models.generateContentStream({
  model: process.env.GEMINI_MODEL!,
  contents: "Hello",
});

for await (const chunk of stream) {
  const delta = chunk.text;
  if (delta) {
    // encaminhar ao client
  }
}
```

- Proxie chunks (SSE) sem bufferizar tudo se a UX for stream.
- Trate conexões cortadas e respostas vazias por safety.

## Multimodal

- Imagens/áudio/PDF conforme o modelo: `inlineData` (base64 + mime) ou Files API / URIs suportadas.
- Valide MIME e tamanho no servidor antes do upload.
- Não assuma que todo model ID aceita o mesmo conjunto de modalidades.

## Vertex vs Google AI

| | Google AI (API key) | Vertex AI |
|--|---------------------|-----------|
| Auth | API key | ADC / service account |
| Config | key + model | project, location, model |
| Uso | protótipo/prod simples | enterprise, IAM, VPC |

- Não misture clients/env das duas stacks no mesmo adapter sem camada clara.
- Em GCP, prefira o padrão já adotado pelo repo (Secret Manager, workload identity).

## Erros e limites

| Situação | Ação |
|----------|------|
| `400` / invalid argument | Corrija request — não retry cego |
| `401` / `403` | Key/IAM — não retry cego |
| `429` | Backoff + rate limits |
| `5xx` / unavailable | Retry limitado ou fallback de modelo |
| Safety block | Mensagem segura ao usuário; log categorizado |

- Monitore tokens / custo por feature quando a API expuser usage metadata.
- Keys e projetos separados por ambiente.

## Compatibilidade

- OpenAI-compatible proxies ≠ Gemini nativo — veja `chatgpt-api` / `ollama-api` se a base URL for `/v1/chat/completions`.
- Claude/Anthropic é outra API — `claude-api`; não reutilize payloads cegamente.

## Anti-padrões

- API key Gemini no bundle do browser
- Loop de function calling sem teto
- Retry em bloqueio de safety ou erro de schema
- Assumir parity total entre model IDs (flash vs pro vs vision)
- Misturar Vertex e API key no mesmo client sem intenção

## Critérios de conclusão

- Key/credenciais e model via env/config seguros
- `generateContent` ou chat com system instruction claros
- Tool loop correto e limitado se houver tools
- Streaming cancelável quando aplicável
- 429/5xx e safety blocks tratados; usage observável nos caminhos críticos
