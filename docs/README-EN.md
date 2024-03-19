## Explanation Docker File
```docker
FROM python:3.12
RUN pip install poetry
COPY . /src
WORKDIR /src
RUN poetry install
EXPOSE 8501
ENTRYPOINT ["poetry","run", "streamlit", "run", "app/app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

- `FROM python:3.12`: Indicates that this image will be based on the official Python version 3.12 image.
- `RUN pip install poetry`: Installs the Poetry tool in the Docker environment.
- `COPY . /src`: Copies the contents of the current directory (where the Dockerfile is located) to the "/src" directory inside the container.
- `WORKDIR /src`: Sets the working directory of the container to "/src".
- `RUN poetry install`: Uses Poetry to install the project dependencies specified in the pyproject.toml file.
- `EXPOSE 8501`: Exposes port 8501 of the container to the outside world. This does not make the port directly accessible from the host, but it's a convention to indicate that the container listens on this port.
- `ENTRYPOINT ["poetry", "run", "streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]`: Sets the default entry point for the container. This specifies the command that will be executed when the container is started. In this case, it runs Streamlit to start the "app.py" application, serving it on port 8501. "--server.address=0.0.0.0" is used to ensure that Streamlit listens on all network interfaces within the container, making it externally accessible.


## Explanation Docker Compose
```yml
version: '3.8'

services:
  db:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydatabase
    ports:
      - "5432:5432"

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: user@domain.com
      PGADMIN_DEFAULT_PASSWORD: adminpassword
    ports:
      - "8080:80"
    depends_on:
      - db

  app:
    build: .
    ports:
      - "8501:8501"
    environment:
      DB_HOST: db
      DB_NAME: mydatabase
      DB_USER: myuser
      DB_PASS: mypassword
    depends_on:
      - db

volumes:
  postgres_data:
```

- `db`: This service uses the official PostgreSQL Docker image. **It sets up a PostgreSQL container with the specified environment variables for the database configuration (username, password, and database name). It also maps a volume to persist the PostgreSQL data and exposes port 5432 to the host.**
- `pgadmin`: This service uses the dpage/pgadmin4 image, **which provides a web-based interface for managing PostgreSQL databases. It sets up the default email and password for logging into pgAdmin, exposes port 8080 to the host, and depends on the db service.**
- `app`: **This service builds an image from the Dockerfile located in the current directory (.).** It exposes port 8501 to the host and sets up environment variables for connecting to the PostgreSQL database (DB_HOST, DB_NAME, DB_USER, DB_PASS). It also depends on the db service.