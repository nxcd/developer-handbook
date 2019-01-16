# Sobre Estilos de código

<!-- TOC -->

- [Sobre Estilos de código](#sobre-estilos-de-código)
  - [Arquivos de configuração](#arquivos-de-configuração)
    - [Git Ignore](#git-ignore)
    - [EditorConfig](#editorconfig)
    - [StandardJS](#standardjs)
    - [.dockerignore](#dockerignore)
    - [Dockerfile](#dockerfile)
    - [docker-compose.yml](#docker-composeyml)
    - [tsconfig.json](#tsconfigjson)
    - [package.json](#packagejson)
  - [Testes](#testes)

<!-- /TOC -->

- Vamos utilizar o estilo pré-pronto [standardJS](https://standardjs.com/) como **linter**. O script de lint rodará no hook de **commit** do git
- Vamos utilizar [PNPM](https://pnpm.js.org/) como gerenciador de dependências porque ele evita instalação de dependências duplicadas e também gera menos instalações de pacotes por gerar SymLinks ao invés de reinstalar tudo (leia a doc)
  - Principalmente para desenvolvimento local, uma vez que CI/CDs podem não ter suporte a este instalador
- Utilizamos o [TypeScript](https://www.typescriptlang.org/) como linguagem de programação pelos motivos abaixo:
  - Em um time grande, a tipagem estática do TS ajuda a manter a ordem do repositório
  - Podemos criar interfaces tanto para os parâmetros de entrada quanto para os dados de saída e também para os retornos de APIs
  - Podemos, facilmente, alterar o alvo de nossas builds para sistemas mais antigos, mais novos e também utilizar propostas ainda não implementadas pelo TC39 através do `tsconfig.json` usando o Babel

## Arquivos de configuração

Todo projeto vai ter arquivos de configuração padrões, os chamados *dotfiles*, que são responsáveis por configurar nossas extensões e editores e devem ficar na raiz do projeto, juntamente com a pasta `.git`.

### Git Ignore

Todo projeto deve ter um `.gitignore` que pode ser gerado [neste endereço](https://www.gitignore.io/api/vim,node,linux,macos,windows,visualstudiocode) e complementado depois. Ele vai prover os principais arquivos de *ignore* para os principais IDEs que usamos e a linguagem Node.

### EditorConfig

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

### StandardJS

Para o standard, vamos instalar **sempre**, a biblioteca *StandardJS* como *Dev Dependency* no NPM através de `pnpm i -D standard`.

Todo o projeto também terá um script de lint que poderá ser acessado usando `pnpm run lint` ou `npm run lint` que, basicamente, irá rodar `standard`. O ideal será rodar este comando antes de dar push para o repositório. Porém, se o desenvolvedor se esquecer, o CI irá fazer o trabalho.

### .dockerignore

Arquivo **.dockerignore**:

```
dist
node_modules
.envrc
.envrc.sample
.gitignore
.editorconfig
.prettierrc
.vscode
```

### Dockerfile

O Dockerfile vai ser utilizado principalmente para construir as imagens de desenvolvimento, uma vez que as imagens de produção serão construídas no CI. Um exemplo de Dockerfile:

```Dockerfile
FROM node:10-alpine

ENV DEBUG expresso:*,projeto:*

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

## Install dependencies
COPY ["./package.json", "./shrinkwrap.yaml", "/usr/src/app/"]

RUN npx pnpm install

## Add source code
COPY ["./src", "./tsconfig.json", "/usr/src/app/"]

## Build
RUN npm run build:clean

EXPOSE 3000

CMD [ "npm", "start" ]
```

### docker-compose.yml

O Docker Compose será usado para construir o ambiente de desenvolvimento utilizando os dockerfiles criados no passo anterior, segue um exemplo:

```yaml
version: '3.7'

services:
  vhost:
    image: jwilder/nginx-proxy:alpine
    ports:
      - "80:80"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro

  mongodb:
    image: mvertes/alpine-mongo
    ports:
      - '27017:27017'
    volumes:
      - mongodb:/data/db

  backend:
    build: ./backend
    environment:
      DATABASE_MONGODB_URI: mongodb://mongodb:27017
      DATABASE_MONGODB_DBNAME: nomedobanco
      NODE_ENV: ${NODE_ENV:-development}
      VIRTUAL_HOST: api.projeto.127.0.0.1.nip.io
    depends_on:
      - vhost
      - mongodb

  frontend:
    build: ./frontend
    environment:
      NODE_ENV: ${NODE_ENV:-development}
      VIRTUAL_HOST: projeto.127.0.0.1.nip.io
      API_URL: http://backend:3000
    depends_on:
      - vhost
      - backend

volumes:
  mongodb:
```

### tsconfig.json

Exemplo de arquivo **tsconfig.json**

```json
{
  "compilerOptions": {
    /* Basic Options */
    "target": "es2015",
    /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */
    "module": "commonjs",
    /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
    "lib": ["es6", "es2015.promise", "es2015.iterable", "es2015.proxy"],
    /* Specify library files to be included in the compilation. */
    // "allowJs": true,                       /* Allow javascript files to be compiled. */
    // "checkJs": true,                       /* Report errors in .js files. */
    // "jsx": "preserve",                     /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
    "declaration": true,                   /* Generates corresponding '.d.ts' file. */
    // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
    // "sourceMap": true,                     /* Generates corresponding '.map' file. */
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    "outDir": "./dist",
    /* Redirect output structure to the directory. */
    "rootDir": "./src",                       /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "composite": true,                     /* Enable project compilation */
    "removeComments": true,
    /* Do not emit comments to output. */
    // "noEmit": true,                        /* Do not emit outputs. */
    // "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
    // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

    /* Strict Type-Checking Options */
    "strict": true,
    /* Enable all strict type-checking options. */
    "noImplicitAny": true,
    /* Raise error on expressions and declarations with an implied 'any' type. */
    "strictNullChecks": true,
    /* Enable strict null checks. */
    // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
    // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
    "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
    "alwaysStrict": true,
    /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    "noUnusedLocals": true,
    /* Report errors on unused locals. */
    "noUnusedParameters": true,
    /* Report errors on unused parameters. */
    "noImplicitReturns": true,
    /* Report error when not all code paths in function return a value. */
    "noFallthroughCasesInSwitch": true,
    /* Report errors for fallthrough cases in switch statement. */

    /* Module Resolution Options */
    // "moduleResolution": "node",            /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    // "baseUrl": "./",                       /* Base directory to resolve non-absolute module names. */
    // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
    // "typeRoots": [],                       /* List of folders to include type definitions from. */
    // "types": [],                           /* Type declaration files to be included in compilation. */
    // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
    "esModuleInterop": true,
    /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */

    /* Source Map Options */
    // "sourceRoot": "./",                    /* Specify the location where debugger should locate TypeScript files instead of source locations. */
    // "mapRoot": "./",                       /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

    /* Experimental Options */
    "experimentalDecorators": true,
    /* Enables experimental support for ES7 decorators. */
    "emitDecoratorMetadata": true,
    /* Enables experimental support for emitting type metadata for decorators. */
  }
}
```

### package.json

Este é um exemplo de um arquivo `package.json` modelo com pacotes iniciais:

```json
{
  "name": "gg",
  "version": "1810.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: No test specified\" && exit 0",
    "lint": "./node_modules/.bin/standard",
    "build": "rm -rf ./dist && ./node_modules/.bin/tsc",
    "build:watch": "./node_modules/.bin/tsc -w",
    "start": "node index.js"
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint"
    }
  },
  "engines": {
    "node": ">=8.11.0",
    "pnpm": ">=2.0.0",
    "npm": ">5.0.0"
  },
  "repository": {
    "type": "git",
    "url": "git@gitlab.nxcd.com.br:nxcd/gg.git"
  },
  "author": "Lucas Santos <lucas.santos@nxcd.com.br>",
  "license": "ISC",
  "devDependencies": {
    "@types/express": "^4.16.0",
    "@types/node": "^10.11.6",
    "husky": "^1.1.2",
    "standard": "^12.0.1",
    "typescript": "^3.1.1"
  }
}
```

## Testes

- Vamos utilizar o [Mocha](http://npmjs.com/package/mocha) como framework de testes
- Vamos utilizar o [Chai](https://www.npmjs.com/package/chai) como lib de asserção, porém o Chai não é uma lib obrigatória, você poderá usar o próprio [Assert](https://nodejs.org/api/assert.html) nativo do Node (preferível pois não adiciona outra dependência).