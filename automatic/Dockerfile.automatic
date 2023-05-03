FROM nvidia/cuda:11.8.0-base-ubuntu22.04
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update -y && apt-get upgrade -y && apt-get install -y libgl1 libglib2.0-0 wget git git-lfs python3-pip python-is-python3 && rm -rf /var/lib/apt/lists/*

RUN adduser --disabled-password --gecos '' user
RUN mkdir /content && chown -R user:user /content
WORKDIR /content
USER user

RUN pip3 install --upgrade pip
RUN pip uninstall triton
RUN pip install --pre torch torchvision torchtriton --extra-index-url https://download.pytorch.org/whl/nightly/cu118 --force
RUN pip install numexpr
RUN pip install ninja setuptools


RUN git clone -b master https://github.com/AUTOMATIC1111/stable-diffusion-webui
RUN git clone https://github.com/Mikubill/sd-webui-controlnet /content/stable-diffusion-webui/extensions/sd-webui-controlnet

RUN cd stable-diffusion-webui && git reset --hard

ADD --chown=user https://huggingface.co/ckpt/ControlNet/resolve/main/control_canny-fp16.safetensors /content/stable-diffusion-webui/extensions/sd-webui-controlnet/models/control_canny-fp16.safetensors

RUN sed -i -e '''/    prepare_environment()/a\    os.system\(f\"""sed -i -e ''\"s/dict()))/dict())).cuda()/g\"'' /content/stable-diffusion-webui/repositories/stable-diffusion-stability-ai/ldm/util.py""")''' /content/stable-diffusion-webui/launch.py
RUN sed -i -e 's/fastapi==0.90.1/fastapi==0.89.1/g' /content/stable-diffusion-webui/requirements_versions.txt
RUN sed -i -e 's/    start()/    #start()/g' /content/stable-diffusion-webui/launch.py
RUN cd stable-diffusion-webui && python launch.py --skip-torch-cuda-test

ADD --chown=user https://civitai.com/api/download/models/29460 /content/stable-diffusion-webui/models/Stable-diffusion/realisticvision2.safetensors

EXPOSE 7861
CMD  cd /content/stable-diffusion-webui && python webui.py --nowebui --api --listen
