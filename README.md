# How to set up flash-attention for AMD GPUs on ComfyUI
Full credit goes to https://gist.github.com/alexheretic/d868b340d1cef8664e1b4226fd17e0d0

## Pre-requisites
- MI200x, MI250x, MI300x, MI355x, or RDNA 3/4 GPU https://rocm.docs.amd.com/en/latest/reference/gpu-arch-specs.html
- ROCm 6.0 and above
- uv python environment manager (optional) https://docs.astral.sh/uv/

## Steps
### Clone ComfyUI and create a python environment:
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
### Install flash-attention
```
git clone https://github.com/Dao-AILab/flash-attention
cd flash-attention
FLASH_ATTENTION_TRITON_AMD_ENABLE="TRUE" pip install --no-build-isolation .
```
### Finish installing ComfyUI requirements
```
uv pip install -r requirements.txt
```
