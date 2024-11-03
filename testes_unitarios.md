# Guia de Testes Unitários em .NET com xUnit e Moq

Este guia passo a passo é voltado para desenvolvedores que desejam aprender a implementar testes unitários em projetos .NET, usando xUnit para estrutura de testes e Moq para criação de mocks.

## Conteúdo

- [Visão Geral](#visão-geral)
- [Configuração Inicial](#configuração-inicial)
  - [Instalação do xUnit](#instalação-do-xunit)
  - [Instalação do Moq](#instalação-do-moq)
  - [Estrutura do Projeto de Testes](#estrutura-do-projeto-de-testes)
- [Criação de Testes Unitários com xUnit](#criação-de-testes-unitários-com-xunit)
  - [Estrutura Básica de Teste xUnit](#estrutura-básica-de-teste-xunit)
  - [Atributos e Anotações](#atributos-e-anotações)
- [Mocking com Moq](#mocking-com-moq)
  - [Configuração de Mocks](#configuração-de-mocks)
  - [Exemplo de Mock com Moq](#exemplo-de-mock-com-moq)
- [Dicas para Escrever Bons Testes](#dicas-para-escrever-bons-testes)

---

## Visão Geral

Testes unitários são uma prática essencial para garantir que o comportamento do seu código esteja correto e para facilitar a manutenção e expansão do projeto. Em .NET, o xUnit é uma das bibliotecas mais populares para estruturação de testes, enquanto o Moq é usado para simular objetos externos e dependências.

## Configuração Inicial

### Instalação do xUnit

Para instalar o xUnit em um projeto de testes .NET, você pode utilizar o **NuGet**. No terminal, execute:

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
```

- `xunit`: pacote principal para criar testes.
- `xunit.runner.visualstudio`: permite executar testes no Visual Studio.

### Instalação do Moq

Para instalar o Moq, execute o seguinte comando:

```bash
dotnet add package Moq
```

O Moq permit criar simulações (mocks) de dependências, essencial para isolar o código em testes.

### Estrutura do Projeto de Testes

É comum que o projeto de testes siga o nome do projeto principal com .UnitTests como sufixo. Exemplo:

```plaintext
AnimeAPI.sln
├── AnimeAPI/
├── AnimeAPI.Application/
├── AnimeAPI.Infrastructure/
└── AnimeAPI.UnitTests/   <-- Projeto de testes
```

No arquivo `.sln`, o projeto de testes deve referenciar os projetos que você irá testar, e você deve adicionar referências entre os projetos com o comando:

```bash
dotnet add reference ../AnimeAPI/AnimeAPI.csproj
```

### Criação de Testes Unitários com xUnit

#### Estrutura Básica de Teste xUnit

Um teste unitário básico em xUnit possui três partes principais: Arrange, Act e Assert.

```csharp
public class AnimeServiceTests
{
    [Fact]
    public void MetodoDeve_RetornarTrue_QuandoCondicaoForVerdadeira()
    {
        // Arrange
        var servico = new AnimeService();

        // Act
        var resultado = servico.VerificaCondicao(true);

        // Assert
        Assert.True(resultado);
    }
}
```

#### Atributos e Anotações

Os principais atributos para marcar métodos de teste no xUnit são:

- `[Fact]`: Indica um teste que não possui parâmetros.
- `[Theory]`: Permite passar uma série de parâmetros ao teste, útil para testar diferentes cénarios.

Exemplo de uso do `[Theory]` com `[InlineData]`:

```csharp
[Theory]
[InlineData(1, 2, 3)]
[InlineData(-1, -1, -2)]
public void Soma_DeveRetornarResultadoCorreto(int a, int b, int resultadoEsperado)
{
    // Arrange
    var servico = new Calculadora();

    // Act
    var resultado = servico.Soma(a, b);

    // Assert
    Assert.Equal(resultadoEsperado, resultado);
}
```

### Mocking com Moq

Mocks são úteis para isolar o teste de dependências externas, como bancos de dados ou serviços de API.

#### Configuração de Mocks

Para criar um mock de uma interface ou dependência, utilize o Moq da seguinte forma:

```csharp
var mockRepositorio = new Mock<IAnimeRepository>();
```

#### Exemplo de Mock com Moq

Considere um serviço `AnimeService` que depende de `IAnimeRepository`. Com o Moq, podemos simular o comportamento do repositório sem precisar de uma implementação real:

```csharp
public class AnimeServiceTests
{
    private readonly Mock<IAnimeRepository> _mockRepository;
    private readonly AnimeService _animeService;

    public AnimeServiceTests()
    {
        _mockRepository = new Mock<IAnimeRepository>();
        _animeService = new AnimeService(_mockRepository.Object);
    }

    [Fact]
    public void GetAnime_DeveRetornarAnime_QuandoAnimeExiste()
    {
        // Arrange
        var anime = new Anime { Id = 1, Nome = "Naruto" };
        _mockRepository.Setup(repo => repo.GetById(1)).Returns(anime);

        // Act
        var resultado = _animeService.GetAnime(1);

        // Assert
        Assert.NotNull(resultado);
        Assert.Equal("Naruto", resultado.Nome);
    }
}
```

No exemplo acima:

- `Setup`: Configura o método `GetById` do repositório para retornar um anime específico.
- `Assert`: Verifica se o serviço retorna o anime corretamente.

#### Verificar Chamadas de Método com `Verify`

Você também pode usar o método `Verify` do Moq para garantir que um método foi chamado durante o teste:

```csharp
_mockRepository.Verify(repo => repo.GetById(1), Times.Once);
```

### Dicas para Escrever Bons Testes

1. **Independência**: Cada teste deve ser independente de outros testes.
2. **Nominação Clara**: Nomeie seus testes de forma descritiva. Exemplo: `MetodoDeve_RetornarValorCorreto_QuandoCondicaoVerdadeira`.
3. **Evitar Lógica Complexa**: Testes unitários não devem conter lógica complexa ou condições.
4. **Mantenha o Teste Simples**: Foque em uma única funcionalidade por teste.
5. **Cobertura de Código**: Escreva testes suficiente para cobrir a maior parte possível de código.

Esse guia oferece uma introdução prática ao uso de xUnit e Moq para testar projetos em .NET. Com prática e aplicação dos conceitos descritos aqui, será possível criar testes unitários eficazes e manter seu código mais confiável.

### Evitar Lógica Complexa nos Testes

Testes unitários devem ser simples e focados. O objetivo de um teste unitário é validar uma unidade específica de código com um conjunto claro de entradas e saídas esperadas, sem introduzir complexidade adicional. Abaixo estão os pontos principais sobre por que e como evitar lógica complexa nos testes unitários.

#### 1. **Por Que Evitar Lógica Complexa?**

- **Facilidade de Leitura**: Testes unitários devem ser fáceis de ler e entender. Lógica complexa nos testes torna o entendimento do que está sendo testado e o diagnóstico de falhas muito mais difícil.
- **Reduzir o Risco de Erros nos Testes**: Quanto mais lógica complexa você adiciona aos testes, maior o risco de introduzir erros nos próprios testes, tornando-os menos confiáveis.
- **Isolamento**: Testes devem se concentrar em verificar o comportamento de uma unidade de código e não em condições de fluxo alternativo. Testes complexos podem, muitas vezes, desviar da simplicidade e do isolamento necessário para identificar problemas.

#### 2. **O Que Significa Lógica Complexa em Testes?**

Lógica complexa em testes pode se manifestar de várias formas:

- **Estruturas Condicionais (if, else)**: Adicionar condições nos testes pode gerar casos de testes múltiplos dentro de um único teste, o que é confuso e faz com que o teste não esteja validando apenas uma unidade de funcionalidade.
- **Estruturas de Loop (for, while)**: Loops podem complicar o teste, especialmente quando você está tentando verificar diferentes casos com diferentes entradas e saídas.
- **Lógica de Negócio**: Lógica de negócios e cálculos devem ser testados em seus próprios métodos, e não duplicados no próprio teste.

#### 3. **Como Evitar Lógica Complexa?**

Aqui estão algumas práticas para manter seus testes simples e sem lógica complexa:

- **Divida Testes com Vários Cenários**: Em vez de usar estruturas condicionais para verificar várias condições em um único teste, crie testes separados para cada cenário. Cada teste deve validar um único caminho ou resultado.

  ```csharp
  // Evite isso:
  [Fact]
  public void VerificaValor_QuandoCondicaoAForVerdadeiraOuCondicaoBForFalsa()
  {
      if (condicaoA)
      {
          Assert.Equal(esperado, resultadoA);
      }
      else
      {
          Assert.Equal(esperado, resultadoB);
      }
  }

  // Prefira separar cenários em testes individuais:
  [Fact]
  public void VerificaValor_QuandoCondicaoAForVerdadeira()
  {
      // Arrange, Act
      Assert.Equal(esperado, resultadoA);
  }

  [Fact]
  public void VerificaValor_QuandoCondicaoBForFalsa()
  {
      // Arrange, Act
      Assert.Equal(esperado, resultadoB);
  }
  ```

- **Use Teorias e Dados Inline para Cenários Múltiplos**: Para evitar loops e estruturas condicionais, utilize `[Theory]` e `[InlineData]` para passar diferentes entradas para o mesmo método de teste.

  ```csharp
  [Theory]
  [InlineData(1, 2, 3)]
  [InlineData(-1, -1, -2)]
  [InlineData(5, 5, 10)]
  public void Soma_DeveRetornarResultadoCorreto(int a, int b, int resultadoEsperado)
  {
      // Arrange
      var servico = new Calculadora();

      // Act
      var resultado = servico.Soma(a, b);

      // Assert
      Assert.Equal(resultadoEsperado, resultado);
  }
  ```

- **Mock para Isolar Dependências**: Em vez de reproduzir a lógica de dependências no teste, utilize mocks para simular o comportamento da dependência de forma controlada.

  ```csharp
  var mockRepositorio = new Mock<IRepositorio>();
  mockRepositorio.Setup(r => r.BuscarPorId(1)).Returns(new Objeto { Id = 1 });

  var resultado = servico.GetObjeto(1);

  Assert.NotNull(resultado);
  ```

- **Mantenha o Teste Estruturado com Arrange, Act, Assert**: Seguir o padrão "Arrange, Act, Assert" ajuda a manter os testes diretos e focados. Cada etapa do teste deve ser claramente separada e sem introduzir elementos desnecessários.

  ```csharp
  [Fact]
  public void VerificaCondicaoDeRetorno()
  {
      // Arrange
      var servico = new Servico();

      // Act
      var resultado = servico.CalculaCondicao();

      // Assert
      Assert.Equal(valorEsperado, resultado);
  }
  ```

---

Manter os testes simples ajuda a garantir que você está testando o código, e não o próprio teste. Quanto mais direto e independente for um teste, mais eficaz ele será para garantir que a funcionalidade do código está intacta e funcionando como esperado.
