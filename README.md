# AI-Stack

This project sets up an AI stack at home for various machine learning and AI tasks, including Open Web UI, Ollama, Stable Diffusion, and Whisper services. The stack leverages Docker for containerization and Tailscale for secure access.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Folder Structure](#folder-structure)
3. [Setup](#setup)
4. [Services](#services)
5. [Usage](#usage)
6. [Environment Variables](#environment-variables)
7. [Ports](#ports)
8. [Troubleshooting](#troubleshooting)

## Prerequisites

Before you begin, ensure you have:

- [Docker](https://docs.docker.com/get-docker/) installed.
- [Docker Compose](https://docs.docker.com/compose/install/) installed.
- A Tailscale account and generated Auth key.

## Folder Structure

The folder structure of the project is as follows:

```
ai-stack
|── config
|   └── open-webui.json
|── state
├── .env
├── docker-compose.yaml
├── ollama
├── open-webui
├── stable-diffusion-webui-docker
└── whisper
```

- `.env`: Environment variables file.
- `docker-compose.yaml`: Docker Compose configuration file.
- `ollama`, `open-webui`, `stable-diffusion-webui-docker`, `whisper`: Directories for respective services and configurations.

## Setup

### 1. Clone the Repository

```bash
git clone <repository-url>
cd ai-stack
```

### 2. Configure Environment Variables

Create a `.env` file in the root directory if it does not exist already:

```bash
touch .env
```

Populate the `.env` file with the necessary environment variables:

```
PUID=1000
PGID=1000
BRAVE_SEARCH_API_KEY=your-brave-api-key
DB_USER=your-db-user
DB_PASS=your-db-pass
TS_AUTHKEY=your-tailscale-auth-key
WHISHPER_HOST=your-whisper-host
LT_LOAD_ONLY=en,fr,es
```

### 3. Changes for ComfyUI

Before starting the stack, in the `ai-stack` directory, you’ll want to clone the repo (or just copy the necessary files).
(this will create the folder for you)

```bash
git clone https://github.com/AbdBarho/stable-diffusion-webui-docker.git
```

After cloning you'll want to make a change to the Dockerfile

```bash
nano stable-diffusion-webui-docker/services.comfy/Dockerfile
```

I commented out the pinning to commit hash and just grabbed the latest comfy

```bash
FROM pytorch/pytorch:2.3.0-cuda12.1-cudnn8-runtime

ENV DEBIAN_FRONTEND=noninteractive PIP_PREFER_BINARY=1

RUN apt-get update && apt-get install -y git && apt-get clean

ENV ROOT=/stable-diffusion
RUN --mount=type=cache,target=/root/.cache/pip \
  git clone https://github.com/comfyanonymous/ComfyUI.git ${ROOT} && \
  cd ${ROOT} && \
  git checkout master && \
#  git reset --hard 276f8fce9f5a80b500947fb5745a4dde9e84622d && \
  pip install -r requirements.txt

WORKDIR ${ROOT}
COPY . /docker/
RUN chmod u+x /docker/entrypoint.sh && cp /docker/extra_model_paths.yaml ${ROOT}

ENV NVIDIA_VISIBLE_DEVICES=all PYTHONPATH="${PYTHONPATH}:${PWD}" CLI_ARGS=""
EXPOSE 7860
ENTRYPOINT ["/docker/entrypoint.sh"]
CMD python -u main.py --listen --port 7860 ${CLI_ARGS}
```


## Downloading Models[](https://technotim.live/posts/ai-stack-tutorial/#downloading-models)

You’ll want to grab any models you like from [HuggingFace](https://huggingface.co/). I am using [stabilityai/stable-diffusion-3-medium](https://huggingface.co/stabilityai/stable-diffusion-3-medium)

You’ll want to download all of the models and then transfer them to your server and put them in the appropriate folders

Models will need to bt placed in the `Stable-diffusion` folder.

```bash
stable-diffusion-webui-docker/data/models/Stable-diffusion
```

Models are any file in the root of `stable-diffusion-3-medium` that have the extension `*.safetensors`

For clips, you’ll need to create this folder (because it doesn’t exist)

```bash
mkdir stable-diffusion-webui-docker/data/models/CLIPEncoder
```

## Example Workflows for ComfyUI and Stable Diffusion 3 Medium[](https://technotim.live/posts/ai-stack-tutorial/#example-workflows-for-comfyui-and-stable-diffusion-3-medium)

You’ll need to download the same workflows to the machine that accesses ComfyUI so you can import them into the browser.

Example workflows are also available on HuggingFace in the [Stable Diffusion 3 Medium repo](https://huggingface.co/stabilityai/stable-diffusion-3-medium/tree/main/comfy_example_workflows)

This should show up as a service on your tailnet port `7860`

### 4. Run Docker Compose

```bash
docker compose up -d --build --force-recreate --remove-orphans
```

This will start all services defined in the `docker-compose.yaml` file.

## Services

### 1. Tailscale - `ts-open-webui`

Provides a secure VPN connection to access services.

### 2. Ollama - `ollama`

A general-purpose AI service.

### 3. Open Web UI - `open-webui`

A user interface to interact with various AI-related tasks.

### 4. Stable Diffusion Web UI - `stable-diffusion-webui`

Provides image generation capabilities.

### 5. MongoDB - `mongo`

Database service for Whisper AI.

### 6. LibreTranslate - `translate`

Translation service for Whisper.

### 7. Whisper AI - `whisper`

An AI service for speech-to-text, translation, etc.

## Usage

1. **Accessing Services**

   Services are accessed securely through Tailscale. Make sure your device is connected to the Tailscale network with the appropriate tags.

2. **Interacting with the AI Stack**

   Each service exposed will be available through their respective endpoints. Check the output of `docker-compose up -d` to see which ports are forwarded through Tailscale.

## Environment Variables

Refer to the `.env` file for configuring environment-specific variables:

- `PUID`: User ID for permissions.
- `PGID`: Group ID for permissions.
- `BRAVE_SEARCH_API_KEY`: API key for Brave web search.
- `DB_USER`: Database username for MongoDB.
- `DB_PASS`: Database password for MongoDB.
- `TS_AUTHKEY`: Auth key for Tailscale.
- `WHISHPER_HOST`: Host configuration for Whisper.
- `LT_LOAD_ONLY`: Languages to load in LibreTranslate.

## Ports

The services are networked through Tailscale and do not expose ports directly to your local machine. Ensure your Tailscale configuration allows access to these services.

## Troubleshooting

1. **Logs**

   Check logs for each service to debug issues:

   ```bash
   docker logs <container_name>
   ```

2. **Connectivity Issues**

   Ensure your device is properly connected to the Tailscale network.

3. **Docker Compose**

   If you encounter any issues starting the containers, you can bring down the stack and bring it up again:

   ```bash
   docker-compose down
   docker-compose up -d
   ```

## Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Tailscale Documentation](https://tailscale.com/kb/)
- [TechnoTim](https://technotim.live/posts/ai-stack-tutorial/#basic-auth-hash)
