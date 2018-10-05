# Documentação de arquitetura

<!-- TOC -->

- [Documentação de arquitetura](#documentação-de-arquitetura)
  - [Sobre processos de desenvolvimento](#sobre-processos-de-desenvolvimento)
  - [Sobre Estilos de código](#sobre-estilos-de-código)
  - [Sobre a arquitetura de código](#sobre-a-arquitetura-de-código)
    - [Estrutura de pastas](#estrutura-de-pastas)
    - [Camada de apresentação](#camada-de-apresentação)
    - [Camada de Serviço](#camada-de-serviço)
    - [Camada de dados](#camada-de-dados)
      - [Conexões](#conexões)
      - [Repositórios](#repositórios)
    - [Coluna de domínio](#coluna-de-domínio)
    - [Coluna de utilidades](#coluna-de-utilidades)
  - [Sobre infraestrutura e disposição do sistema](#sobre-infraestrutura-e-disposição-do-sistema)
    - [Infraestrutura](#infraestrutura)
    - [Mapa do Sistema](#mapa-do-sistema)
      - [NextGate](#nextgate)
      - [Billing](#billing)
      - [Next-ID](#next-id)

<!-- /TOC -->

## Sobre processos de desenvolvimento

// Metodologias vão aqui

## Sobre Estilos de código

// StyleGuides e linguagens vão aqui

## Sobre a arquitetura de código

Estamos utilizando o modelo baseado em *Domain Driven Design*, com ou sem [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html).

Este modelo se baseia em 3 camadas base que se comunicam de forma *top-down*, ou seja, a camada mais superior irá interagir com a mais inferior, mas nunca o contrário.

Todo o cenário é baseado na imagem a seguir:

![](./images/NXCD-Architecture.jpeg)

### Estrutura de pastas

A estrutura de pastas seguirá o modelo abaixo, mas você pode checar em tempo real acessando o [diretório de exemplo](./folder-structure):

```
project-x
    |   .dockerignore
    |   .editorconfig
    |   .envrc
    |   .envrc-sample
    |   .gitignore
    |   app-config.ts
    |   ci-config.yaml
    |   Dockerfile
    |   package.json
    |
    +---.git
    \---src
        |   index.ts
        |
        +---data
        |   +---connections
        |   |       mongo.ts
        |   |       sql-server.ts
        |   |
        |   \---repositories
        |           entity-x.ts
        |           entity-y.ts
        |
        +---domains
        |   |   domain-base-error.ts
        |   |   index.ts
        |   |
        |   +---entity-x
        |   |   |   entity.ts
        |   |   |   initial.state.ts
        |   |   |
        |   |   +---commands
        |   |   |       create.ts
        |   |   |       delete.ts
        |   |   |       update.ts
        |   |   |
        |   |   +---errors
        |   |   |       entity-x-error.ts
        |   |   |
        |   |   +---events
        |   |   |       entity-x-was-created.ts
        |   |   |       entity-x-was-deleted.ts
        |   |   |       entity-x-was-finished.ts
        |   |   |       entity-x-was-updated.ts
        |   |   |
        |   |   \---interfaces
        |   |           data.ts
        |   |           filter.ts
        |   |
        |   \---entity-y
        |       +---commands
        |       +---errors
        |       +---events
        |       \---interfaces
        +---libs
        |       cpf-validator.ts
        |       helper-abc.ts
        |
        +---presentation
        |   |   app.ts
        |   |   server.ts
        |   |
        |   \---routes
        |           route-a.ts
        |           route-b.ts
        |
        \---services
                entity-x.ts
                entity-y.ts
```

- `project-x`: É a pasta inicial que vai conter o nome do projeto, tudo estará dentro desta pasta
  - **Arquivos de estrutura**: Abaixo da raiz, no primeiro nível, vamos ter os dotfiles necessários para configuração de CI/CD, Docker e outras informações; aqui será aonde vamos ter o `package.json` e o `tsconfig.json`
  - `.env` ou `.envrc`: Variáveis de ambiente devem ficar em um arquivo `.envrc` para **execução local**, toda a execução em imagem teremos um CI que vai passar as variáveis de ambiente para o container. Todo `.envrc` deve ter um `.envrc-sample` com a lista das variáveis de ambiente daquela aplicação
  - `src`: Pasta que conterá todo o código fonte da aplicação
    - `index.ts`: É o *entrypoint* da aplicação, será nele que a aplicação vai começar
    - `data`: É a camada de dados da aplicação, conforme especificada [em sua seção](#camada-de-dados)
      - `connections`: Aqui ficarão todos os arquivos referentes a conexões com bancos de dados e outras fontes
      - `repositories`: Aqui ficarão os manipuladores dos clientes de fontes de dados existentes em `connections`, esta será a pasta que conterá os arquivos que serão consumidos pela [camada de serviços](#camada-de-serviço)
    - `domains`: Será a coluna de domínio, conforme especificada [em sua seção](#coluna-de-dominio)
      - `index.ts`: Retornará um objeto com todas as entidades
      - `domain-base.error.ts`: O erro base do domínio que todas as entidades devem estender
      - `<entity-x>`: Cada pasta dentro da camada de domínio terá o nome de uma entidade deste domínio
        - `entity.ts`: Toda entidade terá um arquivo `entity.ts` que será o arquivo que juntará todos os comandos, eventos e erros. Em teoria, será o local aonde as demais camadas irão buscar a instância do domínio em si
        - `initial.state.ts`: Toda entitade tem um estado inicial que será colocado neste arquivo
        - `commands`: Comandos são as instruções necessárias para se manipular esta entidade, aqui ficarão os métodos e funções que retornam os arrays de eventos (no caso de event-sourcing) necessários para criar tal entidade
        - `errors`: Aonde ficarão as classes de erro da entidade, todas devem estender o arquivo base de erro
        - `events`: Somente utilizado no caso de event-sourcing, será aonde estarão armazenados as ações que podem ser executadas sobre esta entidade em forma de eventos, todos os arquivos devem ter o nome `<nome-da-entidade>-descrição-do-evento.ts`
        - `interfaces`: Aqui ficarão as interfaces do TypeScript para tal entidade, não só as interfaces mas também, se necessários, Enums e qualquer outra estrutura de definição de tipos
    - `libs`: Aqui ficarão os arquivos de bibliotecas, helpers e qualquer outro arquivo que sirva de repositório de utilidades
    - `presentation`: A camada de apresentação ficará descrita aqui, conforme [descrição de sua seção](#camada-de-apresentação)
      - `app.ts`: Neste arquivo serão carregadas todas as rotas da aplicação, exportará o modelo base de um webserver
      - `server.ts`: Irá receber `app.ts` e irá iniciar um servidor web
      - `routes`: Aqui ficarão os handlers das rotas, toda a rota deverá ter o nome `<verbo>-descrição.ts`
    - `services`: Camada de serviços conforme [descrição](#camada-de-serviço)
      - `<entity-x>`: Para cada entidade em `domains` temos que ter um arquivo aqui, que será o responsável por se comunicar com a camada de dados e manipular os dados no banco seguindo o comando que for executado pelo domínio

### Camada de apresentação

A camada de apresentação será responsável por receber as chamadas do [NextGate](#nextgate) e encaminhar para a camada de serviços.

Esta camada também vai ser responsável por receber as respostas da camada de serviço e apresentar para o usuário em forma de JSON ou qualquer outro content type.

Além disso, todos os erros que vierem das camadas inferiores, sejam eles relativos ou não à camada de apresentação, serão tratados e exibidos pelo usuário nesta mesma camada.

### Camada de Serviço

### Camada de dados

#### Conexões

#### Repositórios

### Coluna de domínio

### Coluna de utilidades

## Sobre infraestrutura e disposição do sistema

### Infraestrutura

O fluxo de infraestrutura não é muito bem definido em detalhes porque ainda não foi totalmente pensado, inicialmente pensamos em utilizar um modelo como o abaixo:

![](./images/NXCD-Infrastructure.jpeg)

1. Desenvolvedores vão codificar a aplicação
2. O push no repositório vai ativar o build no CI
3. CI irá rodar os testes automatizados
4. Se todos os testes passarem, o CI irá construir a imagem
5. Depois de construída, vamos enviar a imagem para ser armazenada em um registry para que possamos baixar estas imagens individualmente depois e testar versão a versão
6. Após o término da build da imagem, o CI irá iniciar um deploy na infraestrutura do Kubernetes

### Mapa do Sistema

O sistema ficará organizado da seguinte forma:

![](./images/NXCD-System-flow.png)

Teremos um total de três aplicações:

#### NextGate

Será o gateway pelo qual todas as mensagens irão trafegar suas responsabilidades incluem:

  - Autorizar escopos de usuário baseado em um token recebido pelo serviço de autenticação externo Auth0
  - Verificar Rate Limits e concorrências dos clientes se comunicando com as aplicações de cobrança
  - Fazer o proxy entre a requisição das áreas logadas e as áreas não logadas
  - Contar números de requisições e processos simultâneos, inserindo os dados de telemetria nos bancos de dados

Se alguma das regras não passarem, a request não será completada

#### Billing

A aplicação de cobrança será a inteligência por trás dos dados brutos de acesso e uso de aplicações do usuário. Esta aplicação:

- Quando esta API está sendo utilizada
- Quanto tempo de uso cada usuário está tendo
- Quantidade de chamadas de uma API
- Calcular preços baseados em todos esses dados

O NextGate se comunicará com esta aplicação para poder descobrir se um usuário excedeu ou não sua cota de uso.

> Talvez seja necessário quebrar esta aplicação em algumas partes para ter responsabilidades mais isoladas.

#### Next-ID

Aplicação propriamente dita