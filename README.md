# Docker-WhisperX (Flask API Edition)

This repository is a fork of the original [jim60105/docker-whisperX](https://github.com/jim60105/docker-whisperX) project, restructured as a **Web API** microservice.

The CLI-based structure of the original project has been modified to enable the WhisperX model to respond to HTTP requests via a **Flask** server. This allows you to easily use the model as a standalone service in projects such as real-time audio analysis, speech-to-text, or speaker diarization.

## Key Changes Made

* **Flask API Integration:** A new `main.py` file has been added to the repository. This file hosts a web server that listens for `POST` requests at the root (`/`) directory and returns Turkish (`tr`) transcriptions using the `medium` sized model.
* **Dockerfile Updates:**
  * The `flask` dependency required for the API to function was added to the build stage.
  * An `EXPOSE 5000` port instruction was added to the Dockerfile to allow communication with the outside world.
  * The `ENTRYPOINT` argument was updated to `[ "dumb-init", "--", "python", "/app/main.py" ]` so that the Flask server starts automatically when the container runs, instead of waiting for a CLI command.

## How to Run

### 1. Building the Docker Image
After cloning the project, you can build the image locally:

```bash
docker build --build-arg WHISPER_MODEL=medium -t whisperx-api:latest .
```

### 2. Starting the Container
Run the image you created by exposing port 5000 and enabling GPU support:

```bash
docker run --gpus all -p 5000:5000 whisperx-api:latest
```

*Note: The WhisperX model (loaded as `medium` - `float16` on `cuda`) is loaded into RAM when the API starts. This ensures high-speed responses starting from the very first request.*

### 3. Sending Requests to the API
The API expects *raw audio* data sent via the `POST` method. The incoming data is converted from `int16` to `float32` within the system before being fed into the model.

**Example request via Python:**

```python
import requests

# Reading the raw audio file to be processed
with open('audio_sample.raw', 'rb') as f:
    audio_bytes = f.read()

# Sending a POST request to the Flask API
url = "http://localhost:5000/"
response = requests.post(url, data=audio_bytes)

if response.status_code == 200:
    print("Transcription Result:\n", response.json())
else:
    print(f"Error ({response.status_code}):", response.text)
```

## Configuration Notes
In the current `main.py` configuration, the following parameters are hardcoded:
* **Model:** `medium`
* **Device / Precision:** `cuda` / `float16`
* **Language:** `tr` (Turkish)
* **Batch Size:** `16`

You can update these parameters within the `main.py` file according to your needs.

---
*License and Original Project Details: Since this project is based on the original docker-whisperX project, the core MIT and BSD-4 license requirements are maintained.*