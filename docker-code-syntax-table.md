# Dockerfile Code Syntax

| Dockerfile Syntax | Explanation | Example |
| --- | --- | --- |
| FROM | Specifies the base image for subsequent instructions | `FROM python:3.9-alpine` |
| LABEL | Adds metadata to an image | `LABEL maintainer="John Doe <john.doe@example.com>"` |
| RUN | Executes a command in the container | `RUN pip install -r requirements.txt` |
| CMD | Specifies the default command for the container | `CMD ["python", "app.py"]` |
| EXPOSE | Informs Docker that the container will listen on the specified network ports | `EXPOSE 80` |
| ENV | Sets environment variables | `ENV FLASK_APP=app.py` |
| COPY | Copies files from the host machine to the container | `COPY . /app` |
| ADD | Copies files from the host machine to the container, and also supports downloading files from URLs and extracting compressed archives | `ADD https://example.com/archive.tar.gz /app` |
| WORKDIR | Sets the working directory for subsequent instructions | `WORKDIR /app` |
| ARG | Defines build-time variables that can be passed to the Docker build command | `ARG VERSION=1.0` |
| VOLUME | Creates a mount point for external volumes | `VOLUME /data` |
