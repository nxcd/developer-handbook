# Documentação de arquitetura

<!-- TOC -->

- [Documentação de arquitetura](#documentação-de-arquitetura)
  - [Sobre a arquitetura](#sobre-a-arquitetura)
    - [Estrutura de pastas](#estrutura-de-pastas)
    - [Camada de apresentação](#camada-de-apresentação)
    - [Camada de Serviço](#camada-de-serviço)
    - [Camada de dados](#camada-de-dados)
      - [Conexões](#conexões)
      - [Repositórios](#repositórios)
    - [Coluna de domínio](#coluna-de-domínio)
    - [Coluna de utilidades](#coluna-de-utilidades)

<!-- /TOC -->

## Sobre a arquitetura

Estamos utilizando o modelo baseado em *Domain Driven Design*, com ou sem [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html).

Este modelo se baseia em 3 camadas base que se comunicam de forma *top-down*, ou seja, a camada mais superior irá interagir com a mais inferior, mas nunca o contrário.

Todo o cenário é baseado na imagem a seguir:

![](./images/NXCD-Architecture.jpeg)

### Estrutura de pastas



### Camada de apresentação

A camada de apresentação será responsável

### Camada de Serviço

### Camada de dados

#### Conexões

#### Repositórios

### Coluna de domínio

### Coluna de utilidades