## Explicação Docker File
```docker
FROM python:3.12
RUN pip install poetry
COPY . /src
WORKDIR /src
RUN poetry install
EXPOSE 8501
ENTRYPOINT ["poetry","run", "streamlit", "run", "app/app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

- `FROM python:3.12`: Indica que esta imagem será baseada na imagem oficial do Python versão 3.12.
- `RUN pip install poetry`: Instala a ferramenta Poetry no ambiente Docker.
- `COPY . /src`: Copia o conteúdo do diretório atual (onde está localizado o Dockerfile) para o diretório "/src" dentro do contêiner.
- `WORKDIR /src`: Define o diretório de trabalho do contêiner como "/src".
- `RUN poetry install`: Usa o Poetry para instalar as dependências do projeto especificadas no arquivo pyproject.toml.
- `EXPOSE 8501`: Expõe a porta 8501 do contêiner para o mundo externo. Isso não faz com que a porta seja acessível diretamente do host, mas é uma convenção para informar que o contêiner escuta nessa porta.
- `ENTRYPOINT ["poetry", "run", "streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]`: Define o ponto de entrada padrão para o contêiner. Isso especifica o comando que será executado quando o contêiner for iniciado. Neste caso, ele executa o Streamlit para iniciar a aplicação "app.py", servindo-a na porta 8501. "--server.address=0.0.0.0" é usado para garantir que o Streamlit escute em todas as interfaces de rede dentro do contêiner, permitindo que seja acessível externamente.


## Explicação Docker Compose
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
- `db`: Este serviço utiliza a imagem oficial do Docker do PostgreSQL. **Ele configura um contêiner PostgreSQL com as variáveis de ambiente especificadas para a configuração do banco de dados (nome de usuário, senha e nome do banco de dados). Também mapeia um volume para persistir os dados do PostgreSQL e expõe a porta 5432 para o host.**
- `pgadmin`: Este serviço utiliza a imagem dpage/pgadmin4, que fornece uma interface baseada na web para gerenciar bancos de dados PostgreSQL. **Ele configura o email e a senha padrão para fazer login no pgAdmin, expõe a porta 8080 para o host e depende do serviço db.**
- `app`: Este serviço **cria uma imagem a partir do Dockerfile localizado no diretório atual (.).** Ele expõe a porta 8501 para o host e configura as variáveis de ambiente para conectar ao banco de dados PostgreSQL (DB_HOST, DB_NAME, DB_USER, DB_PASS). Também depende do serviço db.