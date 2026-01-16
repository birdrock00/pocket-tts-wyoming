# Pocket-TTS Wyoming Protocol Server

Wyoming protocol server for [Pocket-TTS](https://github.com/kyutai-labs/pocket-tts), enabling Home Assistant integration with voice selection support.

## Quick Start with Docker Compose

Create a `docker-compose.yml` file (or use the file included in this project):

```yaml
services:
  pocket-tts-wyoming:
    build: .
    container_name: pocket-tts-wyoming
    network_mode: host
    environment:
      - WYOMING_PORT=10201
      - WYOMING_HOST=0.0.0.0
      - DEFAULT_VOICE=alba
      - MODEL_VARIANT=b6369a24
      - ZEROCONF=pocket-tts
    restart: unless-stopped
    volumes:
      - pocket-tts-hf-cache:/root/.cache/huggingface
      - pocket-tts-cache:/root/.cache/pocket_tts

volumes:
  pocket-tts-hf-cache:
    driver: local
  pocket-tts-cache:
    driver: local
```

### Configuration

You can customize the following environment variables in the compose file before starting:

| Variable | Default | Description |
|----------|---------|-------------|
| `WYOMING_PORT` | `10201` | The port the Wyoming protocol server listens on. Change if you have a conflict with another service. |
| `WYOMING_HOST` | `0.0.0.0` | The network interface to bind to. `0.0.0.0` accepts connections from any interface. |
| `DEFAULT_VOICE` | `alba` | The default voice used when none is specified. See [Available Voices](#available-voices) for options. |
| `MODEL_VARIANT` | `b6369a24` | The Pocket-TTS model variant to use. This corresponds to a specific model checkpoint. |
| `ZEROCONF` | `pocket-tts` | Service name for mDNS/Zeroconf discovery. Home Assistant uses this to auto-discover the TTS server. Set to empty string to disable. |

Build and start:

```bash
docker compose up -d --build
```

## Building the Docker Image Manually

```bash
docker build -t pocket-tts-wyoming .
```

## Running with Docker

```bash
docker run -d \
  --name pocket-tts-wyoming \
  -p 10201:10201 \
  -e DEFAULT_VOICE=alba \
  -v pocket-tts-hf-cache:/root/.cache/huggingface \
  -v pocket-tts-cache:/root/.cache/pocket_tts \
  pocket-tts-wyoming
```

The volume mounts are recommended to cache model files and avoid re-downloads on restart.



## Available Voices

alba, marius, javert, jean, fantine, cosette, eponine, azelma

## Home Assistant Integration

The server supports Zeroconf/mDNS for automatic discovery.

1. Start the Docker container
2. Go to Settings -> Devices & Services -> Add Integration
3. Search for "Wyoming Protocol"
4. The server should appear in the "Discovered" section, or enter `tcp://<server-ip>:10201` manually
5. Configure a Voice Assistant pipeline to use the TTS service and select a voice

## Troubleshooting

- **Slow startup**: First run downloads ~500MB of model weights. Use volume mounts to persist the cache.
- **Connection issues**: Verify port 10201 is open and check logs with `docker logs pocket-tts-wyoming`
- **Voice not found**: Ensure the voice name matches one of the 8 predefined voices listed above.
