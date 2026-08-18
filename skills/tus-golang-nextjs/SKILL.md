---
name: tus-golang-nextjs
description: Upload resumível com protocolo TUS — servidor tusd/Gin em Go e cliente Next.js (tus-js-client). Use ao implementar upload de vídeo/arquivos grandes, retomar transferência, hooks pós-upload ou integrar TUS entre frontend Next e backend Go.
---

# TUS — Go + Next.js

TUS é PATCH resumível com `Upload-Offset`. O browser fala **direto** com o backend Go. Next.js não deve proxyar o arquivo inteiro. Após `PostFinish`, enfileire DASH: `golang-video-dash`.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **github.com/tus/tusd/v2** | Handler TUS (criação, HEAD, PATCH, concatenação se precisar) |
| **Gin** | Mount do handler, auth, CORS |
| **tus-js-client** | Cliente no App Router (`"use client"`) |
| **Hooks** | Validar meta, mover arquivo, disparar job |

TUS 1.0.x: header `Tus-Resumable: 1.0.0`. Preserve a major do `tusd` já no `go.mod`.

## Regras inegociáveis

- Upload **direto** Go (ou rewrite de path sem bufferizar o body no Node).
- Authn/authz na **criação** (`POST`) e de novo nos `PATCH`/`HEAD` do mesmo upload.
- Limite `Upload-Length` e MIME (`video/mp4`, …) no hook — não confie no client.
- CORS: exponha e permita headers TUS (`Tus-Resumable`, `Upload-Offset`, `Upload-Length`, `Upload-Metadata`, `Location`).
- Disco/S3: um upload = um ID opaco; filename original só em metadata.
- `PostFinish`: transicione status e enfileire processamento; não transcode no hook.

## Backend Go (tusd + Gin)

```go
composer, err := tusd.NewStoreComposer()
store := filestore.FileStore{Path: cfg.TusDir}
store.UseIn(composer)

handler, err := tusd.NewHandler(tusd.Config{
    BasePath:              "/files/",
    StoreComposer:         composer,
    NotifyCompleteUploads: true,
    PreUploadCreateCallback: func(hook tusd.HookEvent) error {
        // authz já feita no middleware; valide length/MIME aqui
        return nil
    },
})

r.Use(tusCORS())
g := r.Group("/files")
g.Use(Authenticate())
g.Any("/*path", gin.WrapH(http.StripPrefix("/files/", handler)))
```

- `BasePath` e `StripPrefix` **iguais** ao path público (inclua trailing slash como o tusd espera).
- Consuma `handler.CompleteUploads` (ou `UnroutedHandler` + hooks) numa goroutine com dono — ver `golang-goroutines`.
- Metadata TUS é Base64; decode e parse (`filename`, `contentType`, `videoId`) com tamanho limitado.
- Produção: store S3 (`s3store`) + `memorylocker`/`redis` locker entre instâncias; file store só em single-node/dev.
- `MaxSize` no config tusd alinhado ao produto (ex. 2–10 GiB).

### CORS (browser → Go)

Permita `POST, HEAD, PATCH, OPTIONS, DELETE` e headers:

`Authorization, Cookie, Content-Type, Tus-Resumable, Upload-Length, Upload-Offset, Upload-Metadata, Upload-Defer-Length, Upload-Concat`

Exponha: `Location, Tus-Resumable, Tus-Version, Tus-Extension, Tus-Max-Size, Upload-Offset, Upload-Length`.

`Access-Control-Allow-Credentials` só se a sessão for cookie; senão Bearer no `Authorization`.

## Frontend Next.js

Não use Server Action para o binário. Componente client + `tus-js-client`:

```tsx
"use client";

import { Upload } from "tus-js-client";

export function startVideoUpload(file: File, token: string) {
  return new Promise<string>((resolve, reject) => {
    const upload = new Upload(file, {
      endpoint: process.env.NEXT_PUBLIC_TUS_ENDPOINT!, // https://api.example.com/files/
      chunkSize: 5 * 1024 * 1024,
      retryDelays: [0, 1000, 3000, 5000],
      headers: { Authorization: `Bearer ${token}` },
      metadata: {
        filename: file.name,
        contentType: file.type || "video/mp4",
      },
      onError: reject,
      onSuccess: () => resolve(upload.url!),
      onProgress: (sent, total) => {
        /* percent = sent / total */
      },
    });
    upload.findPreviousUploads().then((prev) => {
      if (prev[0]) upload.resumeFromPreviousUpload(prev[0]);
      upload.start();
    });
  });
}
```

- `NEXT_PUBLIC_TUS_ENDPOINT` aponta ao **Go**, não a `/api` do Next, salvo rewrite streaming sem limite de body.
- Fingerprint/`removeFingerprintOnSuccess`: retomar após refresh na mesma origem.
- Pause/abort: `upload.abort()`; não deixe PATCH órfão sem UX.
- Valide `file.type` e size **também** no UI; o servidor é a fonte da verdade.
- Após success: chame a API/action só para “registrar URL/id” — metadados, não o arquivo.

## Next.js: o que evitar

| Evite | Faça |
|-------|------|
| Route Handler que lê o File e faz `fetch` ao Go | Cliente TUS → Go |
| `serverActions.bodySizeLimit` enorme | Binário fora do Next |
| Esquecer OPTIONS preflight | CORS TUS completo no Go |
| Cookie `SameSite=Strict` + API cross-site | Bearer ou SameSite=None+Secure consciente |

Rewrite (`next.config` `rewrites` para `/files`) só se não bufferizar; prefira domínio de API.

## Ligação ao DASH

```
create video (API) → status=uploading
TUS POST/PATCH → bytes no store
PostFinish → status=processing → job FFmpeg DASH
ready → player usa /vod/{id}/manifest.mpd
```

- Amarrar `Upload-Metadata` a um `videoId` já autenticado (criado antes do TUS).
- Não publique o MP4 cru se o produto for só DASH; trate o original como scratch e expire.

## Anti-padrões

- Multipart clássico “um POST de 4GB” pelo Next
- TUS sem auth nos PATCH
- Aceitar qualquer `Upload-Length`
- Hook síncrono pesado (FFmpeg no `PostFinish`)
- CORS `*` com credentials
- Confiar no `filename` para path no disco

## Critérios de conclusão

- POST/HEAD/PATCH TUS funcionam e retomam após corte de rede
- Authz em criação e continuação
- Limites de tamanho/MIME no servidor
- CORS/headers TUS corretos no browser
- Next só orquestra UI + metadata
- `PostFinish` enfileira DASH sem bloquear o upload
