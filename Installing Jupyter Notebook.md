# Installing Jupyter Notebook

1. Install NVIDIA Container Toolkit

   ```
   curl -fsSL https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-runtime-keyring.gpg
   distribution=$(. /etc/os-release;echo $ID$VERSION_ID) 
   echo "deb [signed-by=/usr/share/keyrings/nvidia-container-runtime-keyring.gpg] https://nvidia.github.io/libnvidia-container/stable/$distribution /" | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```

   ```
   sudo apt update
   sudo apt install -y nvidia-container-toolkit
   ```

   then, Restart Docker:

   ```
   sudo systemctl restart docker
   ```
2. Verify GPU Passthrough

   ```
   docker run --rm --gpus all nvidia/cuda:12.2.2-runtime-ubuntu22.04 nvidia-smi
   ```
3. **Docker Compose**

   ```
   version: "3.9"
   
   services:
     jupyter:
       image: jupyter/tensorflow-notebook:latest  # Or use jupyter/pytorch-notebook
       container_name: jupyter-gpu
       runtime: nvidia
       deploy:
         resources:
           reservations:
             devices:
               - driver: nvidia
                 count: all
                 capabilities: [gpu]
       ports:
         - "8888:8888"
       volumes:
         - ./notebooks:/home/jovyan/work
       environment:
         - NVIDIA_VISIBLE_DEVICE
   ```
4. Generate Password

   ```python
   python3 -c "from notebook.auth import passwd; print(passwd())"
   ```

```
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make
```

```
cd ~/work/notebooks/AI\ Train/llama.cpp
python3 -m venv venv
source venv/bin/activate
```

```
python convert_hf_to_gguf.py ~/work/notebooks/ai-train/export/merged-zrah-model --outfile ~/work/notebooks/ai-train/export --outtype f16 --model-name zrah-1.2
```