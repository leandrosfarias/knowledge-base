# Logging Eficiente com Serilog e Seq

O logging é uma prática essencial para monitoramento, depuração e manutenção de qualquer aplicação. Serilog é uma biblioteca de logging de alto desempenho para .NET, que permite capturar logs estruturados. Seq é uma plataforma para coletar, armazenar e visualizar logs estruturados de várias fontes. Quando integrados, Serilog e Seq fornecem uma solução poderosa para registrar e analisar dados de logs.

## 1. Por Que Usar Logging Estruturado?

O logging estruturado permite capturar logs como objetos com propriedades específicas, facilitando buscas e filtros. Em vez de apenas armazenar mensagens de texto, logs estruturados permitem definir propriedades como userId, transactionId, endpoint e outros dados específicos que tornam mais fácil encontrar problemas e gerar insights.

## 2. Configuração do Serilog no Projeto .NET

1. **Adicionar o Serilog ao Projeto**

Para usar o Serilog, adicione os seguintes pacotes NuGet ao seu projeto .NET:

```shell
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Seq
```

2. **Configurar o Serilog no `Program.cs`**

A configuração do Serilog pode ser feita no arquivo `Program.cs`. Aqui está um exemplo básico de configuração para direcionar logs ao console e ao Seq:

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configuração do Serilog com Seq
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .Enrich.FromLogContext()
    .WriteTo.Console()  // Loga no console
    .WriteTo.Seq("http://localhost:5341")  // Loga no Seq (ajuste para o endereço do Seq)
    .CreateLogger();

builder.Host.UseSerilog();  // Integrando Serilog com o pipeline do ASP.NET

var app = builder.Build();
```

3. **Configuração do `appsettings.json`**

Serilog pode ser configurado para ler parâmetros diretamente do appsettings.json:

```
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      { "Name": "Console" },
      { "Name": "Seq", "Args": { "serverUrl": "http://localhost:5341" } }
    ]
  }
}
```

4. Iniciar o Projeto com Serilog

Agora, Serilog capturará logs e os enviará para o console e para o Seq. Você poderá visualizar e monitorar os logs com mais clareza e flexibilidade.

3. **Configuração do Seq**

1. Instalar o Seq

Faça o download do Seq em https://datalust.co/download e siga as instruções de instalação. Você também pode rodar o Seq em um container Docker com o comando:

```shell
docker run --name seq -d --restart unless-stopped -e ACCEPT_EULA=Y -p 5341:80 datalust/seq
```

2. Configurar o Serilog para Enviar Logs para o Seq

Como configurado no exemplo anterior, defina o endpoint do Seq em `WriteTo.Seq("http://localhost:5341")`. Você poderá acessar a interface do Seq em `http://localhost:5341` para visualizar os logs.

3. Filtragem e Pesquisa no Seq

O Seq permite que você filtre e pesquise logs usando consultas estruturadas. É possível pesquisar por propriedades de log, como `TransactionId`, `UserId`, e ver tendências de erros e métricas de desempenho.

**4. Boas Práticas de Logging com Serilog e Seq**

1. Definir Níveis de Log Adequados

- Use `Debug` para informações detalhadas de desenvolvimento, `Information` para eventos normais, `Warning` para eventos suspeitos e `Error` para problemas críticos.

2. Enriquecer Logs com Dados de Contexto

- Use `.Enrich.WithProperty()` para adicionar propriedades importantes, como `UserId`, `TransactionId` e `RequestPath`:

```csharp
Log.ForContext("UserId", userId)
   .ForContext("TransactionId", transactionId)
   .Information("Processando pedido para o usuário {UserId}");
```

3. Manter a Performance

- Evite sobrecarregar logs com dados excessivos. Limite a quantidade de dados nos níveis `Information` e `Debug`.

4. Segurança dos Logs

- Certifique-se de não registrar informações sensíveis como senhas ou dados pessoais. Use redatores automáticos quando possível para remover ou mascarar dados confidenciais.

5. Usar o Seq para Alertas e Monitoramento

- Configure alertas no Seq para monitorar níveis específicos de erro ou eventos críticos. Você pode integrar alertas com notificações por email ou Slack, permitindo respostas rápidas a problemas.

**5. Exemplos Práticos de Logging com Serilog e Seq**

1. Log de Requisição e Resposta HTTP

Logar informações de requisição e resposta pode ser muito útil para monitorar o tráfego da API:

```csharp
app.Use(async (context, next) =>
{
    var requestPath = context.Request.Path;
    Log.Information("Requisição recebida: {RequestPath}", requestPath);

    await next();

    Log.Information("Resposta enviada para {RequestPath} com status {StatusCode}",
                     requestPath, context.Response.StatusCode);
});
```

2. Log de Erro em Operações de Banco de Dados

Adicione logs em operações de banco de dados para facilitar a identificação de problemas relacionados a dados:

```csharp
try
{
    // Operação no banco de dados
}
catch (Exception ex)
{
    Log.Error(ex, "Erro ao acessar o banco de dados para a operação {Operation}", "InsertUser");
}
```

6. Conclusão

Integrar **Serilog** com **Seq** em seu projeto .NET permite implementar uma estratégia de logging estruturada e eficiente. Serilog facilita a coleta de logs detalhados e Seq oferece uma poderosa interface de análise. Com essas ferramentas, você pode monitorar e entender o comportamento da sua aplicação em tempo real, resolver problemas rapidamente e manter a qualidade do código.

Ao seguir essas práticas, você estará preparado para manter logs de alta qualidade e usar os dados coletados para otimizar a performance e confiabilidade da sua aplicação.
