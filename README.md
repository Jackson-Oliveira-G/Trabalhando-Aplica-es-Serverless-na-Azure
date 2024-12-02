# Trabalhando-Aplicaçoes-Serverless-na-Azure
Trabalhando Aplicações Serverless na Azure

Para desenvolver um projeto trabalhando com **aplicações Serverless na Azure**, podemos utilizar o **Azure Functions**, que permite executar código em resposta a eventos sem a necessidade de gerenciar servidores. Também podemos integrar com outros serviços do Azure, como **Azure Blob Storage**, **Azure Cosmos DB**, e **Azure API Management**, para criar uma aplicação completa.

Neste guia, vamos construir um projeto simples de **API Serverless** utilizando **Azure Functions** para lidar com requisições HTTP e **Azure Cosmos DB** para armazenar e recuperar dados.

### Passos do Projeto:

1. **Configuração do Ambiente**:
   - Crie uma conta na Azure (se ainda não tiver).
   - Instale o **Azure CLI** (se ainda não tiver).
   - Instale o **Visual Studio Code** (ou outra IDE de sua preferência).
   - Instale a extensão **Azure Functions** para o Visual Studio Code.
   - Instale o **.NET Core SDK** (para desenvolvimento com C#) ou **Node.js** (se você preferir JavaScript/TypeScript).

2. **Criar uma Função Serverless no Azure**:
   Vamos criar uma função HTTP que será acionada via uma requisição e interagirá com o **Azure Cosmos DB** para salvar e obter dados.

#### Etapa 1: Criar uma Função Serverless na Azure

1. **Criando a Função no Visual Studio Code**:
   - Abra o Visual Studio Code.
   - Instale a extensão "Azure Functions".
   - Abra o painel do Azure Functions no VS Code e clique em "Criar Novo Projeto".
   - Escolha a linguagem de sua preferência (C# ou JavaScript/TypeScript).
   - Escolha o template "HTTP Trigger" (para responder a requisições HTTP).
   - Nomeie a função (por exemplo, `HttpExampleFunction`).
   
   O Visual Studio Code irá criar o projeto com uma função básica que responde a requisições HTTP.

#### Exemplo em C# (para uma função HTTP trigger):

```csharp
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

public static class HttpExampleFunction
{
    [FunctionName("HttpExampleFunction")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        // Pega o corpo da requisição (em JSON)
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);

        // Conexão com o Cosmos DB
        var client = new DocumentClient(new Uri("https://<your-cosmos-db-uri>"), "<your-cosmos-db-key>");
        var collectionUri = UriFactory.CreateDocumentCollectionUri("your-database", "your-collection");

        // Inserir no Cosmos DB
        var document = new { id = Guid.NewGuid(), name = data?.name, age = data?.age };
        await client.CreateDocumentAsync(collectionUri, document);

        return new OkObjectResult($"Documento inserido com sucesso.");
    }
}
```

#### Exemplo em Node.js (para uma função HTTP trigger):

```javascript
const { CosmosClient } = require("@azure/cosmos");

module.exports = async function (context, req) {
    context.log('HTTP trigger function processed a request.');
    
    // Extrair o corpo da requisição (JSON)
    const data = req.body;

    // Conectar ao Azure Cosmos DB
    const client = new CosmosClient({
        endpoint: "https://<your-cosmos-db-uri>",
        key: "<your-cosmos-db-key>"
    });

    const container = client.database("your-database").container("your-container");

    // Inserir no Cosmos DB
    const { resource: createdItem } = await container.items.create({
        id: data.id,
        name: data.name,
        age: data.age
    });

    context.res = {
        status: 200,
        body: "Documento inserido com sucesso!"
    };
};
```

#### Etapa 2: Configurar o **Azure Cosmos DB**

1. **Criar um Banco de Dados e Contêiner no Cosmos DB**:
   - Acesse o portal da Azure e crie uma nova instância do **Azure Cosmos DB** (se ainda não tiver).
   - Escolha o **API** do Cosmos DB que você prefere usar (SQL API é o mais comum).
   - Crie um banco de dados e um contêiner. O contêiner pode ser configurado para armazenar documentos no formato JSON.

2. **Obter as Credenciais de Conexão**:
   - Acesse a instância do Cosmos DB no portal da Azure.
   - Vá para **Keys** e copie o **URI** e a **Primary Key** para usar no código da função Azure.

#### Etapa 3: Testando Localmente

1. **Executar a Função Localmente**:
   - No Visual Studio Code, abra o terminal e execute o comando para rodar a função localmente:
   
     ```
     func start
     ```

2. **Testar a Função**:
   - Após o servidor ser iniciado, a função estará disponível em `http://localhost:7071/api/HttpExampleFunction`.
   - Você pode usar o **Postman** ou **cURL** para enviar uma requisição `POST` com um corpo JSON.

   Exemplo de corpo JSON:
   ```json
   {
       "name": "João",
       "age": 30
   }
   ```

3. **Verificar no Cosmos DB**:
   - Acesse o **Azure Cosmos DB** e verifique se o documento foi inserido com os dados enviados na requisição.

#### Etapa 4: Implantando na Azure

1. **Publicar a Função na Azure**:
   - No Visual Studio Code, clique com o botão direito no projeto da Azure Function e selecione **Deploy to Function App**.
   - Siga as instruções para criar e configurar o Function App na Azure, selecionando a assinatura e o grupo de recursos.
   - Após o deployment ser concluído, o URL da sua função será fornecido.

2. **Testar a Função na Nuvem**:
   - Agora que a função está na Azure, você pode testar a API HTTP na URL fornecida pela Azure.
   - Envie uma requisição `POST` para o endpoint gerado, utilizando o **Postman** ou **cURL**, com o mesmo corpo JSON que testamos localmente.

### Conclusão

Este projeto exemplifica como criar uma **aplicação serverless** usando o **Azure Functions** e o **Azure Cosmos DB**. A solução não requer gerenciamento de servidores e é altamente escalável, aproveitando as capacidades da Azure para gerenciar os recursos de maneira eficiente.

Além disso, você pode expandir o projeto integrando outros serviços Azure, como:
- **Azure Blob Storage** para armazenar arquivos.
- **Azure API Management** para gerenciar e expor APIs REST de forma segura.
- **Azure Event Grid** ou **Azure Service Bus** para eventos assíncronos e integração com outros sistemas.

Esse tipo de arquitetura é ideal para situações onde você precisa de escalabilidade automática e baixo custo, como para aplicativos web, APIs, e funções de processamento em tempo real.
