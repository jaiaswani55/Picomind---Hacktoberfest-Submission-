# Local AI Server on Raspberry Pi 3B

A lightweight, locally running AI chatbot server built on the **Raspberry Pi 3 Model B with just 1 GB RAM**. This project explores running small language models on low-resource hardware using [`llama.cpp`](https://github.com/ggml-org/llama.cpp) and quantised GGUF models.

The goal is simple: run AI locally, keep inference on-device, and minimise reliance on cloud APIs.

## Features

- **Local inference:** Generate responses directly on the Raspberry Pi.
- **Low-memory operation:** Designed for a device with 1 GB of physical RAM.
- **Quantised models:** Use compact GGUF models to reduce memory requirements.
- **CPU-based inference:** No dedicated GPU required.
- **Offline operation:** Chat without an internet connection after downloading the model and installing the required software.
- **Experimentation:** Test different models, context sizes, thread counts and generation speeds.

## Hardware

| Component | Specification |
|---|---|
| Board | Raspberry Pi 3 Model B |
| RAM | 1 GB |
| Storage | 8 GB microSD card |
| Processor | Broadcom BCM2837, quad-core 64-bit ARM Cortex-A53 |
| Accelerator | No dedicated AI accelerator |

## Software Stack

- **Operating system:** Raspberry Pi OS Lite
- **Inference engine:** `llama.cpp`
- **Model format:** GGUF
- **Model quantisation:** 4-bit, such as `Q4_K_M`
- **Interface:** Command-line chatbot; server API can be added separately

## Model Selection

The project focuses on small instruction-tuned language models that can fit within the Pi's limited memory.

| Model | Intended use |
|---|---|
| SmolLM2-135M-Instruct | Experiments prioritising speed and low memory usage |
| SmolLM2-360M-Instruct | Exploring the balance between speed and response quality |
| Qwen2.5-0.5B-Instruct | Testing a larger compact chatbot model |

Model performance varies with quantisation, context length, software build and available memory. These models are experimental candidates, not guaranteed to run equally well on the hardware.

## Getting Started

### 1. Prepare the operating system

Install Raspberry Pi OS Lite using [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

A minimal installation is recommended to preserve memory and storage. Ensure the selected OS image supports your Raspberry Pi and the chosen `llama.cpp` build.

### 2. Install build dependencies

Update the package lists and install the tools needed to build `llama.cpp`:

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

Building on a Raspberry Pi 3 can take time. If memory pressure causes compilation failures, reduce parallelism or compile on another compatible system and transfer the resulting binaries.

### 4. Download a compatible GGUF model

Choose a small, instruction-tuned GGUF model from a trusted model repository. Check its licence and use a quantised version appropriate for the device.

Keep the model file on the microSD card, for example:

```text
models/
└── model.gguf
```

The filename above is a placeholder; it is not an actual downloaded model.

### 5. Run the chatbot

From the `llama.cpp` directory, run:

```bash
./build/bin/llama-cli \
  -m /path/to/model.gguf \
  -t 4 \
  -c 512 \
  -n 100
```

Replace `/path/to/model.gguf` with the actual model path.

These are initial settings for experimentation. Try different thread counts and context sizes while monitoring memory use and temperature.

## Running an HTTP Server

`llama.cpp` also includes an HTTP server executable, commonly built as `llama-server`.

For a compatible model, a basic launch command is:

```bash
./build/bin/llama-server \
  -m /path/to/model.gguf \
  -t 4 \
  -c 512 \
  --host 127.0.0.1 \
  --port 8080
```

This binds the server to the local machine. To access it from another device, configure network binding and firewall rules deliberately. Do not expose an unauthenticated AI server directly to the public internet.

## Performance Testing

The project can be evaluated using:

- **Tokens per second (TPS):** Measure generation speed.
- **Memory consumption:** Track RAM usage and swap activity.
- **Time to first token:** Measure how long the model takes to begin responding.
- **Response quality:** Test a consistent set of questions across models.
- **Thermal behaviour:** Check whether sustained inference causes CPU throttling.

For repeatable performance measurements, use `llama-bench` where supported and record the model, quantisation, context size, thread count and software version.

## Hardware Constraints

The Raspberry Pi 3 Model B is a low-resource platform. This project has several practical limitations:

- Only 1 GB of physical RAM is available.
- Larger models may fail to load or leave insufficient memory for the operating system.
- CPU-only inference can be slow.
- Swap can help with memory pressure but is much slower than RAM and does not improve inference speed.
- The 8 GB microSD card limits the space available for the OS, build dependencies and models.

Small models are more practical than large general-purpose LLMs on this hardware.

## Project Goals

1. Run a language model locally on low-cost hardware.
2. Measure the performance of quantised models on ARM CPUs.
3. Explore memory-efficient inference and configuration.
4. Compare model size, response quality and generation speed.
5. Develop a lightweight foundation for a private, offline chatbot.

## Future Improvements

- Add a simple web-based chat interface.
- Expose a controlled local API for other applications.
- Benchmark additional compact GGUF models.
- Optimise startup time, memory usage and context configuration.
- Add system monitoring for CPU temperature, RAM and TPS.
- Explore startup automation and local-network access.

## Privacy

Prompts and generated responses can remain on the device when local inference is used. However, network access, telemetry, downloaded dependencies and any additional integrations should be reviewed separately to verify the privacy properties of the complete setup.

## Acknowledgements

- [`llama.cpp`](https://github.com/ggml-org/llama.cpp) for efficient local language-model inference.
- [Hugging Face](https://huggingface.co/models) for hosting model repositories and documentation.
- The developers of the selected open-weight language models.

## Licence

The licence for this project should be specified by the repository owner. Individual language models may have separate licence terms; check the selected model's licence before redistributing its files.

---

**Project status:** Experimental — development and performance testing on Raspberry Pi 3 Model B with 1 GB RAM.
