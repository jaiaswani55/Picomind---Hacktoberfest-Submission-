# Setup Guide — Local AI Server on Raspberry Pi 3B

This guide explains how to prepare a Raspberry Pi 3 Model B to run Qwen2.5-0.5B-Instruct locally using `llama.cpp`.

The target hardware has 1 GB RAM and an 8 GB microSD card, so the setup prioritises low resource usage.

## 1. Requirements

### Hardware

- Raspberry Pi 3 Model B
- 8 GB microSD card
- Compatible power supply
- Computer for preparing the SD card
- Internet connection for initial setup
- Optional microphone for speech-to-text (STT)

### Software

- Raspberry Pi OS Lite
- Git
- CMake
- C/C++ build tools
- `llama.cpp`
- Qwen2.5-0.5B-Instruct in GGUF format

> **Important:** Keep sufficient free disk space during installation. Building software on an 8 GB card can require substantially more temporary storage than running the finished program.

## 2. Install Raspberry Pi OS

1. Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/).
2. Insert the microSD card into your computer.
3. Select Raspberry Pi 3 Model B as the target device.
4. Choose a compatible Raspberry Pi OS Lite image.
5. Configure your username, password and network settings if required.
6. Write the image to the SD card.
7. Insert the card into the Raspberry Pi and boot it.

A Lite installation without a graphical desktop is recommended.

For this low-memory system, Raspberry Pi OS Lite 32-bit is a conservative choice. A compatible 64-bit installation is also possible, but verify that your selected software and binaries match the OS architecture.

## 3. Update the System

After logging in, run:

```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

Reconnect after the Raspberry Pi restarts.

Check available storage and memory:

```bash
df -h /
free -h
uname -m
```

These commands show the available filesystem space, memory usage and CPU architecture.

## 4. Install Build Dependencies

Install the tools required to compile `llama.cpp`:

```bash
sudo apt install -y git cmake build-essential
```

If you are using SSH, keep the session stable while compiling. The Pi 3 has limited CPU and RAM, so building can take a while.

## 5. Download and Build llama.cpp

Clone the official repository:

```bash
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp
```

Configure the build:

```bash
cmake -B build
```

Compile with limited parallelism:

```bash
cmake --build build --config Release -j2
```

If compilation fails due to memory pressure, stop other applications and try a single build job:

```bash
cmake --build build --config Release -j1
```

If the SD card runs out of space, building on another compatible Linux computer and transferring the required binaries may be easier.

Official documentation: https://github.com/ggml-org/llama.cpp

## 6. Download Qwen2.5-0.5B-Instruct

This project uses Qwen2.5-0.5B-Instruct, an instruction-tuned language model with approximately 0.5 billion parameters.

Official GGUF repository:

https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF

Choose a compatible Q4_K_M GGUF file if available.

Create a directory for model files:

```bash
mkdir -p ~/local-ai-server/models
```

Download the selected model from its official repository using the download method documented by the repository.

Store it in the `models` directory. Confirm the exact filename:

```bash
ls -lh ~/local-ai-server/models/
```

Do not assume the filename in an example matches the actual downloaded file.

## 7. Run the Language Model

First, check that the model file exists and that the executable was built:

```bash
ls -lh ~/local-ai-server/models/
ls -lh ~/llama.cpp/build/bin/llama-cli
```

If you cloned `llama.cpp` somewhere other than `~/llama.cpp`, adjust the paths accordingly.

Run the chatbot:

```bash
~/llama.cpp/build/bin/llama-cli \
  -m ~/local-ai-server/models/YOUR_MODEL_FILE.gguf \
  -t 2 \
  -c 512 \
  -n 100
```

Replace `YOUR_MODEL_FILE.gguf` with the real filename.

Start with two CPU threads. You can test four threads later and compare the results. A higher thread count does not necessarily improve generation speed.

The context size of 512 tokens is a starting point intended to limit memory consumption. The prompt and generated output both consume context capacity.

## 8. Run the Local HTTP Server

If the server executable was built successfully, you can run:

```bash
~/llama.cpp/build/bin/llama-server \
  -m ~/local-ai-server/models/YOUR_MODEL_FILE.gguf \
  -t 2 \
  -c 512 \
  --host 127.0.0.1 \
  --port 8080
