# TurboQuant KV cache (turbo2/3/4) on PrismML llama.cpp

Branch `feat/turboquant-kv-cache` = branch `prism` (PrismML-Eng/llama.cpp) + ported
TurboQuant KV-cache feature set (turbo2 / turbo3 / turbo4 cache types) from
TheTom/llama-cpp-turboquant (tip `a3d5603d1`).

- 104 cherry-picked commits + 1 post-merge build-fix commit.
- CUDA + CPU build verified. Ternary Bonsai 2 (PQ2_0) smoke-tested on RTX 3060 (~32 tok/s decode).
- TurboQuant KV types: `-ctk turbo2|turbo3|turbo4`, `-ctv turbo2|turbo3|turbo4` (requires `-fa on`).

## Build

### Linux (CUDA 12.8)
```bash
cmake -B build-cuda128 -DCMAKE_BUILD_TYPE=Release -DGGML_CUDA=ON \
  -DCMAKE_CUDA_COMPILER=/usr/local/cuda-12.8/bin/nvcc -DCUDAToolkit_ROOT=/usr/local/cuda-12.8 \
  -DCMAKE_CUDA_ARCHITECTURES="86;120" -DLLAMA_CURL=OFF
cmake --build build-cuda128 -j12
```
Note: CUDA 13.2 + WSL driver 610.x crashes large binaries (`munmap_chunk`); use 12.8.

### Windows (VS 2022, Developer PowerShell)
```powershell
cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_CUDA=ON
cmake --build build --config Release -j
# binary lands in build\bin\Release\llama-cli.exe (+ llama-cli-impl.dll in the same folder)
```

## Run (Ternary Bonsai 2 27B example)
```bash
hf download prism-ml/Ternary-Bonsai-2-27B-gguf Ternary-Bonsai-2-27B-PQ2_0.gguf --local-dir .
./build-cuda128/bin/llama-cli -m Ternary-Bonsai-2-27B-PQ2_0.gguf \
  -ngl 99 -fa on -c 8192 -ctk turbo4 -ctv turbo4 \
  --temp 1.0 --top-p 0.95 --top-k 20 -st -n 128 -p "Hello"
```

Notes:
- With large GQA ratios the runtime auto-upgrades K to q8_0 (by design, logs a warning).
  Set `TURBO_AUTO_ASYMMETRIC=0` to force turbo4 on both K and V (verified working).
- Use `-st` (single-turn) for scripted runs so llama-cli exits after generating.
