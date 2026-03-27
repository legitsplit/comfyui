# How to set up flash-attention for AMD GPUs on ComfyUI
Full credit goes to https://gist.github.com/alexheretic/d868b340d1cef8664e1b4226fd17e0d0

## Pre-requisites
- MI200x, MI250x, MI300x, MI355x, or RDNA 3/4 GPU https://rocm.docs.amd.com/en/latest/reference/gpu-arch-specs.html
- ROCm 6.0 and above
- uv python environment manager (optional) https://docs.astral.sh/uv/

## Steps:
### Clone ComfyUI and create a Python environment:
```
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
uv venv venv --python 3.13
source venv/bin/activate
```
### Install Torch (check your gfx version)
```
uv pip install --upgrade pip wheel setuptools
uv pip install --pre torch torchvision torchaudio triton --index-url https://rocm.nightlies.amd.com/v2/gfx120X-all/
```
### Install Flash-Attention
```
git clone https://github.com/Dao-AILab/flash-attention
cd flash-attention
FLASH_ATTENTION_TRITON_AMD_ENABLE="TRUE" pip install --no-build-isolation .
```
### Finish installing ComfyUI requirements
```
cd ..
uv pip install -r requirements.txt
```
### Create a launch script (e.g. with nano)
```
#!/bin/bash
export HIP_VISIBLE_DEVICES=0
export COMFYUI_ENABLE_MIOPEN=1
export MIOPEN_FIND_MODE=FAST
export MIOPEN_ENABLE_CACHE=1

# slower, but more stable / fewer OOMs. No OOMs? Maybe you don't need this.
export PYTORCH_NO_HIP_MEMORY_CACHING=1

# triton
export TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL=1
export FLASH_ATTENTION_TRITON_AMD_ENABLE=TRUE
## Significantly faster attn_fwd performance for wan2.2 workflows
export FLASH_ATTENTION_FWD_TRITON_AMD_CONFIG_JSON='{"BLOCK_M":128,"BLOCK_N":64,"waves_per_eu":1,"PRE_LOAD_V":false,"num_stages":1,"num_warps":8}'

# pytorch switches on NHWC for rocm > 7, causes signifant miopen regressions for upscaling
# todo: fixed now? since what pytorch version?
export PYTORCH_MIOPEN_SUGGEST_NHWC=0

cd ComfyUI/
source venv/bin/activate
python3 main.py --use-flash-attention --disable-pinned-memory
# --use-flash-attention: use faster flash attention installed above.
# --disable-pinned-memory: github.com/Comfy-Org/ComfyUI/issues/11781#issuecomment-3802152655
# --cache-ram 32: optional, helps prevent comfy from using up all 64GB of ram.
```
### Make launch script executable
```
sudo chmod +x script.sh
```
