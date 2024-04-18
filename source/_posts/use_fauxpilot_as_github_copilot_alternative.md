---
title: Use Fauxpilot as an alternative to GitHub Copilot for offline code assistance in VSCode and Vim.
date: 2024-04-17 15:18:15
tags: ai tools
---

### Why Use Fauxpilot?
[Fauxpilot](https://github.com/fauxpilot/fauxpilot) is an open-source alternative to GitHub Copilot, leveraging the power of large-scale models to deliver a range of AI-driven functionalities, including dialogue and code suggestions. In this article, we'll focus exclusively on its code suggestion capabilities, with other features to be detailed in future posts.

### Local Setup
To run Fauxpilot, the following prerequisites must be met:
- A Windows or Linux system with an NVIDIA graphics card.
- At least 50GB of local storage space (required for the large model).
- Docker and [nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit).
- The `curl` and `zstd` command-line tools, necessary for model downloads.

I am using Ubuntu.

The Docker installation is simple and won't be elaborated here. The NVIDIA Container Toolkit is a project that enables containers to utilize NVIDIA GPU resources. The following commands can be used for installation:

```
# Configure the production repository:
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Configure the repository to use experimental packages: 
sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Update and install:
sudo apt-get update && apt-get install -y nvidia-container-toolkit

# Configure the container runtime to use NVIDIA GPUs:
sudo nvidia-ctk runtime configure --runtime=docker
```

Then, simply restart Docker:
```
sudo systemctl restart docker
```
Installation methods for other systems can be found in the complete [installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

Next, enter the project root directory and run the startup script directly:
```
cd fauxpilot
./setup.sh
```
<!--more-->
![fauxpilot_setup](/images/fauxpilot_setup.jpg)

After that, you can use `launch.sh` and `shutdown.sh` to start and stop the service.

When launching, you will be presented with some parameters to choose from; pressing Enter to maintain the defaults is fine. My graphics card has 16GB of VRAM, and the `codegen-6B-multi` model was selected by default, which requires 13GB of VRAM. However, after actual testing, this led to VRAM overflow, and switching to the `codegen-2B-multi` model was more stable. Everyone should choose according to their hardware situation. The [official GPU support matrix](https://github.com/fauxpilot/fauxpilot/wiki/GPU-Support-Matrix) also lists the supported models.

Then you just wait for the model to finish downloading and it will start automatically. The first launch might be a bit slow.
Once you see the graphics card's performance metrics being printed, like the following information, it's ready to use:
```
triton-1         |
triton-1         | I0410 09:51:23.359229 90 grpc_server.cc:4587] Started GRPCInferenceService at 0.0.0.0:8001
triton-1         | I0410 09:51:23.360050 90 http_server.cc:3303] Started HTTPService at 0.0.0.0:8000
triton-1         | I0410 09:51:23.464509 90 http_server.cc:178] Started Metrics Service at 0.0.0.0:8002
```

At this point, we can use the command line to test if it's successful:
```
curl --location 'http://localhost:5000/v1/engines/codegen/completions' \
--header 'Accept: application/json' \
--header 'Content-type: application/json' \
--data '{"prompt":"def hello","max_tokens":100,"temperature":0.1,"stop":["\n\n"]}'
```
If you see prompt content being output, it's successful.
![fauxpilot_postman](/images/fauxpilot_postman.jpg)

### Configuring the Editor

The configuration for VSCODE is quite straightforward; we just need to switch to our address in the plugin settings. Open `setting.json` and edit the following content:
```
    "github.copilot.advanced": {
        "debug.overrideEngine": "codegen",
        "debug.testOverrideProxyUrl": "http://localhost:5000",
        "debug.overrideProxyUrl": "http://localhost:5000"
    }
```

The configuration for VIM is a bit more complex. The copilot.vim official site does not provide an API address override parameter, and the `g:copilot_proxy` parameter is just a regular request proxy feature. Therefore, we need to modify its API call address. Locate the installation directory of your `copilot.vim` plugin and then run the following commands to replace it:
```
sed 's|https://api.github.com/copilot_internal|http://127.0.0.1:5000/copilot_internal|g' -i copilot.vim/copilot/dist/agent.js
sed 's|https://copilot-proxy.githubusercontent.com|http://127.0.0.1:5000|g' -i copilot.vim/copilot/dist/agent.js
```
After restarting, you can see the effect.

### Conclusion
In practice, the speed in simple projects is similar to the official version, but in larger projects (over 500 files), the speed is slower, which is probably related to the model's precision. Those with better hardware conditions can choose better graphics card resources for processing. It is also a convenient local alternative solution. Compared to the official version, which costs $20 per month, this is a more cost-effective option for friends with graphics cards.

