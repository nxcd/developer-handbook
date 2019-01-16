# Sobre processos de desenvolvimento

<!-- TOC -->

- [Sobre processos de desenvolvimento](#sobre-processos-de-desenvolvimento)
  - [Scrum](#scrum)
    - [Sprint Planning](#sprint-planning)
    - [Sprint Review](#sprint-review)
    - [Sprint Retrospective](#sprint-retrospective)
    - [Status de tarefas](#status-de-tarefas)
    - [DoD - Definition of Done](#dod---definition-of-done)
  - [Sobre versionamento](#sobre-versionamento)
    - [Funcionamento](#funcionamento)
      - [Fluxo de _bugfix_](#fluxo-de-_bugfix_)
      - [Fluxo de _hotfix_](#fluxo-de-_hotfix_)
      - [Changelogs](#changelogs)
- [Codinome da versão](#codinome-da-versão)
  - [- Descrição do bug 1 (#número-da-PR)](#--descrição-do-bug-1-número-da-pr)
    - [Convenções para números de versão](#convenções-para-números-de-versão)
      - [Patch](#patch)
      - [Minor](#minor)
      - [Major](#major)
      - [Exemplos](#exemplos)

<!-- /TOC -->

O processo de desenvolvimento do sistema é baseado no modelo ágil, utilizando a metodologia Scrum.

## Scrum

O [Scrum](https://www.desenvolvimentoagil.com.br/scrum/) é uma metodologia baseada em *Sprints*. Assim como todas as metodologias ágeis, ele possui um modelo base para ser seguido mas que deve ser adaptado e alterado de time para time e de empresa para empresa, inclusive, é comumente dito que: "se você está usando o Scrum conforme o livro, provavelmente ele não vai funcionar".

O Scrum possui duas figuras importantes:

- **O P.O**: Geralmente é uma pessoa ligada ao negócio **e (mas não só)** à tecnologia como um todo. Na maioria dos casos é um funcionário com este cargo especificamente, porém, no nosso caso, o mais indicado para que ocupe esse cargo é ou o Fabrício ou o Juliano, devido a proximidade de ambos com os clientes e o conhecimento das funcionalidades do sistema.
- **O Scrum Master**: Pode ser qualquer pessoa do time que esteja ou não esteja trabalhando no projeto. O objetivo desta figura é proteger o tempo do time e o processo geral. Garantir que todos estão entendendo o processo do Scrum e também garantir que todos estão trabalhando dentro da metodologia. Será ele que irá tirar dúvidas de processo e também guiar o time durante a sprint. Este cargo pode ser fixo ou então se alternar entre os membros do time.

Além das figuras de organização, o Scrum também tem os chamados "rituais", que são reuniões que acontecem em momentos do projeto. Para falar delas primeiramente precisamos entender o que é uma **sprint** e também o que é um **épico**:

- **Sprint**: É um intervalo de tempo que, geralmente, dura de 7 a 14 dias. Uma sprint muito curta faz com que não seja possível a inclusão de tarefas muito longas e uma sprint muito longa torna o processo de *deploy* difícil de gerenciar. Uma vez que a sprint está em andamento, **não poderão ser adicionadas novas tarefas à mesma**, apenas removidas.
- **Épico ou Estória**: Um épico é uma tarefa grande, por exemplo "Criar tela de login", que será quebrada em N tarefas menores pela equipe de forma que fiquem pequenas o suficiente para serem feitas por uma única pessoa e independente o suficiente para que seja separada do resto da lista. Este tipo de tarefa só pode ser criada pelo P.O. No nosso processo atual, **um épico é representado por um milestone**.

Quando falamos de reuniões "rituais" temos algumas principais.

### Sprint Planning

É o processo onde todos do time (incluindo o P.O e o Scrum Master) se reunem para realizar o planejamento da próxima sprint.

Ela geralmente acontece um dia depois da sprint anterior terminar (ou então no mesmo dia). Nesta reunião serão definidas todas as tarefas que serão passadas do *backlog* para tarefas ativas, é importante que as tarefas sejam bem especificadas e, na medida do possível, independentes umas das outras, de forma que todos do time possam realizar tarefas em paralelo ao invés de ter de esperar outros terminarem.

Uma planning **geralmente dura entre uma e três horas**, tentando ser o mais rápido possível.

Aqui será aonde definiríamos pesos para as tarefas (conforme o scrum diz) mas foi definido que não será utilizado esse modelo.

> Faremos as Plannings na quinta feira à tarde

### Sprint Review

A review é realizada no dia da finalização de uma sprint. Será nesta reunião que todos os membros vão apresentar o que foi feito para o P.O poder comunicar aos usuários.

Individualmente, todas as tarefas devem ser mostradas e testadas como se um usuário as estivesse usando. Esta reunião geralmente **demora cerca de 1h30min**. Caso alguma das tarefas não esteja pronta ou então com defeito, ela voltará para o *backlog* para ser repriorizada na próxima planning.

> Faremos as reviews na quinta-feira antes da próxima planning

### Sprint Retrospective

Esta é uma reunião interna do time, que pode ou não ser necessária. Em times pequenos ela geralmente não se faz muito importante, pois será aonde o time irá conversar sobre o processo em si, nesta reunião o P.O não precisa estar presente, mas a presença do Scrum Master é importante.

A reunião não pode durar mais de 1h e, nela, os membros vão escrever o que foi bom na última sprint e os pontos de melhoria. Desta forma o processo pode melhorar na próxima sprint.

O intervalo de execução dessas reuniões geralmente é definido como **uma vez a cada 2 sprints** ou então 2 vezes por mês, pois executar este ritual muitas vezes acaba sendo improdutivo e não trazendo muitos problemas, mas demorar a executar também cria um problema por conta dos membros do time não lembrarem de sprints anteriores.

### Status de tarefas

As tarefas no Scrum geralmente caem em 4 categorias distintas:

- **Backlog:** Aqui ficam todas as tarefas que podem ou não serem feitas. Todos os membros do time tem permissão de adicionar uma tarefa aqui, geralmente quem adiciona essas tarefas é o P.O nas chamadas **estórias ou épicos**.
- **To do:** Lista de tarefas que já foram retiradas do backlog e estão prontas para serem feitas **na sprint atual**. Tarefas de sprints anteriores não podem entrar no *to do* de uma nova sprint.
- **Doing:** Tarefas em progresso
- **Testing:** Tarefa com desenvolvimento finalizado mas ainda não testada, deverá ser testada **por uma pessoa que não desenvolveu a tarefa**
- **Blocked:** Tarefa que está finalizada (com ou sem testes) mas está bloqueada por algum motivo externo à equipe
- **Done:** Tarefas completamente finalizadas que vão ser arquivadas (de acordo com o DoD abaixo) ao final da sprint

### DoD - Definition of Done

Uma parte importante do Scrum é o chamado "Definição de pronto", para que o time possa entender o que é uma tarefa feita e o que ainda está por fazer. Nossa definição de pronto é:

> Uma task pronta é aquela que já está desenvolvida e testada, contém uma PR já mesclada para o branch `release/` (em casos de feature) ou para a `master` (em casos de hotfix).

## Sobre versionamento

Estaremos utilizando o [Git Flow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) (com um cheat sheet [aqui](https://danielkummer.github.io/git-flow-cheatsheet/)). Um modelo de organização do repositório que é suportado por diversos sistemas como: [SourceTree](http://www.sourcetreeapp.com/) e [GitKraken](https://www.gitkraken.com/), mas também com [implementações em linha de comando](https://github.com/nvie/gitflow).

### Funcionamento

Assim como o Scrum, vamos adaptar o modelo do Git Flow para funcionar de uma forma mais simples. Não vamos ter o branch `develop` e vamos fazer todos os desenvolvimentos no branch `release/nome-da-release`:

- Iniciamos somente com o branch `master`, dele clonamos um branch `release/<data-final-da-sprint>-<codename>`, de onde todos os demais branches `feature/nome-da-feature` serão clonados
- Ao iniciar uma nova feature, vamos clonar do branch `release/...` o branch `feature/nome-da-feature`
- Uma vez que a funcionalidade terminar, ela será novamente mesclada ao branch `release/...`
- Quando a sprint for terminada, vamos mesclar o branch `release/...` no branch master (**isso significa que vamos publicar o branch em produção**)
  - Aqui será aonde vamos adicionar, atualizar e corrigir os números de versão
  - Também vamos tagear a release com seu nome (o codename)
- Depois de mesclada, o branch da `release/...` é removido do GitLab e um novo branch `release/nome-da-proxima-release` é criado a partir da `master`

> O branch `master` será **sempre** o branch da versão que está em produção no momento

> **Funcionamento do Git Flow original**
>
> O git flow original funciona da seguinte maneira:
>
> - Iniciamos com um *branch* de desenvolvimento, normalmente chamado de `develop`, todo o desenvolvimento novo será mesclado neste branch.
> - A cada nova funcionalidade (ou **feature**), um novo branch é criado a partir do `develop` seguindo a nomenclatura > `feature/nome-da-feature`, é neste branch que todo o desenvolvimento será feito
> - Uma vez que uma funcionalidade é terminada, ela é mesclada de volta no branch `develop`
> - Quando a sprint termina, temos que criar uma nova release, para isso vamos mesclar o branch `develop` em um novo branch chamado > `release/nome-da-release`
>   - Neste processo será aonde vamos adicionar, atualizar e corrigir os números de versão
>   - Aqui será aonde vamos realizar a criação das tags para esta release
> - Depois de finalizar a criação da release vamos fazer o *shipping* da mesma. Para isto vamos mesclar o branch `release/nome-da-release` de > volta no branch `develop` e também no branch `master`.

#### Fluxo de _bugfix_

Segue o mesmo fluxo da feature, porém o nome da branch é `bugfix/` e não `feature/`. Deve **sempre** ser usado quando o branch não implementa nada de novo e também não será mesclado para a `master`, ele apenas resolve um problema.

#### Fluxo de _hotfix_

Além do fluxo padrão de desenvolvimento de novas funcionalidades, temos o fluxo de correção de bugs, as chamadas `hotfixes`.

Para isto, vamos seguir os passos:

- Criamos um novo branch `hotfix/descrição-do-fix` a partir da branch `master`, **e não mais da `release/...`**
- Realizamos a correção nesta branch
- Ao finalizar, mesclamos a branch nos branches `release/...` e `master`
  - Aqui vamos criar uma tag para o branch master informando a descrição do fix

Então a publicação segue normalmente.

#### Changelogs

**Sempre** ao finalizar uma sprint, quando formos lançar uma release nova para a master, teremos de fazer o *tagging* desta release. Este tagging deverá conter uma descrição que é um changelog, no modelo abaixo:

---
---
---
# Codinome da versão

:cactus: **Features:**

- Descrição da feature 1 (#número-da-PR)

:butterfly: **Melhorias:**

- Descrição da melhoria 1 (#número-da-PR)

:bug: **Bugs:**

- Descrição do bug 1 (#número-da-PR)
---
---
---

### Convenções para números de versão

Vamos usar o [Semantic Versioning](http://semver.org), mas com algumas alterações nas regras.

#### Patch

O número de Patch é destinado __somente__ aos deploys em produção fora do tempo da sprint, ou seja, bugs e hotfixes. Sempre que tivermos que publicar alguma coisa em produção __durante__, __antes__ ou __depois__ de uma release, ou seja, fora da data especificada de deploy, vamos incrementar este número.

Ele também deve ser incrementado __para todos os deploys individualmente__, por dois motivos:

- Isto ajuda a manter o _track_ da quantidade de hotfixes que tivemos entre uma sprint e outra
- Atualiza a versão na master para que o Kubernetes possa fazer o download novamente e não usar o cache do cluster

#### Minor

Será o número do dia do mês que a sprint terminou.

Ele será incrementado somente nos merges __da release com a master__. Por exemplo, no mês de __Junho de 2018__, durante a primeira sprint do mês o número será a data de deploy, digamos `12`. Assim acontecerá na segunda sprint do mês, onde o número será incrementado para a data de quando ela foi publicada.

#### Major

O Major da versão conterá __sempre__ 4 dígitos, sendo estes os dois últimos dígitos do ano + o mês com dois dígitos, por exemplo:

Uma versão do mes de __Junho de 2018__ seria: `1806.0.0`

A versão Major só é atualizada quando há uma virada de mês, quando isto acontece, a Minor e a Patch são __zeradas__.

#### Exemplos

- Uma build da sprint terminada em 20 de Julho de 2018: `1807.20.0`
- Uma build da sprint terminada em 18 de Agosto de 2018 com 10 hotfixes implementados: `1808.18.10`
