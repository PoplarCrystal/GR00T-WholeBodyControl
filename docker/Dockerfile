ARG BASE_IMAGE=nvcr.io/nvidia/isaac-lab:3.0.0
FROM ${BASE_IMAGE}

USER root
SHELL ["/bin/bash", "-euo", "pipefail", "-c"]

WORKDIR /workspace/sonic
COPY gear_sonic /workspace/sonic/gear_sonic

# Keep the simulation stack and CUDA PyTorch supplied by Isaac Lab.
RUN mkdir -p /opt/sonic \
    && "${VIRTUAL_ENV}/bin/python" -c \
        'from importlib.metadata import version; from pathlib import Path; Path("/opt/sonic/base-constraints.txt").write_text("".join(f"{p}=={version(p)}\n" for p in ("torch", "torchvision", "numpy", "newton", "warp-lang", "mujoco", "mujoco-warp")))' \
    && uv pip install --python "${VIRTUAL_ENV}/bin/python" --no-cache \
        --constraint /opt/sonic/base-constraints.txt \
        -e '/workspace/sonic/gear_sonic[training]' \
    && "${VIRTUAL_ENV}/bin/python" -c \
        'import sys, torch, numpy, scipy, newton, warp, mujoco, transformers, trl; assert sys.version_info[:2] == (3, 12); print("SONIC runtime:", sys.version, torch.__version__, numpy.__version__, scipy.__version__, newton.__version__, warp.__version__, mujoco.__version__, transformers.__version__, trl.__version__)'

ENV PATH="${VIRTUAL_ENV}/bin:${PATH}" \
    OMNI_KIT_ALLOW_ROOT=1 \
    PYTHONUNBUFFERED=1

HEALTHCHECK NONE
ENTRYPOINT ["/bin/bash"]
