---
name: ollama-api
description: Integração com a API do Ollama — chat, generate, streaming, tools, embeddings e endpoint OpenAI-compatible. Use ao conectar apps a modelos locais Ollama, depurar /api/chat ou migrar clients OpenAI para localhost.
---

# Ollama API

Ollama expõe HTTP local para LLMs. Prefira `/api/chat` para conversas multi-turn; use o endpoint OpenAI-compatible quando quiser reutilizar clients existentes.

## Stack e escopo

| Endpoint | Papel |
|----------|--------|
| `POST /api/chat` | Chat messages + streaming |
| `POST /api/generate` | Completion de prompt único |
| `POST /api/embeddings` | Vetores |
| `GET /api/tags` | Modelos puxados |
| `POST /v1/chat/completions` | Compatível OpenAI |

Base default: `http://localhost:11434`. Modelo deve existir localmente (`ollama pull`).

## Regras inegociáveis

- Rode e configure Ollama no servidor/backend — não exponha a porta crua à internet sem auth/proxy.
- Trate timeouts e cancelamento; geração pode ser longa.
- Streaming: consuma NDJSON (`stream: true`) até `done`.
- Fixe o nome do modelo via config/env; falhe claro se o modelo não estiver puxado.
- Não envie secrets do usuário em logs de prompt.

## Chat

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [
    {"role": "system", "content": "You are concise."},
    {"role": "user", "content": "Explain iterators in one sentence."}
  ],
  "stream": false
}'
```

```ts
const res = await fetch("http://localhost:11434/api/chat", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: process.env.OLLAMA_MODEL,
    messages,
    stream: false,
    options: { temperature: 0.2 },
  }),
});
const data = await res.json();
const text = data.message?.content;
```

- Roles: `system` / `user` / `assistant` / `tool` conforme suporte do modelo.
- `options` (temperature, num_ctx, etc.) por request ou default do modelfile.
- Tools/function calling: use o campo `tools` + mensagens `tool` quando o modelo suportar.

## Streaming

- `stream: true` → linhas JSON (NDJSON); última com `"done": true`.
- No server do app: faça proxy do stream para o client (SSE) se o browser não falar com Ollama direto.
- Propague abort (`AbortSignal` / cancel do reader) até o upstream.

## OpenAI-compatible

```ts
// baseURL: http://localhost:11434/v1
// apiKey: "ollama" (placeholder)
```

- Útil para libs que já falam Chat Completions.
- Nem todo parâmetro OpenAI é honrado — valide no modelo alvo.
- Embeddings: `/api/embeddings` ou rota compatível do seu client.

## Ops e qualidade

- `ollama pull <model>` no provisionamento/CI; pin de tag/digest quando possível.
- GPU/CPU e `num_parallel` afetam latência — meça p95 no hardware real.
- Concurrency: fila no app se Ollama saturar.
- Health: `GET /api/tags` ou generate curto no readiness.

## Anti-padrões

- Browser → Ollama sem proxy/auth em rede não confiável
- Assumir que todo modelo suporta tools/vision
- Bloquear o event loop sem timeout lendo stream
- Logar prompts completos com PII
- Trocar modelo em prod sem teste de qualidade/latency

## Critérios de conclusão

- Modelo configurável e presente no host
- Chat ou generate com erros/timeouts tratados
- Streaming cancelável se a UX for stream
- Ollama não exposto publicamente sem proteção
- Contrato de mensagens estável para o app
