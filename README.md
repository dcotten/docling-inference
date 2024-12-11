# Docling inference server

This project provides a FastAPI wrapper around the [docling](https://github.com/DS4SD/docling)
document parser to make it easier to use in distributed production environments.

## Running

The easiest way to run this project is using docker

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
  docling-inference
```

### Docker compose

```yaml
services:
  docling-inference:
    image: docling-inference
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

Dependencies are handled with [pypoetry](https://python-poetry.org/) in this project.
Follow their installation instructions if you do not have it.

```bash
# Install the dependencies
poetry install

# Activate the shell
poetry shell

# Start the server
python src/main.py
```

## Building

Build the project docker image with `docker build -t docling-inference:dev .`

## Configuration

Configuration is handled through environment variables. Here is a list of the available
configuration variables. They are defined in `src/config.py`

- `NUM_WORKERS`: The number of processes to run
- `LOG_LEVEL`: The lowest level of logs to display. One of DEBUG, INFO, WARNING, CRITICAL, ERROR
- `DEV_MODE`: Sets automatic reload of the service. Useful during development
- `PORT`: The port to run the server on
