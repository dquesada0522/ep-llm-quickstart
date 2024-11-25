# Ollama Docker Setup Guide (Windows Edition)

Welcome to the Ollama Docker Setup Guide! This README provides step-by-step instructions to set up and run Ollama using the LLaMA 3.2B model in a Docker container on a Windows system. It covers configuring the model for a larger token context size, integrating the Ollama API with a Python web application, and troubleshooting common issues. Additionally, you'll find an appendix with detailed instructions for installing Docker on various operating systems and a glossary of key terms.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup and Run Ollama in Docker](#setup-and-run-ollama-in-docker)
  - [1. Install Docker](#1-install-docker)
  - [2. Download the LLaMA 3.2B Model](#2-download-the-llama-32b-model)
  - [3. Pull the Ollama Docker Image](#3-pull-the-ollama-docker-image)
  - [4. Running the Docker Container with LLaMA 3.2B](#4-running-the-docker-container-with-llama-32b)
- [Configuring Token Context Size](#configuring-token-context-size)
- [Calling the Ollama API from a Python Web Application](#calling-the-ollama-api-from-a-python-web-application)
  - [Python Code Snippet Using `requests`](#python-code-snippet-using-requests)
  - [Sample PowerShell API Call](#sample-powershell-api-call)
- [Troubleshooting](#troubleshooting)
- [Appendix: Installing Docker](#appendix-installing-docker)
  - [Windows](#windows)
  - [macOS](#macos)
  - [Linux](#linux)
- [Glossary](#glossary)

---

## Prerequisites

Before you begin, ensure you have the following:

- **Docker Installed:** Refer to the [Appendix](#appendix-installing-docker) for installation instructions.
- **Sufficient Hardware Resources:** Running large language models like LLaMA 3.2B requires significant CPU and memory resources.
- **Basic Knowledge of Docker:** Familiarity with container concepts will help, though detailed steps are provided.

---

## Setup and Run Ollama in Docker

### 1. Install Docker

Ensure Docker is installed on your Windows machine. Refer to the [Appendix](#appendix-installing-docker) for detailed installation steps.

### 2. Download the LLaMA 3.2B Model

Download the LLaMA 3.2B model and place it in the `C:\models` directory.

#### Steps:

1. **Create the Models Directory:**

   Open PowerShell with administrative privileges and run:

   ```powershell
   New-Item -ItemType Directory -Path C:\models
   ```

2. **Download the Model:**

   Replace `<MODEL_DOWNLOAD_URL>` with the actual URL to download the LLaMA 3.2B model.

   ```powershell
   Invoke-WebRequest -Uri "<MODEL_DOWNLOAD_URL>" -OutFile "C:\models\llama-3.2B.zip"
   ```

3. **Extract the Model Files:**

   Ensure you have a tool like [7-Zip](https://www.7-zip.org/) installed. Then, extract the downloaded ZIP file:

   ```powershell
   Add-Type -AssemblyName System.IO.Compression.FileSystem
   [System.IO.Compression.ZipFile]::ExtractToDirectory("C:\models\llama-3.2B.zip", "C:\models\llama-3.2B")
   ```

   **Note:** Ensure the extracted model files are directly within `C:\models\llama-3.2B`.

### 3. Pull the Ollama Docker Image

Open PowerShell and execute the following command to pull the latest Ollama Docker image:

```powershell
docker pull ollama/ollama:latest
```

### 4. Running the Docker Container with LLaMA 3.2B

Run the Docker container with the LLaMA 3.2B model. The model files located in `C:\models\llama-3.2B` will be mounted into the container.

```powershell
docker run -d `
  --name ollama `
  -p 11434:11434 `
  -v C:\models\llama-3.2B:/models/llama-3.2B `
  ollama/ollama:latest `
  --model llama-3.2B `
  --context-size 10000
```

**Parameters Explained:**

- `-d`: Runs the container in detached mode.
- `--name ollama`: Names the container "ollama" for easier reference.
- `-p 11434:11434`: Maps port 11434 of the container to port 11434 on the host machine.
- `-v C:\models\llama-3.2B:/models/llama-3.2B`: Mounts the host directory containing the model to the container.
- `--model llama-3.2B`: Specifies the model to use.
- `--context-size 10000`: Sets the token context size to 10,000.

---

## Configuring Token Context Size

To configure the model to accept a token context size of at least 10,000, ensure you include the `--context-size` flag when running the Docker container as shown above.

If you need to adjust this setting after the container is running, you can update the Docker run command or modify the container's configuration files as necessary. Restart the container after making changes to apply the new settings.

**Restarting the Container:**

```powershell
docker stop ollama
docker rm ollama
# Re-run the docker run command with updated parameters if needed
```

---

## Calling the Ollama API from a Python Web Application

Integrate the Ollama API with your local Python web application to perform inference tasks. Below are step-by-step guides along with code snippets using the `requests` library and a sample PowerShell API call.

### Python Code Snippet Using `requests`

```python
import requests
import json

# Define the API endpoint
API_URL = "http://localhost:11434/v1/models/llama-3.2B/infer"

# Define the headers
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_API_TOKEN"  # Replace with your actual token if required
}

# Define the payload
payload = {
    "prompt": "Once upon a time",
    "max_tokens": 100,
    "temperature": 0.7
}

try:
    response = requests.post(API_URL, headers=headers, data=json.dumps(payload))
    response.raise_for_status()  # Raises an HTTPError if the response was an unsuccessful status code
    result = response.json()
    print("Inference Result:", result.get("text"))
except requests.exceptions.HTTPError as http_err:
    print(f"HTTP error occurred: {http_err}")  # Handle HTTP errors
except Exception as err:
    print(f"An error occurred: {err}")  # Handle other possible errors
```

**Explanation:**

1. **API_URL:** Ensure the URL matches your Docker container's exposed port and model endpoint.
2. **Headers:** If your API requires authentication, include the `Authorization` header with your token.
3. **Payload:** Customize the `prompt`, `max_tokens`, and other parameters as needed.
4. **Error Handling:** The try-except blocks handle potential errors gracefully.

**Usage:**

Ensure your Docker container is running and the Ollama API is accessible at `http://localhost:11434`. Run your Python application, and it should successfully communicate with the Ollama API to perform inference.

### Sample PowerShell API Call

You can also perform an API call using PowerShell. Here's a sample script:

```powershell
# Define the API endpoint
$apiUrl = "http://localhost:11434/v1/models/llama-3.2B/infer"

# Define the headers
$headers = @{
    "Content-Type"  = "application/json"
    "Authorization" = "Bearer YOUR_API_TOKEN"  # Replace with your actual token if required
}

# Define the payload
$payload = @{
    "prompt"      = "Once upon a time"
    "max_tokens"  = 100
    "temperature" = 0.7
} | ConvertTo-Json

try {
    $response = Invoke-RestMethod -Uri $apiUrl -Method Post -Headers $headers -Body $payload
    Write-Output "Inference Result: $($response.text)"
}
catch {
    Write-Error "An error occurred: $_"
}
```

**Explanation:**

1. **$apiUrl:** Ensure the URL matches your Docker container's exposed port and model endpoint.
2. **$headers:** If your API requires authentication, include the `Authorization` header with your token.
3. **$payload:** Customize the `prompt`, `max_tokens`, and other parameters as needed.
4. **Error Handling:** The try-catch block handles potential errors gracefully.

**Usage:**

Run the PowerShell script in your PowerShell terminal to communicate with the Ollama API and perform inference.

---

## Troubleshooting

Encountering issues during setup or operation? Below are common problems and their solutions.

### Model Loading Errors

- **Symptom:** Container logs show errors related to model loading or missing files.
- **Solution:**
  - Verify that the model files are correctly placed in the `C:\models\llama-3.2B` directory.
  - Ensure that the Docker volume is correctly mounted by checking the `-v` flag in the Docker run command.
  - Check that the model name specified in the Docker run command matches the actual model.

### API Connectivity Issues

- **Symptom:** Unable to connect to the Ollama API from your application.
- **Solution:**
  - Ensure the Docker container is running: `docker ps`
  - Verify that port `11434` is open and not blocked by a firewall.
  - Check the API URL and ensure it matches the Docker container's exposed port.
  - Review container logs for any API-related errors:

    ```powershell
    docker logs ollama
    ```

### Insufficient Resources

- **Symptom:** The container fails to start or crashes due to resource constraints.
- **Solution:**
  - Allocate more memory or CPU resources to Docker via Docker Desktop settings.
  - Ensure your host machine meets the minimum hardware requirements for running the LLaMA 3.2B model.

### Token Context Size Configuration Not Applied

- **Symptom:** The model does not accept the configured token context size.
- **Solution:**
  - Double-check the `--context-size` flag in your Docker run command.
  - Restart the Docker container after making configuration changes:

    ```powershell
    docker stop ollama
    docker rm ollama
    # Re-run the docker run command with updated parameters if needed
    ```

  - Consult the Ollama documentation to ensure compatibility with the specified context size.

### General Tips

- **Check Logs:** Use `docker logs ollama` to view real-time logs for debugging.
- **Update Docker Image:** Ensure you are using the latest version of the Ollama Docker image:

  ```powershell
  docker pull ollama/ollama:latest
  ```

- **Consult Documentation:** Refer to the official Ollama documentation for advanced configurations and updates.

---

## Appendix: Installing Docker

Follow the instructions below to install Docker on your operating system.

### Windows

1. **Download Docker Desktop:**
   - Visit the [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop) page.
   - Click on "Download for Windows."

2. **Install Docker Desktop:**
   - Run the downloaded installer.
   - Follow the on-screen instructions.
   - Ensure that the **WSL 2** feature is enabled during installation.

3. **Start Docker Desktop:**
   - After installation, launch Docker Desktop from the Start menu.
   - Complete any initial setup prompts.

4. **Verify Installation:**
   - Open PowerShell.
   - Run the following command to verify the installation:

     ```powershell
     docker --version
     ```

### macOS

1. **Download Docker Desktop:**
   - Visit the [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop) page.
   - Click on "Download for Mac."

2. **Install Docker Desktop:**
   - Open the downloaded `.dmg` file.
   - Drag and drop the Docker icon into the Applications folder.

3. **Start Docker Desktop:**
   - Launch Docker from the Applications folder.
   - Follow any on-screen setup instructions.

4. **Verify Installation:**
   - Open Terminal.
   - Run the following command to verify the installation:

     ```bash
     docker --version
     ```

### Linux

Docker installation steps can vary based on the Linux distribution. Below are general steps for Ubuntu. For other distributions, refer to the [official Docker documentation](https://docs.docker.com/engine/install/).

1. **Update Package Index:**

   ```bash
   sudo apt-get update
   ```

2. **Install Prerequisite Packages:**

   ```bash
   sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
   ```

3. **Add Docker’s Official GPG Key:**

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. **Set Up the Docker Repository:**

   ```bash
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. **Install Docker Engine:**

   ```bash
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

6. **Verify Installation:**

   ```bash
   docker --version
   ```

7. **(Optional) Manage Docker as a Non-Root User:**

   ```bash
   sudo usermod -aG docker $USER
   ```

   - Log out and log back in for the changes to take effect.

---

## Glossary

| Term                               | Definition                                                                                                                                               |
|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Inference**                      | The process of using a trained machine learning model to make predictions or generate outputs based on input data.                                       |
| **Token**                          | A basic unit of text (such as a word or subword) that a language model processes.                                                                         |
| **Context Size**                   | The maximum number of tokens a model can consider in a single input. Larger context sizes allow the model to handle more extended inputs or conversations. |
| **API (Application Programming Interface)** | A set of protocols and tools that allow different software applications to communicate with each other.                                                    |
| **Docker Volume**                  | A mechanism for persisting data generated by and used by Docker containers.                                                                                |
| **Container**                      | A lightweight, standalone, and executable package that includes everything needed to run a piece of software, including code, runtime, system tools, and libraries. |
| **Detached Mode**                  | Running a Docker container in the background, allowing the terminal to be free for other commands.                                                       |
| **LLaMA (Large Language Model Meta AI)** | A family of large language models developed by Meta (Facebook) for various natural language processing tasks.                                                |
| **API Token**                      | A unique identifier used to authenticate requests associated with your project for usage and billing purposes.                                            |
| **Endpoint**                       | A specific URL or URI where an API can be accessed by a client application.                                                                                 |
| **Temperature**                    | A parameter that controls the randomness of the model's output. Higher values lead to more diverse outputs, while lower values make the output more deterministic. |

---

Thank you for using this guide to set up Ollama with Docker on Windows! If you encounter issues not covered in the troubleshooting section, consider reaching out to the Ollama community or consulting the official documentation for further assistance.