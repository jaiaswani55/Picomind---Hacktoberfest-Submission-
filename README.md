# Local AI Server on Raspberry Pi 3B

A lightweight, locally running AI chatbot powered by **Qwen2.5-0.5B-Instruct**, designed for the Raspberry Pi 3 Model B with just 1 GB of RAM.

This project explores running a quantised small language model on low-resource hardware using [`llama.cpp`](https://github.com/ggml-org/llama.cpp), with speech-to-text (STT) integration to support voice-based interaction. The goal is to build a compact, locally operated AI system without depending on cloud-based inference APIs.

## Features

- **Local AI inference** — run Qwen2.5-0.5B-Instruct directly on the Raspberry Pi.
- **Speech-to-text (STT)** — developing voice input by converting spoken audio into text.
- **Low-memory deployment** — designed for a device with 1 GB of physical RAM.
- **Quantised model** — use the GGUF format to reduce model storage requirements.
- **CPU-only execution** — no dedicated GPU required.
- **Offline operation** — intended to work without internet access after the required software and models have been installed.
- **Lightweight architecture** — designed for low-cost, resource-constrained hardware.

## Hardware

| Component | Specification |
|---|---|
| Board | Raspberry Pi 3 Model B |
| Processor | Broadcom BCM2837, quad-core 64-bit ARM Cortex-A53 |
| RAM | 1 GB |
| Storage | 8 GB microSD card |
| AI acceleration | CPU only |
| Audio input | Microphone, when connected |

## Software Stack

- **Operating system:** Raspberry Pi OS Lite
- **Language model:** Qwen2.5-0.5B-Instruct
- **Inference engine:** `llama.cpp`
- **Model format:** GGUF
- **Quantisation:** Q4_K_M
- **Programming language:** Python and C/C++ components, as required
- **Speech processing:** Speech-to-text engine, selected according to hardware and performance requirements
- **Execution:** Local CPU inference

## AI Language Model

### Qwen2.5-0.5B-Instruct

Qwen2.5-0.5B-Instruct is a compact, instruction-tuned language model from the Qwen2.5 family, developed by the Qwen team at Alibaba Cloud.

The model has approximately 0.5 billion parameters and is intended for lightweight text-generation and instruction-following tasks.

This project uses a quantised GGUF version to reduce storage and memory requirements.

**Official model repository:** [Qwen2.5-0.5B-Instruct-GGUF](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF)

### Why Qwen2.5-0.5B-Instruct?

- Small enough to experiment with on resource-constrained hardware.
- Instruction-tuned for conversational interactions.
- Compatible with `llama.cpp` through a supported GGUF model.
- Can generate responses locally without a cloud inference API.
- Suitable for exploring compact language models on low-cost hardware.

### Model Configuration

| Property | Configuration |
|---|---|
| Model | Qwen2.5-0.5B-Instruct |
| Parameters | Approximately 0.5 billion |
| Format | GGUF |
| Target quantisation | Q4_K_M |
| Runtime | `llama.cpp` |
| Context size | Start with 512 tokens |
| Execution | CPU |

The actual model file and quantisation should be verified against the downloaded file. Successful operation depends on memory availability, runtime configuration and context length.

## Speech-to-Text (STT)

The project is being developed to support voice input, allowing users to interact with the chatbot by speaking instead of typing.

Speech-to-text converts spoken audio into text that can be processed by Qwen2.5-0.5B-Instruct.

### STT Workflow

1. **Audio capture:** Record speech through a connected microphone.
2. **Speech recognition:** Convert the audio into text using an STT engine.
3. **Text processing:** Pass the recognised text to the chatbot.
4. **Local AI inference:** Generate a response using Qwen2.5-0.5B-Instruct through `llama.cpp`.
5. **Response display:** Present the generated text to the user.

### STT Design Goals

- Enable voice-based interaction.
- Minimise memory consumption and processing latency.
- Prefer local speech recognition where supported.
- Support offline operation after downloading the required recognition model and dependencies.
- Allow speech recognition and language-model inference to operate within the Pi's limited resources.

### Hardware Considerations

Speech recognition and language-model inference share the Raspberry Pi's CPU and RAM. Running both simultaneously can significantly affect response speed and stability.

The choice of STT engine, recognition model size, audio sample rate and recording duration will affect performance.

A lightweight recognition engine and short audio segments are sensible starting points for this hardware.

**Current status:** The STT implementation and its performance should be documented according to the actual engine and configuration used in the project.

## Installation

### 1. Install Raspberry Pi OS Lite

Install a compatible Raspberry Pi OS Lite image using [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

A minimal installation without a desktop environment is recommended to preserve RAM and storage.

### 2. Install Build Dependencies

Update the system and install the tools required to build `llama.cpp`:

```bash
sudo apt update
sudo apt install -y git cmake build-essential
```

Building on the Raspberry Pi 3 can take time and may encounter memory pressure. If compilation fails, reduce parallelism or compile on a compatible computer and transfer the binaries.

### 3. Build llama.cpp

```bash
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp

cmake -B build
cmake --build build --config Release -j2
```

Refer to the [`llama.cpp` repository](https://github.com/ggml-org/llama.cpp) for current build requirements and instructions.

### 4. Download the Language Model

Download a compatible Qwen2.5-0.5B-Instruct GGUF model from the official repository:

[Browse Qwen2.5-0.5B-Instruct-GGUF files](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF)

Choose the Q4_K_M quantisation if available and compatible with your runtime.

A possible project structure is:

```text
local-ai-server/
├── README.md
├── MODEL_DETAILS.md
├── models/
│   └── qwen2.5-0.5b-instruct-q4_k_m.gguf
├── stt/
│   └── ...
└── src/
    └── ...
```

The directory names and model filename are examples and may differ from the actual implementation.

### 5. Run the Chatbot

From the `llama.cpp` directory, run:

```bash
./build/bin/llama-cli \
  -m /path/to/model.gguf \
  -t 4 \
  -c 512 \
  -n 100
```

Replace `/path/to/model.gguf` with the actual path to the downloaded model.

These settings are initial values for testing. Monitor RAM consumption and compare thread counts to determine which configuration works best on the device.

## Local HTTP Server

`llama.cpp` provides an HTTP server executable for exposing model inference through an API.

Example command:

```bash
./build/bin/llama-server \
  -m /path/to/model.gguf \
  -t 4 \
  -c 512 \
  --host 127.0.0.1 \
  --port 8080
```

This command assumes the server executable has been built and the model can be loaded successfully.

Binding to `127.0.0.1` restricts access to the local machine. For local-network access, configure network binding and access controls carefully. Do not expose an unauthenticated server directly to the public internet.

## Performance Evaluation

The project can be evaluated using the following metrics:

| Metric | Purpose |
|---|---|
| Tokens per second (TPS) | Measures language-model generation speed |
| Time to first token | Measures response startup latency |
| RAM consumption | Measures memory requirements |
| Model load time | Measures startup overhead |
| STT transcription latency | Measures speech-recognition processing time |
| Transcription accuracy | Evaluates recognition quality |
| CPU temperature | Helps identify thermal throttling |
| End-to-end response time | Measures the time from spoken input to chatbot response |

Actual measurements should be recorded on the Raspberry Pi rather than estimated.

## Limitations

- The device has only 1 GB of physical RAM.
- CPU-only inference limits generation speed.
- The language model has limited reasoning and factual reliability compared with larger models.
- Long context windows increase memory requirements.
- Speech recognition competes with language-model inference for CPU and RAM.
- Swap memory is substantially slower than physical RAM.
- An 8 GB microSD card restricts the space available for the operating system, model files, build tools and STT dependencies.
- Performance depends on the selected STT engine and model.

## Future Improvements

- Complete the speech-to-text integration.
- Add a lightweight chat interface.
- Connect the STT pipeline directly to the local inference server.
- Measure end-to-end voice response latency.
- Benchmark generation speed under different configurations.
- Monitor CPU temperature, RAM usage and swap activity.
- Automate startup of the local AI service.
- Improve local-network access controls.

## Privacy

The system is intended to process chatbot prompts locally without requiring a cloud inference API.

Fully offline voice interaction is possible only when the chosen STT engine, recognition model and all required dependencies support local execution and have already been installed.

Microphone permissions, network activity and any additional integrations should be reviewed separately.

## Acknowledgements

- [`llama.cpp`](https://github.com/ggml-org/llama.cpp) — local language-model inference.
- [Qwen](https://huggingface.co/Qwen) — Qwen2.5 language-model family.
- [Hugging Face](https://huggingface.co/) — model hosting and documentation.
- Developers of the speech-recognition engine used in the final implementation.

## Licence

Specify the licence for this project in the repository. Check the Qwen model's licence and the licences of all dependencies before redistributing model files or third-party software.

---

**Project status:** Experimental.

**Target platform:** Raspberry Pi 3 Model B, 1 GB RAM, 8 GB microSD card.

**Primary objective:** Explore local language-model inference and speech-to-text integration on low-resource hardware.
