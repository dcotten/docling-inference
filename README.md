# Docling inference server

This project provides a FastAPI wrapper around the
[docling](https://github.com/DS4SD/docling) document parser to make it easier to
use in distributed production environments.

## Running

The easiest way to run this project is using docker. There are two image families,
one for cuda machines and one for cpu:

- Cuda: ghcr.io/aidotse/docling-inference:rev
- CPU: ghcr.io/aidotse/docling-inference:cpu-rev

```bash
# Create volumes to not have to download models every time
docker volume create hf_cache
docker volume create ocr_cache

# Run the container
docker run -d \
  --gpus all \
  -p 8080:8080 \
  -e NUM_WORKERS=8 \
  -v hf_cache:/root/.cache/huggingface \
  -v ocr_cache:/root/.EasyOCR \
  ghcr.io/aidotse/docling-inference:latest
```

### Docker compose

```yaml
services:
  docling-inference:
    image: ghcr.io/aidotse/docling-inference:latest
    ports:
      - 8080:8080
    environment:
      - NUM_WORKERS=8
    volumes:
      - hf_cache:/root/.cache/huggingface
      - ocr_cache:/root/.EasyOCR
    restart: always
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

volumes:
  hf_cache:
  ocr_cache:
```

### Local python

Dependencies are handled with [uv](https://docs.astral.sh/uv/) in this
project. Follow their installation instructions if you do not have it.

```bash
# Create a virtual environment
uv venv

# Install the dependencies
uv sync --extra cpu
# OR if you have cuda devices
uv sync --extra cu121

# Activate the shell
source .venv/bin/activate

# Start the server
python src/main.py
```

## Using the API

When the server is started you can find the interactive API documentation at the `/docs`
endpoint. If you're running locally with the example command, this will be
`http://localhost:8080/docs`.

There are two main methods to parse documents that take the data on two different formats.
You can use the `/parse/url` to parse a document from a download link. To call it with
`curl` from the command line:

```sh
curl -X 'POST' \
  'http://localhost:8080/parse/url' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "include_json": false,
  "output_format": "markdown",
  "url": "https://arxiv.org/pdf/2408.09869"
}'
```

You can also parse files directly with the `/parse/file` endpoint:

```sh
curl -X 'POST' \
  'http://localhost:8080/parse/file' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@file-path.pdf;type=application/pdf' \
  -F 'data={"include_json":false,"output_format":"markdown"}'
```

Tip: You can use a service like https://curlconverter.com/ to convert curl commands to
your favourite http client, e.g. `requests`.

For a full list of available options, please refer to the interactive documentation.

## Building

Build the project docker image with one of the following commands

- Cuda: `docker build -t ghcr.io/aidotse/docling-inference:dev .`
- CPU: `docker build -f Dockerfile.cpu -t ghcr.io/aidotse/docling-inference:dev .`

## Configuration

Configuration is handled through environment variables. Here is a list of the
available configuration variables. They are defined in `src/config.py`

- `NUM_WORKERS`: The number of processes to run.
- `LOG_LEVEL`: The lowest level of logs to display. One of DEBUG, INFO, WARNING,
  CRITICAL, ERROR.
- `DEV_MODE`: Sets automatic reload of the service. Useful during development
- `PORT`: The port to run the server on.
- `AUTH_TOKEN`: Token to use for authentication. Token is expected in the
  `Authorization: Bearer: <token>` format in the request header. The service is
  unprotected if this option is omitted.
- `OCR_LANGUAGES`: List of language codes to use for optical character optimization.
  Default is `"es,en,fr,de,sv"`. See https://www.jaided.ai/easyocr/ for the list
  of all available languages.
- `DO_CODE_ENRICHMENT`: Use a code enrichment model in the pipeline. Processes images of code to code.
- `DO_FORMULA_ENRICHMENT`: Use a formula enrichment model in the pipeline. Converts formulas to LaTeX.
- `DO_PICTURE_CLASSIFICATION`: Use a picture classification model in the pipelinese. Classifies the type of image into a category.
- `DO_PICTURE_DESCRIPTION`: Use a picture description model in the pipeline. Uses a small multimodal model to describe images.
