## 1. Estruturar o Projeto ASP.NET

- Estruture seu projeto ASP.NET Core, organizando as pastas e arquivos conforme o padrão desejado (ex: Clean Architecture).

- Configure o arquivo Program.cs para automatizar tarefas, como execução de migrations ao iniciar.

## 2. Configurar o Arquivo .dockerignore

- Inclua um arquivo .dockerignore na raiz do projeto para excluir arquivos e pastas desnecessárias no container (como bin, obj, .git, entre outros).

## 3. Criar o Dockerfile

- O Dockerfile contém as instruções para construir a imagem do projeto. Um exemplo básico para ASP.NET 8.0:

```dockerfile
# Etapa de build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copie a solução e os arquivos de projeto para restaurar as dependências
COPY AnimeAPI.sln .
COPY ["src/AnimeAPI.Domain/AnimeAPI.Domain.csproj", "AnimeAPI.Domain/"]
COPY ["src/AnimeAPI.Application/AnimeAPI.Application.csproj", "AnimeAPI.Application/"]
COPY ["src/AnimeAPI.Infrastructure/AnimeAPI.Infrastructure.csproj", "AnimeAPI.Infrastructure/"]
COPY ["src/AnimeAPI.API/AnimeAPI.API.csproj", "AnimeAPI.API/"]

# Restaurar dependências
RUN dotnet restore "AnimeAPI.sln"

# Copiar o restante dos arquivos e compilar o projeto
COPY . .
WORKDIR /src/AnimeAPI.API
RUN dotnet publish -c Release -o /app/publish

# Etapa final para imagem leve de execução
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "AnimeAPI.API.dll"]

```

## 4. Configurar o Docker Compose (se necessário)

- Um `docker-compose.yml` é útil para orquestrar múltiplos container, como API e banco de dados.

Exemplo de `docker-compose.yml` para API com PostgreSQL:

```yaml
version: "3.9"
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Host=db;Database=anime_db;Username=postgres;Password=1234
    depends_on:
      - db

  db:
    image: postgres:17
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 1234
      POSTGRES_DB: anime_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## 5. Configurar o Arquivo de Ambiente (`.env`)

- Crie um arquivo `.env` com variáveis sensíveis como a chave de API, se não quiser colocá-las diretamente no `docker-compose.yml`.

## 6. Adicionar Execução de Migrations (Opcional)

- Caso o projeto utilize um ORM como EntityFrameworkCore, para gerar as tabelas necessárias dentro do ambiente docker, é necessário configurar para executar as migrations quando iniciar o projeto. Para garantir que o banco de dados esteja atualizado, configure as migrations no `Program.cs` da API, chamando `Database.Migrate()`.

## 7. Construir e Rodar o Container

- Execute os comandos no terminal para construir e iniciar os containers:

```bash
docker-compose up --build
```

- Este comando irá construir e iniciar todos os containers definidos no `docker-compose.yml`.

## 8. Testar a API

- Com a aplicação rodando, teste as rotas da API acessando o Swagger ou chamando a API diretamente. No exemplo, a API está exposta em `http://localhost:5000`.
