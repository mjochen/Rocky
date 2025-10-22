# LLM

## The setup

Docker compose:
* Ollama
    * Runs model
    * Model needs to be downloaded, but only once (volume)
    * GPU would be nice
* rest-wrapper
    * Python fastApi, needs to be built
    * Has personality on files (accessible, not just on volume)
    * Is go between for webUI
* WebUI
    * Optional (is big)
    * But nice

# Old information


## What You Can Realistically Run

You’re unlikely to run GPT-3.5 or anything close in size unless you're okay with slow responses and reduced context length. However, these options are promising:
Tiny / Small Models (Recommended for Your Use Case)

- [Phi-2 (2.7B)] — Tiny model by Microsoft, fast, very capable for basic pet-style conversation. Runs well on CPU.
- [Gemma 2B (CPU quantized)] — Google's small open model, also good on CPU with quantization.
- [Mistral 7B (quantized)] — Might be pushing it for your CPU, but possible with quantized formats like int4 and llama.cpp.

## Use in Your Robot Pet

Since you just need short bursts of interaction, not long coherent essays:

- You’re golden with a 2–3B model.
- Add some behavior logic around it (“If you see the owner, say something random and cute”).
- Use external scripts for speech-to-text and text-to-speech — e.g. whisper.cpp and piper.

## Bonus Tip: Ollama

Super easy setup tool for running local models:

curl -fsSL https://ollama.com/install.sh | sh
ollama run phi

Or, to run Mistral:

ollama run mistral

It handles downloads, quantization, and optimization for you.

## Summary

Your setup can handle it, especially with a lightweight model like Phi-2 or Gemma 2B, assuming:

- You're fine with sub-second to multi-second latency.
- You quantize to int4 or int8.
- You offload bigger stuff (e.g., vision or long-term memory) elsewhere.

Let me know if you want a step-by-step for setting it up with ollama or llama.cpp, or if you're thinking of specific personality behavior!

## Benefits of GPU Access

1. Massive Speed Boost for LLMs
    - Even though the K620 is old (Kepler-era, 2014) and has only 2 GB VRAM, it's still much faster than CPU for running quantized small models (like Phi-2 or TinyLlama).
    - Expect response times to drop from seconds/token on CPU to tenths of a second/token on GPU.
1. Enables running llama.cpp or ggml with GPU offload
    - These libraries can push most of the inference to the GPU.
    - Even partial offloading (e.g., first few layers) helps reduce CPU load.
1. Better parallelism
    - Your VM can stay responsive for other tasks since the GPU handles the heavy lifting.
    
    