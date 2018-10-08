# Documentação de arquitetura

[TOC]

## Sobre processos de desenvolvimento

O processo de desenvolvimento do sistema é baseado no modelo ágil // Continua amanhã

## Sobre Estilos de código

- Vamos utilizar o estilo pré-pronto [standardJS](https://standardjs.com/) como **linter**.

- [Prettier](https://prettier.io) como **formatter**.
- Vamos utilizar [PNPM](https://pnpm.js.org/) como gerenciador de dependências porque ele evita instalação de dependências duplicadas e também gera menos instalações de pacotes por gerar SymLinks ao invés de reinstalar tudo (leia a doc)
  - Principalmente para desenvolvimento local, uma vez que CI/CDs podem não ter suporte a este instalador
- Utilizamos o [TypeScript](https://www.typescriptlang.org/) como linguagem de programação pelos motivos abaixo:
  - Em um time grande, a tipagem estática do TS ajuda a manter a ordem do repositório
  - Podemos criar interfaces tanto para os parâmetros de entrada quanto para os dados de saída e também para os retornos de APIs
  - Podemos, facilmente, alterar o alvo de nossas builds para sistemas mais antigos, mais novos e também utilizar propostas ainda não implementadas pelo TC39 através do `tsconfig.json` usando o Babel

>  **Importante**
>
> Linters e formatters são coisas diferentes. Enquanto o Linter vai somente dizer aonde que o código está fugindo do estilo, o formatter irá, de fato alterar o estilo de código e salvar os arquivos.

### Arquivos de configuração

Todo projeto vai ter arquivos de configuração padrões, os chamados *dotfiles*, que são responsáveis por configurar nossas extensões e editores e devem ficar na raiz do projeto, juntamente com a pasta `.git`.

#### Git Ignore

Todo projeto deve ter um `.gitignore` que pode ser gerado [neste endereço](https://www.gitignore.io/api/vim,node,linux,macos,windows,visualstudiocode) e complementado depois. Ele vai prover os principais arquivos de *ignore* para os principais IDEs que usamos e a linguagem Node.

#### EditorConfig

O [EditorConfig](https://editorconfig.org) é um modelo de padronização de código em editores e IDEs, a grande maioria dos editores possuem um plugin para ler arquivos `.editorconfig` que ficam na raiz do projeto e aplicar suas regras ao código escrito em todos os arquivos:

**.editorconfig:**

```ini
[*]
end_of_line = lf
insert_final_newline = true

[*.js]
charset = utf-8
indent_style = space
indent_size = 2
trim_trailing_whitespace = true

[*.html]
charset = utf-8
indent_style = space
indent_size = 2
```

Como a maioria do time usa o [VSCode](https://code.visualstudio.com/), o plugin para ele está [no link da extensão](https://github.com/editorconfig/editorconfig-vscode)

#### StandardJS

Para o standard, vamos instalar **sempre**, a biblioteca *StandardJS* como *Dev Dependency* no NPM através de `pnpm i -D standard`.

Todo o projeto também terá um script de lint que poderá ser acessado usando `pnpm run lint` ou `npm run lint` que, basicamente, irá rodar `standard`. O ideal será rodar este comando antes de dar push para o repositório. Porém, se o desenvolvedor se esquecer, o CI irá fazer o trabalho.

#### Prettier

Vamos utilizar o Prettier para poder padronizar o modelo de desenvolvimento de código. Para isso, vamos executar `pnpm install --exact -D prettier`.

> Vamos utilizar `—exact` porque mudanças de versão do Prettier causam alterações de estilo no core do sistema e é recomendado utilizar a versão exata no `package.json`

Arquivo de configuração `.prettierrc` que ficará na raiz: 

**.prettierrc:**

```json
{
  "tabWidth":2,
  "useTabs":false,
  "semi":false,
  "singleQuote":true,
  "trailingComma":"none",
  "bracketSpacing":true,
  "arrowParens":"always"
}
```

Descrição das regras do arquivo:

- Uma `TAB` equivale a 2 espaços
- Sempre vamos usar espaços ao invés de *tabs*
- Não haverá `;` no final das linhas
- Sempre usaremos `'` ao invés de `"`
- Não vamos usar `,` no final da última propriedade de objetos
- Em um objeto do tipo `{key:value}` vamos ter espaços entre chaves, ficando `{ key: value }`
- Toda arrow function do tipo `param => { … }` vai se tornar `(param) => { … }`

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