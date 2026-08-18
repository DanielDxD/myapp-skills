---
name: golang-video-dash
description: Processa e transmite vídeo em Go — transcode MP4 para MPEG-DASH com FFmpeg, ladders ABR, jobs assíncronos e serving de manifest/segmentos. Use ao converter vídeo, empacotar DASH/CMAF, servir .mpd/.m4s ou montar pipeline de VOD em backends Go.
---

# Golang Video DASH

MP4 progressivo não é streaming adaptativo. Transcode **fora** do request HTTP; sirva `.mpd` + segmentos CMAF. Upload resumível: `tus-golang-nextjs`.

## Stack e escopo

| Peça | Papel |
|------|--------|
| **FFmpeg** | Decode, encode, muxer `dash` |
| **ffprobe** | Duração, codecs, resolução |
| **Job/worker** | Fila pós-upload (`golang-goroutines`) |
| **Object storage / disco** | Artefatos DASH |
| **Gin** | Static/range dos segmentos (se o app servir) |

VOD (arquivo completo) é o default. Live DASH/HLS fica fora salvo o projeto já for live.

## Pipeline

```
MP4 (TUS complete) → probe → transcode DASH → store → publicar URL do .mpd
```

1. Upload termina (hook TUS `PreFinish` / `PostFinish`).
2. Enfileire job com `videoID`; responda `202` — não bloqueie o handler.
3. Worker: probe → FFmpeg com timeout/context → grave `manifest.mpd` + `init-*.m4s` + `chunk-*.m4s`.
4. Marque status `ready` / `failed`; aponte o player só ao `.mpd` quando ready.

## Regras inegociáveis

- FFmpeg via `exec.CommandContext` — cancele no shutdown e no timeout do job.
- Nunca transcode no goroutine do handler Gin sem fila + limite de concorrência.
- Paths: IDs opacos; nunca concatene filename do client em `exec`.
- Validar codec/container no probe; rejeite formatos não suportados antes do encode.
- Segmentos + manifest atômicos: publique o `.mpd` só quando o job completar (staging dir → rename/upload).
- MIME corretos: `application/dash+xml` (`.mpd`), `video/iso.segment` ou `video/mp4` (`.m4s`).

## FFmpeg — MP4 → MPEG-DASH

Ladder mínima (ajuste ao produto): 360p / 720p / 1080p, AAC estéreo, H.264 `+faststart` equivalente em CMAF (init separado).

```bash
ffmpeg -y -i "$IN" -hide_banner -loglevel error \
  -filter_complex "[0:v]split=3[v360][v720][v1080]; \
    [v360]scale=w=640:h=360:force_original_aspect_ratio=decrease:force_divisible_by=2[v360o]; \
    [v720]scale=w=1280:h=720:force_original_aspect_ratio=decrease:force_divisible_by=2[v720o]; \
    [v1080]scale=w=1920:h=1080:force_original_aspect_ratio=decrease:force_divisible_by=2[v1080o]" \
  -map "[v360o]" -map "[v720o]" -map "[v1080o]" -map 0:a:0? \
  -c:v libx264 -preset veryfast -profile:v high -pix_fmt yuv420p \
  -c:a aac -b:a 128k -ac 2 \
  -b:v:0 400k -maxrate:v:0 600k -bufsize:v:0 800k \
  -b:v:1 1500k -maxrate:v:1 2000k -bufsize:v:1 3000k \
  -b:v:2 4000k -maxrate:v:2 5000k -bufsize:v:2 8000k \
  -g 48 -keyint_min 48 -sc_threshold 0 \
  -adaptation_sets "id=0,streams=v id=1,streams=a" \
  -use_timeline 1 -use_template 1 -seg_duration 4 \
  -init_seg_name "init-\$RepresentationID\$.m4s" \
  -media_seg_name "chunk-\$RepresentationID\$-\$Number%05d\$.m4s" \
  -f dash "$OUT/manifest.mpd"
```

- GOP alinhado a `seg_duration` (ex. 4s @ 24fps → `-g 96` se fps for 24; derive do probe).
- Sem áudio: omita `-map 0:a` com base no probe — não deixe FFmpeg falhar opaco.
- `libx265`/`av1` só se o player e o custo de CPU forem aceitos; H.264 é o default de compatibilidade.
- Preserve o comando em config/templates versionados; logue o exit code e stderr truncado.

## Job em Go

```go
cmd := exec.CommandContext(ctx, "ffmpeg", args...)
cmd.Dir = workDir
var stderr bytes.Buffer
cmd.Stderr = &stderr
if err := cmd.Run(); err != nil {
    return fmt.Errorf("ffmpeg: %w: %s", err, truncate(stderr.String(), 4<<10))
}
```

- `SetLimit` no worker pool (1–N conforme CPU/GPU).
- Disco: quota por job; limpe `workDir` no `defer` após upload dos artefatos.
- Idempotência: mesmo `videoID` não dispara dois encodes (lock/status `processing`).
- Progresso opcional: parse `-progress pipe:1` se a UX pedir percentual.

## Serving

- CDN / object storage na frente dos `.m4s`; origin Go só se o volume for baixo.
- CORS se o player (Next.js) estiver noutro host: `GET`/`HEAD` nos artefatos.
- Cache-Control: `manifest.mpd` curto se o job puder republicar; segmentos imutáveis `immutable`.
- Não use `http.ServeFile` de um diretório com path do usuário sem allowlist de `videoID`.
- Player: dash.js / Shaka no front; URL pública só do `.mpd` (+ token se o VOD for privado).

## Anti-padrões

- Transcode síncrono no `POST` de upload
- `ffmpeg` com args interpolados do filename original
- Um único bitrate “1080p” vendido como DASH
- Servir o MP4 original no player “enquanto o DASH não fica pronto” sem produto explícito
- GOP/segmentos desalinhados (seeking quebrado)
- Fila unbounded de FFmpeg

## Critérios de conclusão

- Job assíncrono com context, limite e status
- `.mpd` + init/media segments publicados atomicamente
- Ladder ABR e codecs documentados
- MIME, CORS e cache corretos
- Sem path traversal / injection no FFmpeg
- Player consegue ABR e seek no artefato gerado
