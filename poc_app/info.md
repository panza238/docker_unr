# App information

## Getting started with FastAPI
In the [first steps tutorial](https://fastapi.tiangolo.com/tutorial/first-steps/) found in the FastAPI documentation, one ca find a minimal example of a FastAPI application. This example is used as a starting point for the PoC application.

To start the application, run the following command from the `poc_app` directory:
```zsh
poetry run uvicorn src.main:app --reload
```

## Containerizing the API
The application is containerized using Docker. The Dockerfile is located in the root of the app's directory (`poc_app/Dockerfile`). Once we run the app inside a container it basically runs as a service.

### Dependecies and requirements
To avoid installing `poetry` inside the container, we will recourse to the beloved `requirements.txt` file. Since we are managing the project with `poetry`, we need to generate the `requirements.txt` file from the `poetry.lock` file. To do so, run the following command from the `poc_app` directory:
```zsh
poetry export --format requirements.txt --output requirements.txt --without-hashes
```

### Generating the Dockerfile
The Dockerfile is pretty self-descriptive. It is based on the `python:3.10` image and installs the dependencies from the `requirements.txt` file. It also copies the source code into the container and exposes the port `8000` (the port where the API is running).
The copying process follows Docker's convention of `<host> - <container>`, meaning that `COPY ./host_dicrectory ./container_directory` will copy the contents of `host_directory` in the host into `container_directory` inside the container. 
One thing to point out is the command ran when the container is started: `CMD ["uvicorn", "src.main:app", "--port", "8000", "--host", "0.0.0.0"]. You will notice two small (but important) changes from the command we ran to start the app locally:
1. The `--reload` flag is missing. This is because `--reload` is only used in development environments. This flag forces the `uvicorn` server to reload the app whenever a change is detected in the source code. This is very useful when developing the app, but it is not recommended to use it in production.
2. The `--host` flag is set to `0.0.0.0`. The default value for this is `127.0.0.1`. This change is important because it allows the app to be accessed from outside the container. If we don't set this flag, the app will only be accessible from inside the container. `127.0.0.1` means that the app only acceps requests from the `localhost`.
