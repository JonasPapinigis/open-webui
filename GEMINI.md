# Open WebUI Configuration and Execution Summary

This document outlines the various ways to run and configure the Open WebUI project, based on an analysis of the project's files.

## Execution Methods

There are several ways to run the Open WebUI application, catering to different environments and use cases.

### 1. Docker (Recommended)

The primary and recommended method for running Open WebUI is through Docker. The `README.md` provides several Docker run commands for different scenarios:

*   **Default (Ollama on the same machine):**
    ```bash
    docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
    ```
*   **Ollama on a different server:**
    ```bash
    docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=https://example.com -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
    ```
*   **With Nvidia GPU support:**
    ```bash
    docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda
    ```
*   **Bundled with Ollama:**
    ```bash
    # For GPU
    docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama

    # For CPU
    docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
    ```

### 2. Docker Compose

The `docker-compose.yaml` file provides a convenient way to run the application with both the `open-webui` and `ollama` services.

```bash
docker-compose up -d
```

### 3. Kubernetes (Kustomize & Helm)

The `INSTALLATION.md` file provides instructions for deploying the application to a Kubernetes cluster using either Kustomize or Helm.

*   **Kustomize:**
    ```bash
    # For CPU
    kubectl apply -f ./kubernetes/manifest/base

    # For GPU
    kubectl apply -k ./kubernetes/manifest
    ```
*   **Helm:**
    ```bash
    # Package the Helm chart
    helm package ./kubernetes/helm/

    # For CPU
    helm install ollama-webui ./ollama-webui-*.tgz

    # For GPU
    helm install ollama-webui ./ollama-webui-*.tgz --set ollama.resources.limits.nvidia.com/gpu="1"
    ```

### 4. Python `pip`

The application can be installed and run as a Python package using `pip`.

```bash
pip install open-webui
open-webui serve
```

### 5. Local Development

For local development, the `package.json` file defines a `dev` script that starts a Vite development server.

```bash
npm run dev
```

The backend can be started separately using the `backend/start.sh` script.

## Configuration

The application can be configured through a combination of environment variables and a `config.py` file.

### Environment Variables

The `Dockerfile` and `docker-compose.yaml` files reveal a number of environment variables that can be used to configure the application. Some of the key variables include:

*   `OLLAMA_BASE_URL`: The URL of the Ollama API.
*   `OPENAI_API_KEY`: Your OpenAI API key.
*   `OPENAI_API_BASE_URL`: The base URL for the OpenAI API.
*   `WEBUI_SECRET_KEY`: A secret key for signing JWTs.
*   `OLLAMA_DOCKER_TAG`: The Docker tag for the Ollama image.
*   `WEBUI_DOCKER_TAG`: The Docker tag for the Open WebUI image.
*   `OPEN_WEBUI_PORT`: The port to expose for the Open WebUI container.
*   `CORS_ALLOW_ORIGIN`: A semicolon-separated list of allowed CORS origins.

### Configuration File

The `backend/open_webui/config.py` file contains a comprehensive set of configuration options. These options can be set using environment variables, and many are also stored in the database and can be modified through the Web UI. The `PersistentConfig` class in this file is responsible for managing the configuration, allowing values to be loaded from environment variables and the database.

Some of the key configuration areas in this file include:

*   **Authentication:** `WEBUI_AUTH`, `ENABLE_SIGNUP`, OAuth settings.
*   **API Endpoints:** `OLLAMA_BASE_URLS`, `OPENAI_API_BASE_URLS`.
*   **UI:** `DEFAULT_MODELS`, `DEFAULT_PROMPT_SUGGESTIONS`.
*   **RAG:** `VECTOR_DB`, `CHROMA_DATA_PATH`, `RAG_EMBEDDING_MODEL`.
*   **Tasks:** Configuration for background tasks like title generation and tag generation.
*   **Code Interpreter:** Settings for the code interpreter feature.

## Separating Frontend and Backend

To deploy the frontend and backend separately, the following approach is recommended:

1.  **Frontend:**
    *   Create a `Dockerfile.frontend` that builds the frontend using `npm run build` and serves the static assets with a web server like Nginx.
    *   Build and deploy the frontend container to a public subnet.

2.  **Backend:**
    *   Create a `Dockerfile.backend` that is a stripped-down version of the original `Dockerfile`, containing only the backend code and dependencies.
    *   Build and deploy the backend container to a private subnet.
    *   Configure the backend to connect to the Ollama nodes in the same private subnet using the `OLLAMA_BASE_URL` environment variable.