```

This binds the server to localhost.

Test it from another terminal on the Raspberry Pi:

```bash
curl http://127.0.0.1:8080/health
```

If the endpoint responds successfully, the server is running. If not, check the server logs and the endpoint behaviour for your installed version of `llama.cpp`.

To access the server from another device, configure local-network binding and access controls deliberately. Do not expose an unauthenticated service directly to the public internet.

## 9. Speech-to-Text Setup

The intended voice-input pipeline is:

1. Capture audio from a microphone.
2. Pass the recording to a local speech-to-text engine.
3. Convert the recognised speech into text.
4. Send the text to Qwen2.5-0.5B-Instruct.
5. Display the generated response.

The STT engine must be selected and installed separately. Its dependencies and model files will affect available RAM and storage.

### Hardware considerations

The Raspberry Pi has only 1 GB of physical RAM. Running speech recognition and language-model inference at the same time can cause high CPU usage and memory pressure.

For initial testing:

- Use short audio recordings.
- Select a lightweight STT model.
- Avoid running unnecessary background applications.
- Monitor memory use during both transcription and generation.
- Consider sequential processing instead of running both models simultaneously.

Offline STT requires a recognition engine and model that support local inference.

## 10. Monitor System Resources

Check RAM:

```bash
free -h
```

Check remaining storage:

```bash
df -h /
```

Check CPU temperature:

```bash
vcgencmd measure_temp
```

The temperature command is available on compatible Raspberry Pi OS installations. If it is unavailable, use another supported system-monitoring method.

Check CPU and memory usage:

```bash
top
```

If the system becomes unstable or begins using substantial swap, reduce the context size, close unnecessary processes or reconsider the model configuration.

## 11. Troubleshooting

### The model fails to load

- Verify the path and filename.
- Confirm the file is a valid GGUF model.
- Check that your `llama.cpp` build supports the model architecture.
- Check available RAM and storage.

### The system runs out of memory

- Close unnecessary applications.
- Reduce the context size.
- Reduce build parallelism if compilation is the problem.
- Avoid running STT and LLM inference simultaneously until each works independently.

### Generation is slow

- Compare two and four CPU threads.
- Keep prompts short.
- Avoid excessively long contexts.
- Check CPU temperature and possible thermal throttling.

### The SD card runs out of space

- Check `df -h /`.
- Remove unnecessary build artifacts only after confirming they are not needed.
- Avoid storing duplicate model files.
- Consider a larger microSD card if repeated builds and model downloads exceed available storage.

### The HTTP server cannot be reached

- Confirm that `llama-server` is running.
- Verify the configured port and listening address.
- Check the endpoint using `curl`.
- Remember that `127.0.0.1` is accessible only from the Raspberry Pi itself.

## 12. Recommended Test Checklist

- [ ] Raspberry Pi OS Lite boots successfully.
- [ ] Sufficient SD-card storage remains available.
- [ ] `llama.cpp` compiles successfully.
- [ ] The Qwen GGUF model downloads correctly.
- [ ] The chatbot generates a response.
- [ ] RAM usage remains manageable.
- [ ] The HTTP server starts, if required.
- [ ] Microphone input works, if required.
- [ ] STT converts a short recording into text.
- [ ] Recognised text can be passed to the chatbot.
- [ ] Offline operation is tested after setup.

## 13. Final Notes

This project targets a resource-constrained computer. Actual performance depends on the OS, model quantisation, runtime version, context length, available RAM and thermal conditions.

Record measured tokens per second, transcription latency and peak memory use after testing. Do not assume performance values without measurements on the device.

For model specifications, refer to [MODEL_DETAILS.md](MODEL_DETAILS.md).

For the project overview, refer to [README.md](README.md).
