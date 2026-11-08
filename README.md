# llama.cpp-compose

Docker Compose setup for running [llama.cpp](https://github.com/ggml-org/llama.cpp)'s `llama-server` on a local GPU, with AMD (Vulkan) and NVIDIA (CUDA) profiles.

The server runs in router mode: it serves the models defined in `models-preset.ini`, downloads them from Hugging Face on first use, and keeps one loaded at a time. The OpenAI-compatible API and web UI are on `127.0.0.1:8080` (localhost only; change the `ports` mapping to expose it on your network).

## Requirements

- Linux with a GPU (Docker on macOS can't use the Apple Silicon GPU; run `llama-server` natively there)
- Docker with Compose v2
- For the default model (`Qwen3.8-27B-S`, IQ3_S, 64k context): about 16 GB of VRAM. Lower `ctx-size` or pick a smaller model for less.
- **AMD:** Mesa RADV Vulkan drivers on the host
- **NVIDIA:** the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

## Usage

```sh
docker compose --profile amd up -d      # or: --profile nvidia
curl http://localhost:8080/v1/models
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "Qwen3.8-27B-S", "messages": [{"role": "user", "content": "Hello"}]}'
```

`pull_policy: always` pulls the latest llama.cpp build on each `up`.

## Model presets

Each section in `models-preset.ini` is one model; the section name is the model name clients request. Keys are `llama-server` options without the leading `--`:

```ini
version = 1

[My-Model]
hf-repo = some-org/Some-Model-GGUF
hf-file = some-model-Q4_K_M.gguf
ctx-size = 32768
n-gpu-layers = 99
```

Downloads are cached in `./model-cache`. For gated or private repos, export `HF_TOKEN` before `docker compose up`.

## GPU notes

- **AMD:** the service passes `/dev/dri/renderD128`. If your GPU uses another node, check `ls /dev/dri` and change it. `RADV_PERFTEST=nogttspill` keeps allocations out of system memory; compiled shaders are cached in `./shader-cache-amd`.
- **NVIDIA:** all GPUs are exposed (`gpus: all`).

## License

[MIT](LICENSE)
