<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/icon-light.svg">
    <img src="assets/icon.svg" alt="Rhasspy logo" width="128">
  </picture>
</p>

<h1 align="center">aac-wyoming</h1>

<p align="center">Self-hosted deployment definition for Wyoming voice services: Piper text-to-speech and Faster Whisper speech-to-text.</p>

<p align="center">
  <a href="https://github.com/OHF-Voice/wyoming-piper"><img alt="Upstream" src="https://img.shields.io/badge/upstream-OHF--Voice%2Fwyoming--piper-blue?logo=github"></a>
  <a href="https://hub.docker.com/r/rhasspy/wyoming-piper"><img alt="Image" src="https://img.shields.io/badge/image-rhasspy%2Fwyoming--piper%3Alatest-2496ED?logo=docker&logoColor=white"></a>
  <a href="https://github.com/OHF-Voice/wyoming-piper/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-MIT-green"></a>
</p>

## About

This repository defines a two-container voice stack that speaks the Wyoming protocol, the
network protocol used by Home Assistant and other voice applications to exchange audio.

- **Piper** (text-to-speech) listens on TCP port `10200` and synthesizes speech with the
  German voice `de_DE-thorsten-high`.
- **Faster Whisper** (speech-to-text) listens on TCP port `10300` and transcribes German
  speech with the `small-int8` model.

Voice models are downloaded on first use and cached in persistent volumes.

Key features (as provided by upstream):

- Wyoming protocol server for Piper TTS (`rhasspy/wyoming-piper`)
- Wyoming protocol server for Faster Whisper STT (`rhasspy/wyoming-whisper`)
- Models cached in persistent volumes, so they are not downloaded again after a restart

## Upstream

- Piper project: <https://github.com/OHF-Voice/wyoming-piper>
- Piper container image: `rhasspy/wyoming-piper:latest` (<https://hub.docker.com/r/rhasspy/wyoming-piper>)
- Whisper container image: `rhasspy/wyoming-whisper:latest` (<https://hub.docker.com/r/rhasspy/wyoming-whisper>)
- License: MIT (per the upstream Piper wrapper repository; check the upstream repositories for the terms of each image)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. No
secrets are required: this service defines none in `service.yml`, so no `.env` file is needed.

For a manual run, start the containers from a Compose file that reflects the definition above:

```yaml
services:
  piper:
    image: rhasspy/wyoming-piper:latest
    restart: always
    command: --voice de_DE-thorsten-high --uri tcp://0.0.0.0:10200 --data-dir /data --download-dir /data
    ports:
      - "10200:10200"
    volumes:
      - piper_data:/data
  whisper:
    image: rhasspy/wyoming-whisper:latest
    restart: always
    command: --model small-int8 --language de --uri tcp://0.0.0.0:10300 --data-dir /data --download-dir /data
    ports:
      - "10300:10300"
    volumes:
      - whisper_data:/data

volumes:
  piper_data:
  whisper_data:
```

```bash
docker compose up -d
```

## Configuration

Environment variables (from `service.yml`, shared by the service):

| Variable | Default | Description |
|---|---|---|
| `SERVICE_IMAGE_TAG` | `latest` | Image tag of the service |
| `CUSTOM_NAME` | `Wyoming Voice` | Display name used by integrations |
| `SERVICE_NAME` | `aac-wyoming` | Service name |
| `SERVICE_NAME_LOWERCASE` | `aac-wyoming` | Lowercase service name |
| `SERVICE_HOSTNAME` | `aac-wyoming` | Hostname of the service |
| `TZ` | `Europe/Berlin` | Time zone |
| `PUID` | `1000` | User ID the container runs as |
| `PGID` | `1000` | Group ID the container runs as |

Piper command options: `--voice de_DE-thorsten-high`, `--uri tcp://0.0.0.0:10200`,
`--data-dir /data`, `--download-dir /data`.

Whisper command options: `--model small-int8`, `--language de`, `--uri tcp://0.0.0.0:10300`,
`--data-dir /data`, `--download-dir /data`.

Ports:

| Service | Port | Protocol | Purpose |
|---|---|---|---|
| Piper | `10200` | TCP | Text-to-speech (Wyoming) |
| Whisper | `10300` | TCP | Speech-to-text (Wyoming) |

Volumes:

| Volume | Mounted at | Description |
|---|---|---|
| `piper_data` | `/data` (Piper) | Persistent cache for Piper voice models |
| `whisper_data` | `/data` (Whisper) | Persistent cache for Faster Whisper models |

## Deploy with App-as-Code

The deployment is generated from `service.yml`; change the deployment by editing that file
rather than the rendered output. Releases of this deployment definition are published under
<https://github.com/app-as-code/aac-wyoming>.

## Credits and license

Wyoming, Piper and Faster Whisper belong to their upstream projects. This repository only
contains the deployment definition; the upstream license of each container image applies to
the software it contains. The icon and its source are described in
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md).
