# Local AI Server on Raspberry Pi 3B

A lightweight, locally running AI chatbot powered by **Qwen2.5-0.5B-Instruct**, deployed on a Raspberry Pi 3 Model B with just 1 GB of RAM.

This project explores running a quantised small language model on low-resource hardware using [`llama.cpp`](https://github.com/ggml-org/llama.cpp). The goal is to create a personal AI system that can generate responses locally without depending on cloud-based inference APIs.

## Features

- **Local AI inference** — run Qwen2.5 directly on the Raspberry Pi.
- **Low-memory deployment** — designed for a device with 1 GB of physical RAM.
- **Quantised model** — use the GGUF format to reduce memory requirements.
- **CPU-only execution** — no dedicated GPU required.
- **Offline chatbot** — operate without internet access after setup and model download.
- **Open-source inference engine** — powered by `llama.cpp`.

## Hardware

| Component | Specification |
|---|---|
| Board | Raspberry Pi 3 Model B |
| Processor | Broadcom BCM2837, quad-core 64-bit ARM Cortex-A53 |
| RAM | 1 GB |
| Storage | 8 GB microSD card |
| AI acceleration | CPU only |

## Software Stack

- **Operating system:** Raspberry Pi OS Lite
- **Language model:** Qwen2.5-0.5B-Instruct
- **Inference engine:** `llama.cpp`
- **Model format:** GGUF
- **Quantisation:** Q4_K_M
- **Execution:** Local CPU inference

## Model Information

**Qwen2.5-0.5B-Instruct** is a compact, instruction-tuned language model from the Qwen2.5 family.

The model has approximately 0.5 billion parameters and is intended for lightweight language-generation tasks.

This project uses a quantised GGUF version to reduce storage and memory requirements.

Official model repository: [Qwen2.5-0.5B-Instruct-GGUF](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF)

### Why this model?

- Small enough to experiment with on resource-constrained hardware.
- Instruction-tuned for conversational interactions.
- Supports local inference through `llama.cpp`.
- Does not require a dedicated GPU or cloud API.

**Hardware note:** The Raspberry Pi 3B has only 1 GB of physical RAM. Successful loading and stable operation depend on the OS, model quantisation, context size and runtime configuration. Performance is expected to be limited compared with modern computers.

## Installation

### 1. Install Raspberry Pi OS Lite

Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/) and install a compatible Raspberry Pi OS Lite image.

A minimal installation is recommended to preserve available RAM and storage.

### 2. Install build dependencies

```bash
sudo apt update
sudo apt install -y git cmake build-essential
```

### 3. Build llama.cpp

```bash
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp

cmake -B build
cmake --build build --config Release -j2
```

Compilation may take some time on the Raspberry Pi 3B. If memory pressure causes the build to fail, reduce parallelism or compile on a compatible computer and transfer the binaries.

### 4. Download the model

Download a compatible **Qwen2.5-0.5B-Instruct GGUF** file from the official repository:

[Browse available model files](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF)

Choose the `Q4_K_M` quantisation if available and compatible with your runtime.

Store the model in a dedicated directory, for example:

```text
local-ai-server/
└── models/
    └── qwen2.5-0.5b-instruct-q4_k_m.gguf
```

The filename is an example; use the actual name of the downloaded file.

### 5. Run the chatbot

From the `llama.cpp` directory, execute:

```bash
./build/bin/llama-cli \
  -m /path/to/qwen2.5-0.5b-instruct-q4_k_m.gguf \
  -t 4 \
  -c 512 \
  -n 100
```

Replace the model path with the actual location of your downloaded file.

These settings are a starting point. Monitor memory consumption and test different thread counts if performance is poor.

## Running a Local HTTP Server

`llama.cpp` provides an HTTP server executable that can expose model inference through an API.

Example command:

```bash
./build/bin/llama-server \
  -m /path/to/qwen2.5-0.5b-instruct-q4_k_m.gguf \
  -t 4 \
  -c 512 \
  --host 127.0.0.1 \
  --port 8080
```

This example binds the service to localhost. It can be used for local testing if the executable is available and the model loads successfully.

To allow other devices on your local network to connect, configure the listening address and firewall deliberately. Do not expose an unauthenticated server directly to the public internet.

## Performance Evaluation

The project can be evaluated using the following measurements:

- **Tokens per second (TPS):** Measures text-generation speed.
- **Time to first token:** Measures response startup latency.
- **RAM consumption:** Measures memory used during inference.
- **Context length:** Evaluates the effect of prompt size on memory use.
- **Thermal behaviour:** Checks for CPU throttling during sustained workloads.
- **Response quality:** Tests the model on a consistent set of prompts.

Actual benchmark results should be recorded from the Raspberry Pi rather than estimated.

## Limitations

- Only 1 GB of physical RAM is available.
- CPU-only inference limits generation speed.
- Longer contexts increase memory requirements.
- Swap memory is significantly slower than physical RAM.
- The 8 GB microSD card restricts the space available for the OS, build tools and model files.
- A 0.5B-parameter model has limited reasoning and factual reliability compared with larger models.

## Future Improvements

- Add a lightweight web-based chat interface.
- Provide a local API for other applications.
- Benchmark tokens per second under different configurations.
- Monitor CPU temperature, RAM usage and inference latency.
- Automate server startup.
- Improve local-network access controls.

## Privacy

Inference can remain entirely on the Raspberry Pi without sending prompts to a cloud AI provider. Internet access is still required for initial downloads unless the software and model are transferred to the device manually.

Review additional integrations and network settings separately to ensure the complete setup meets your privacy requirements.

## Acknowledgements

- [`llama.cpp`](https://github.com/ggml-org/llama.cpp) — local language-model inference.
- [Qwen](https://huggingface.co/Qwen) — Qwen2.5 model family.
- [Hugging Face](https://huggingface.co/) — model hosting and documentation.

## Licence

This repository's licence should be specified by its owner. Check the Qwen model's licence and the licences of all bundled dependencies before redistributing them.

---

**Project status:** Experimental. Focused on evaluating Qwen2.5-0.5B-Instruct on a Raspberry Pi 3 Model B with 1 GB RAM and 8 GB storage.
