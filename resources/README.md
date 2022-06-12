# A documentação

Nessa documentação você vai encontrar tudo que pode precisar para construir aplicações para a web com Leme JS.

Logo a seguir está o passo a passo para dominar os recursos necessários para construir aplicações SPA.


    
## Estrutura do componente

Aplicações construídas com Leme JS podem fazer uso de componentes com os recursos abaixo:

- Componentes
    - propriedades
    - eventos
    - hooks
    - métodos
    - componentes filhos
    - estilos css (CSS in JS)

- Gerenciamento de estado local e global
    - Objetos observáveis
    - Store de dados

- Comunicação entre componentes
    - Por eventos
    - Por observadores

- Navegação
    - Rotas


### Funções como componentes

Para Leme JS componentes são apenas funções que recebem parametros e retornam objetos.

```javascript
// index.js

import { observableFactory } from 'lemejs'

import template from './template'
import styles from './styles'

export const appHelloWorld = ({ props }) => {

    const state = observableFactory({ title: 'Olá!' })

    return { state, template, styles }
}

```

Acima o arquivo principal, onde a lógica do componente deve ser escrita.

#### Template do componente

```javascript
//template.js

const template = ({ state, props, html }) => html`
    <h1>Hello, ${title}</h1>
`

```

Acima o arquivo de template, onde toda a estrutura html deve ser escrita.

#### CONTROLADOR DO COMPONENTE

```javascript
//styles.css

const styles = ({ ctx, props, css }) => css`
    ${ctx} h1 { color: blue }
`

```

Acima o arquivo de estilos css, onde toda a configuração visual deve ser escrita.

*Você pode estar pensando que essa quantidade de código é grande para um componente tão básico.  Mas, há um bom motivo para isso...* 

> *LEME JS não faz mágica, não esconde nada de você*.

#### 1 - NOME DO COMPONENTE

Componentes para *Leme JS* são apenas funções e por isso o nome de cada função de componente é muito importante, pois, é através desse nome que a tag html do componente será definida.

```javascript
export const appHelloWorld = ({ props }) => { ... }
```

É como se o nome da função no trecho acima fosse utilizado para criar a tag html do componente abaixo.

```html
<app-hello-world> ...conteúdo... </app-hello-world>
```

Perceba que enquanto o nome das função manipuladora é definido em ***camel Case***, o nome da tag do componente é definido utilizando ***kebab case*** ou seja, semparação por traços das palavras que formam a tag.

> *Esse comportamento é adotado para respeitar as regras do HTML5 sobre criação de tags personalizadas.*

#### 2 - ESTADO 

```javascript
//index.js

export const appHelloWorld = ({ props }) => {
    //código omitido...

    const state = observableFactory({
        title: 'Olá!'
    })

    return {
        state
        //código omitido...
    }
}
```
> Através da variável **state** o estado do componente é definido. 

Qualquer alteração nas propriedades do objeto de estado resultará na atualização do template do componente.

É graças a função fábrica ***observableFactory*** que retorna um objeto observável que se faz possível detectar alterações no estado da aplicação e reagir a elas.

O state é um objeto observável e sempre que modificado Leme JS renderiza o componente novamente.

#### 3 - TEMPLATE 

O template do componente é responsável por exibir os dados armazenados nas propriedades do state.

```javascript
//template.js

const template = ({ state, props, html }) => html`
    <h1>Hello, ${title}</h1>
`
```

Observe que a o template é uma função que recebe os parâmetros state, props e html e retorna um **tagged function** *html* com o valor da propriedade *title* presente no state.

> Acima html é um tagged function, se você não sabe muito sobre tagged functions ou literal templates vai gostar de ler mais sobre o assunto [aqui](https://css-tricks.com/template-literals/).

#### 4 - Propriedades estáticas

Os compnentes podem receber informação através de propriedades estáticas.

Essas propriedades são imutáveis e o componete as lê apenas uma vez durante a renderização.

Para definir propriedades de componentes é possível usar a função **toProps** que define o nome da propriedade e seus valores. Veja abaixo:

```javascript
//template.js
const template = ({ state, props, toProps, html }) => html`
    <app-user 
        ${toProps('profile', state.userProfile)}
        ${toProps('auth', state.auth)}
    >
        Hello, ${state.title}
    </app-user>
```

Acima, a função *toProps* foi utilizada para definir a propriedade *profile* e uma segunda propriedade *auth*.

Essas propriedades foram fornecidas ao componente app-user que poderá acessar as mesmas através de seu objeto props.

```javascript
const appUser = ({ props }) => {
    console.log(props)
    return { template, styles }
}

const template = ({ html, props }) => html`
    <p>${JSON.stringify(props)}</p>
`

const styles = ({ css, props }) => css`
    ${ctx} { 
        display: inline;
        color: ${props.profile.color}
    }

`
```

No exemplo acima, o componente appUser acessa as propriedades definidas anteriormente e que estão dipsoníveis para o seu controlador, template e estilos css.

#### 5 - ESTILOS CSS

A função **styles** é reponsável por definir e retornar os estilos css do componente.

```javascript
//styles.js

const styles = ({ ctx, props, css }) => css`
    ${ctx} h1 { color: blue }
    .ctx-title { color: red }
`
```

Observe que a função styles acima tem acesso aos parâmetros ***ctx, props*** e ***css***.

A variável **ctx** contém o seletor css do componente e deve ser utilizada para aplicar estilos ao elemento ***root** do próprio componente.

A variável ***ctx*** ainda pode ser combinada a classes com o prefixo ***.ctx-*** para indicar que o css respeitará o escopo do componente e não vasará para outros componentes.

> É recomendado usar sempre que possível o prefixo **.ctx-** nas classes de estilização.

O parametro **css** é um ***tagged function*** que tem por responsabilidade formatar aplicar o ***syntax highlight*** nos estilos css oferecendo melhor experiência de desenvolvimento.

No fim das contas será renderizado um css como o apresentado abaixo:

```html
<style>
    app-hello-world h1 { color: blue }
    app-hello-world_8981-18dd_title { color: red }
</style>
```


#### EVENTOS

O arquivo index.js que contém o controllador do componente abaixo foi refatorado para incluir eventos
de interação.

Observe o controlador *./index.js*
```javascript
//index.js

import template from './template'
import styles from './styles'

export const appHelloWorld = ( element ) => {

    const state = observableFactory({
        title: 'this is a simple component.'
    })

    const hooks = () => ({
       afterOnInit
    })

    const afterOnInit = ({on, queryOnce}) => {
        onClickTitle(on, queryOnce)
    }

     const onClickTitle = () => {
        const h1 = queryOnce('h1')
        on('click', h1, ({target}) => console.log(target))
    }

    return { 
        state, 
        template,  
        styles,
        hooks
    }
}
```

Observe acima que os eventos só podem ser adicionados a elementos html caso um hook seja definido.

Cada hook é disparado em momentos específicos do ciclo de vida do componente.

O momento ideal para adicionar eventos a elementos html é após a rederização do componente ou após a inicialização do mesmo.

Para adicionar eventos após a inicialização do componente é possível usar o hook ***afterOnInit*** e após a renderização ***afterOnRender***.

Hooks são apenas funções que podem acessar os métodos ***on, queryOnce e queryAll***.

**queryOnce**

- Encontra um elemento html especifico no componente
- retorna um elemento html

**queryAll** 

- Encontra todas as ocorrencias de um tipo de elemento html dentro do componente.
- returna um array de elementos html

**on** 

- Adiciona um evento a um elemento html para executar uma função.
- O primeiro parâmetro recebido deve ser o nome do evento
- O segundo parâmetro deve ser um elemento ou um array de elementos html
- O terceiro parâmetro deve ser a função 


#### Hooks

Atualmente os hooks são quatro funções que podem ser executadas, cada uma, em um momento específico do ciclo de vida do componente.

- beforeOnInit - Executada um única vez antes da inicialização do componente
- afterOnInit - Executada um única vez depois da inicialização do componente
- beforeOnRender - Executada uma única vez antes da renderização do componente toda vez que é renderizado
- afterOnRender - Executada um aúnica vez depois da renderização do componente toda vez que é renderizado

```javascript
const appHelloWorld = ({ props }) => {

    //código omitido
    const hooks = ({ methods }) => ({
        beforeOnInit,
        afterOnInit,
        beforeOnRender,
        afterOnRender,
    })

    const beforeOnInit = () => {}
    const afterOnInit = () => {}
    const beforeOnRender = () => {}
    const afterOnRender = () => {}

    return {
        hooks
        //código omitido
    }
}
```

Como os hooks também tem acesso aos métodos do componente, pode ser interessante a partir desses hooks, executar operações assincronas como requisições HTTP antes ou após a inicialização do componente.

Também pode ser interessante executar validadores e filtros de dados sempre que o componente é renderizado através dos hooks beforeOnRender e afterOnRender.

Apesar de poucos, os hooks combinados a objetos observáveis são mais que suficientes para tratar a reatividade de qualquer aplicação.

Os **hooks** se combinados com **objetos observáveis** possibilitarão que uma infinidade de ações possam acontecer de forma simples e clara.

### GERENCIAMENTO DE ESTDO LOCAL
O controller - *./index.js*

```javascript

import template from './template'
import styles from './styles'

export const appHelloWorld = ( element ) => {

    const state = observableFactory({
        title: 'Olá!'
    })

    return { state, template,  styles  }
}
```
Na definicação de componente acima, o gerenciamento de estado local é definido pelo trecho em destaque logo a seguir:

```javascript
    const state = observableFactory({
        title: 'Olá!'
    })
```

É dessa forma que deve ser definido o gerenciamento de estado local. Pois, *observableFactory* retorna um objeto observável com propriedades especiais que podem ser utilizadas tanto para atualizar o *state* do componente como para observar atualizações e reagir a elas de alguma forma.

Abaixo a lista de propriedades retornadas por *observableFactory*.

* set - Uma função para atualizar o state
* get - Uma função para ler o state
* on  - Uma função para adicionar observadores
* off - Uma função para remover observadores

A próxima seção dessa documentação, aborda a utilização dessas propriedades.

#### Atualizando o state

```javascript
const state = observableFactory({
    title: 'Default title value'
})

state.set({
    title: 'New title value'
})
```

Observe que acima a função **set** que é uma propriedade especial do **state** recebe como parâmetro um objeto contendo o novo valor da propriedade **title** presente no state. É dessa forma que o state local deve ser atualizado.

#### RECUPERANDO INFORMAÇÕES DO STATE

```javascript
const state = observableFactory({
    title: 'Default title value'
})

// Retorna um objeto estático com as propriedades do state
const newStaticState = state.get() 

// Recupera uma propriedade estática do state
const { title } = state.get() 
```

Observe acima que a função **get** é uma propriedade especial do **state** e é utilizada para recuperar o estado do componente por completo. Mas, caso alterações nas propriedades de *newStaticState* ocorram, essas alterações não serão refletidas no estado original e não desencadearão efeitos colaterais que atualizam o template do componente.

A função **get** pode ser utilzada para recuperar propriedades estáticas específicas do state, bastando informar o nome das propriedades dentro de chaves como no exemplo acima onde a propriedade estática **title** foi extraída do *state* através da função *get*.

As propriedades específicas recuperadas também não desencadeiam efeitos colaterais caso seus valores sejam alterados.

#### OBSERVANDO ALTERAÇÕES NO ESTADO

```javascript
const state = observableFactory({
    title: 'Default title value'
})

const onStateChange = state.on((newPayload) => {
    console.log(newPayload)
})

const onUpdateState = state.on(logger)

logger (value) { console.log(value) }
```

No exemplo acima, **on** foi utilizada para adicionar funções como observadoras para mudanças no estado. 

Quando uma alteração ocorrer em uma das propriedades do estado, as funções observadoras serão executadas.

No primeiro caso uma função anônima foi adicionada como observadora. Já no segundo caso um método do componente foi adicionado.

Observe que para ambos os casos, a referência para o observador adicionado é retornada e armazenada em uma variável para que posteiormente essa variável possa ser utilizada para desligar os observadores caso necessário. Essas referências são:

> onStateChange e OnUpdateState

#### REMOVENDO OBSERVADORES

```javascript
const state = observableFactory({
    title: 'Default title value'
})

const onStateChange = state.on((newPayload) => {
    console.log(newPayload)
})

state.off(onStateChange)
```

Para remover um observador basta informar a referencia deste para a função especial **off**.
No código logo acima temos um exemplo claro desse comportamento.


### Compartilhando o state

Uma maneira simples e eficaz para compartilhar o state entre componentes é através de objetos observáveis.

* **Compartilhado para baixo**

Primeiro um serviço mensageiro pode ser criado como no exemplo abaixo onde messengerDown envia informações para componentes filhos e messengerUp ao componente pai.

```javascript
//messenger.service.js

import { observableFactory } from "r9x_js";

export const messengerUp = observableFactory({})
export const messengerDown = observableFactory({})
```

No componente pai então o serviço mensageiro é importado e messengerDown define as informações a serem enviadas aos componentes filhos.

Ainda no componente pai, messengerUp é configurado para escutar alterações que devem gerar efeitos colaterias no componente pai através da função **messengerUp.on** que adiciona uma função como manipulador a ser executado quando uma mensagem de dados for enviada ao componente pai.

> Receber informações de componentes filhos pode ser interessante por exemplo para validação de formulários onde uma operação só pode ser executada caso todos os campos do formulário estejam validados.

```javascript
//appHelloWorld/index.js

import { messengerDown, messengerUp } from '../services/messenger.service.js'
import { appName } from '../componentes/appName'

const appHelloWorld = ( {props} ) => {
    //código omitido

    const state = observableFactory({
        name: 'Jhon'
    })

    const template = ({ html }) => html`
        <h1>
            Hello, <app-name></app-name>
        </h1>
    `

    const children = () => ({
        appName
    })

    const hooks = () => ({
        beforeOnInit
    })

    const beforeOnInit = () => {
        messengerDown.set({ ...state.get() })
        messengerUp.on((payload) => state.set({ ...payload }))
    }    

    return {
        state,
        template,
        children,
        hooks
    }
}
```

O exemplo acima é de como se parece um **componente pai** compartilhando o state com o componente filho. Mesmo assim, o componente filho se parece muito com o componente pai. Veja abaixo:

* **compartilhando para cima**

```javascript
//appName/index.js

import { messengerDown, messengerUp } from '../services/messenger.service.js'

const appName = ({props}) => {
    //código omitido

    const state = observableFactory({
        name: ''
    })

    const template = ({ state, html }) => html`
        <span>${state.name}!</span>
    `

    const hooks = () => ({
        beforeOnInit
    })

    const beforeOnInit = () => {
        messengerUp.set({ title: 'Other title...'})
        messengerDown.on((payload) => updateState(payload))
    }

    const updateState = (payload) => {
        state.set({ ...payload })
    }

    return {
        state,
        template,
        hooks,
    }
}
```

A semelhança é perceptível. No entando, enquanto o componente pai utiliza o método messengerUp.on para escutar notificações e reagir a elas, o componente filho utiliza messengerUp.set para enviar notificações de dados ao componente pai.

No caso em que o componente pai utiliza o método messengerDown.set para enviar notificações contendo dados ao componente filho, o componente filho utiliza o método messegerDown.on para escutar essas notificações de dados e reagir a elas.

* No componente pai

```javascript
/*appHelloWorld/index.js*/

const beforeOnInit = () => {
    messengerDown.set({ ...state.get() })
    messengerUp.on((payload) =>  state.set({ ...payload }))
}
```

* No componente filho:

```javascript
/*appName/index.js*/

const beforeOnInit = () => {
    messengerUp.set({ anyData: ''})
    messengerDown.on((payload) => updateState(payload))
}
```

### COMUNICAÇÃO ENTRE COMPONENTES

A comunicação entre componentes pode ocorrer entre compontes pai e filho e componentes irmãos.

* **Componente pai:** É o componente que contém outros componentes aninhados.
* **Componente filho:** É o componente aninhado dentro de outro componente.
* **Componentes irmãos:** São componentes aninhados em um mesmo componente pai.

> O tópico anterior **[Compartilhando o state](?id=compartilhando-o-state)** tratou da comunicação
> entre componente pai e filho. Caso não o tenha lido, fique a vontade para fazer isso agora. 

O compartilhamento de dados entre componentes irmãos ocorre da mesma forma que com componentes pai e filho.

Abaixo como componentes irmãos podem se comunicar por objetos observáveis.

* Componente A

```javascript
import { messengerA, messengerB } from '../services/messenger'

const beforeOnInit = () => {
    messengerB.set({ anyData: 'Any text...'})
    messengerA.on((payload) => updateState(payload))
}
```
* Componente B

```javascript
import { messengerA, messengerB } from '../services/messenger'

const beforeOnInit = () => {
    messengerA.set({ otherData: 'Other text...'})
    messengerB.on((payload) => updateState(payload))
}
```

Observe que cada componente tem seu próprio serviço de mensagem assim como na 
comunicação entre componente pai e filho. Porém, como estão aninhados no mesmo
nível, seus serviços mensageiros recebem um sufixo para identificar a que componente
cada gerenciador de mensagem pertence.  

Dessa forma messengerA pertence ao componente A e deve ser utilizado por outros 
componentes para enviar mensagens ao componente A.

De forma identica o messenger B pertence ao componente B e deve ser utilzado por 
outros componentes para enviar mensagens ao componente B.

Observe que no trecho de código abaixo está acontecendo exatamente assim.

* Componente A

```javascript
    messengerB.set({ anyData: ''})
    messengerA.on((payload) => methods.updateState(payload))
```
* Componente B

```javascript
    messengerA.set({ anyData: ''})
    messengerB.on((payload) => methods.updateState(payload))
```

>O componente B faz uso de **messengerA.set** para enviar uma mensagem ao componente A e
>usa **messegerB.on** para ouvir mensagens enviadas a partir componente A.

Dessa forma, qualquer componente que precisar se comunicar com o componente A deve importar
seu serviço de comunicação. Isso torna fácil o gerenciamento de mensagens e deixa claro no
código qual a intenção dos componentes que estão trocando informações.

#### COMPARTILHAMENTO POR EVENTOS

Os componentes também podem se comunicar através de eventos. Esses eventos são baseados no padrão pubsub e sua implementação se parece bastante com a implementação de objetos observáveis.

1. **Criando o serviço de mensagem**

Para criar o serviço de gerenciamento de eventos é necessário importar a fábrica do objeto pubsub, criar um objeto com a fábrica e exportar esse objeto, exatamente como no código abaixo.

```javascript
//services/eventDrive/index.js

import { pubsubFactory } from 'lemejs'
export const eventDrive = pubsubFactory()
```

2. **Usando o serviço nos compoentes**

Como pode observar, para usar o seviço, basta importá-lo.

Uma boa maneira para começar a usá-lo é se inscrevendo para ouvir eventos disparados por outros componentes combinando o uso do ***eventDrive*** com o hook ***beforeOnInit*** exatamente como no código abaixo.

```javascript
//component/anyCompoent/index.js

import { eventDrive } from '../../services/eventDrive'

//código omitido

const beforeOnInit = () => {
    eventDrive.on('onIncrement', (payload) => {
        console.log(payload)
    })            
}

//código omitido
```

No trecho em destaque abaixo, ***eventDrive.on*** faz com que o componente se registre para ser notificado de quando um evento ***onIncremente*** é disparado por outro componente. Dessa forma, quando esse evento ocorrer o componente que se registrou executa o callback/handler passado como segundo parâmetro para eventDrive.on e pode realizar as operações necessárias com os dados recebidos através do callback fornecido ao evento.

> Para adicionar um listener em eventos, **eventDrive.on** recebe 2 parâmetros. Sendo o primeiro uma string contendo o nome do evento e o segundo um callback/handler que recebe um único parâmetro.

3. **Nomes de eventos**

Você pode criar quaisquer nomes para eventos, mas, é recomendado a convenção de prefixar o nome do evento com **on** e utilizar **camelCase** na junção de direntes palavras.

Exemplos:

```javascript
// Evento 1
eventDrive.on('onIncrement', (payload) => {
    console.log(payload)
})  

// Evento 2
eventDrive.on('onChangeTitle', (payload) => {
    console.log(payload)
})  

// Evento 3
eventDrive.on('onGetData', (payload) => {
    console.log(payload)
})  
```


### Gerenciamento de estado global

Aplicações maiores podem precisar de um formato mais flexível e efetivo na comunicação entre componentes de diferentes níveis. Nesse caso uma **store** de dados pode ser implementada para gerir o estado da aplicação tornando a comunicação entre os componentes mais simples.

Implementar uma **store** traz diversos benefícios:

- Simplifica a comunicação entre componentes.
- Diminui a necessidade de objetos observaveis (messengerUp, messengerDown).
- Diminui a necessidade de usar emissores de eventos (eventDrive).
- Permite navegar através do histórico de modificações do state da aplicação.
- Extrai a lógica de modificação de dados de dentro dos componentes e a reduz.
- Ruduz da quantidade de código na aplicação de modo geral.

#### Criando uma store

Primeiro é preciso definir o local onde a store existirá, pois, dentro do diretório
da store devem ser criados 3 arquivos ao menos.

```
./src/store
        |-state
        |   -index.js
        |-mutations
        |   -index.js
        |-index.js            
```

Você pode criar a pasta da store onde preferir, mas, é recomendado que seja dentro da pasta
**./src** da aplicação para facilitar a organização do código.

Pode começar criando a pasta com o nome **store** que é bem sugestivo.

Depois de criar a pasta store, dentro dela é necessário criar as pastas state e mutations e 
um arquivo index.js dentro de cada pasta.

Por fim, será necessário criar um terceiro arquivo index.js na raiz da pasta store.

1. **Definindo o state**

O state nesse caso é global, por tanto, deve concentrar todos os dados que serão compartilhados entre os componentes da aplicação.

Considere um aplicativo de gerenenciamento de usuários formado ao menos por 2 componentes:
    * Cadastro de usuários
    * Listagem de usuários

A store de dados deve ser estruturada com base nos dados consumidos por componentes da aplicação. Logo o state se deve se parecer como no código abaixo:

```javascript
/*src/store/state/index.js*/

export const state = {
    userList: [
        {userId: 1, userName: 'Jhon', userEmail: 'jhon@email.com'},
        {userId: 2, userName: 'Bill', userEmail: 'bill@email.com'},
        {userId: 3, userName: 'Anie', userEmail: 'anie@email.com'},
    ]
}
```

2. **Definindo mutações**

As mutações são funções que podem modificar o state e é dentro das mutações que a lógica de 
atualização deve ser escrita.

As **mutations** são com as funções abaixo:

```javascript
/*src/store/mutations/user/index.js*/

const addUser = (state, payload) => {}
const removeUser = (state, payload) => {}

export const userMutations = { 
    addUser, 
    removeUser 
}
```

Como pode ver, acima foram definidas 2 funções **addUser** e **removeUser** e essas funções
foram declaradas dentro do arquivo index.js na pasta **user** no diretório das mutations. Assim,
o código fica organizado e sempre que uma alteração relativa a usuários se fizer necessária,
as mutations de usuários serão encontradas facilmente.

Observe que cada uma das mutations recebem 2 parâmetros, sendo o primeiro o state global e o segundo a carga de dados a ser criada, alterada, ou removida do state.

```javascript
/*src/store/mutations/user/index.js*/

const addUser = (state, payload) => {
    const userList = [...state.userList, payload.newUser]
    return { ...state, userList }
}

const removeUser = (state, payload) => {
    const userList = state.userList.filter( user => {
        if(user.userId !== payload.userId) return user
    })
    return { ...state, userList }
}

export const userMutations = { 
    addUser, 
    removeUser 
}
```

Acima o código completo das mutations para adicionar e remover usuários da store.

Oberserve que cada **mutation** retorna uma mutação do estado, essa mutação é um objeto contendo todas as alterações do state.

As mutations modificam o estado e retornam a nova representação dele.

Para ter a store completa, é necessário definir o state e seus atributos e as mutatios que o modificarão.

3. **Fabricando a store**

O arquivo index.js na raiz da store deve se parecer com isso:

```javascript
/*src/store/index.js*/

import { state } from './state'
import { userMutations } from './mutations/user'    
```

Primeiro, importa-se state e o módulo de mutações. Na sequência, as mutações podem ser agrupadas 
como no trecho em destaque:

```javascript
/*src/store/index.js*/

import { userMutations } from './mutations/user'

const mutations = {
    ...userMutations,
}
```

Por fim, a store é fabricada através da função **storeFactory** que completa se parece com o código abaixo:

```javascript
/*src/store/index.js*/

import { storeFactory } from 'lemejs'

import { state } from './state'
import { userMutations } from './mutations/user'

const mutations = {
    ...userMutations,
}

export const store = storeFactory({
    state,
    mutations
})
```

#### INTEGRANDO A STORE EM COMPONENTES

Para utilizar a store de dados em componentes basta importá-la e definir o state do componente.

```javascript
//componentes/userCreate/index.js

import { observerFactory } from 'lemejs'
import { store } from '../../store'

const appUserCreate = ({props}) => {
    const state = observerFactory({
        ...store.getState()
    })
}
```

Na sequência pode ser necessário implementar alguma ação para executar a mutation apropriada e realizar a operação necessária.

No caso do cadastro de usuários, a mutation a ser executada deve ser **addUser**.

```javascript
//componentes/appUserCreate/index.js

import { store } from '../../store'

const appUserCreate = ({props}) => {

    const state = {
        ...state.getState(),
        user: {}
    }

    const hooks = () => ({
        afterOnInit
    })

    const afterOnInit = ({on, queryOnce}) => {
        onClickToAddUser(on, queryOnce)
    }

    const onClickToAddUser = (on, queryOnce) => {
        const buttonSave = queryOnce('button')
        on('click', buttonSave, addUser)
    }

    const addUser = () => {

        const newUser = {
            userId: 4, 
            userName: 'Eric', 
            userEmail: 'eric@email.com'
        }

        store.emit('addUser', newUser)
    }
}
```

Conforme o código acima, quando o botão adicionar usuário for clicado, a função addUser será executada e a store emit uma ação **addUser** e transmite os dados do novo usuário através dessa.Isso faz com que a mutation com o mesmo nome da ação seja executada realizando a inclusão do novo usuário na lista de usuários.

Aproveitando o evento de inclusão é possível fazer com que o formulário do componente de cadastro seja apagado por completo para facilitar um novo cadastro e também é possível fazer com que a listagem de usuários seja atualizada no outro componente.

* **Limpando o formulário de cadastro**

Para limpar o formulário de cadastro basta apagar as informações do usuário no state local do componente de cadastro.

É completamente possível tirar proveito dos hooks para ouvir ações da store e reagir a elas. Então, tirando proveito disso o state local pode ser apagado.

```javascript
//componentes/appUserCreate/index.js

import { store } from '../../store'

const appUserCreate = ({props}) => {

    const state = {
        ...store.getState(),
        user: {}
    }

    const hooks = () => ({
        afterOnInit
    })

    const beforeOnInit = (on, queryOnce) => {

        store.on('addUser', (payload) => {
            const user = {}
            state.set({ user })
        })

        onClickToAddUser(on, queryOnce)
    }
    
    const onClickToAddUser = (on, queryOnce) => {
        const buttonSave = queryOnce('button')
        on('click', buttonSave, addUser)
    }     

    const addUser = () => {

        const newUser = {
            userId: 4, 
            userName: 'Eric', 
            userEmail: 'eric@email.com'
        }

        store.emit('addUser', newUser)
    }
}
```

Observe o trecho em destaque:


```javascript
const hooks = () => ({
    beforeOnInit
})

const beforeOnInit = () => {
    store.on('addUser', (payload) => {
        const user = {}
        state.set({ user })
    })
}    
```

No trecho em destaque, através do hook **beforeOnInit** que é executado uma única vez antes
do componente inicializar, foi adicionado um ouvinte da store que será executado sempre que
a action addUser for disparada. Assim, toda vez que um novo usuário for adicionado, o state
local do componente e o formulário de cadastro de usuários serão apagados.

* **Atualizando a lista de usuários**

```javascript
//componentes/appUserCreate/index.js

import { store } from '../../store'

const appUserCreate = ({props}) => {

    const { userList } = store.getState()

    const state = {
        ...store.getState(),
        userList
    }

    const beforeOnInit = () => {
        store.on('addUser', addUser)
    }  

    const addUser = ({ user }) => {
        state.set({ user })
    }
}
```

No código acima, através do hook beforeOnInit foi adicionado um ouvinte da store através do método ***store.on*** e sempre que a action **addUser** for executada a função **addUser** também é executada atualizando o state local do componente o que força o template do mesmo a ser atualizado com a nova lista de usuários.

Ainda resta uma última ação necessária. A remoção de usuários.

```javascript
//componentes/appUserCreate/index.
    
import { store } from '../../store'

const appUserCreate = ({ props }) => {

    const { userList } = store.getState()

    const state = {
        title: 'User list',
        userList
    }

    const hooks = () => ({
        beforeOnInit
    })
    
    const beforeOnInit = ({on, queryAll}) => {
        store.on('addUser', addUser)
        store.on('removeUser', updateUserList)

        onClickToRemoveUser(on, queryAll)
    }   
    
    const onClickToRemoveUser = (on, queryAll) => {
        const buttonRemove = queryAll('button')
        on('click', buttonRemove, removeUser)
    }   

    const addUser = ({ user }) => {
        state.set({ user })
    }

    const removeUser = ({ target }) => {
        const id = +target.dataset.id
        store.emit('removeUser', { id })
    }

    const updateUserList = ({ userList }) => {
        state.set({ userList })
    }  

}
```

Primeiro foi definido um ouvinte para a action **removeUser** da store e esse ouvinte executa o
método do componente **updateUserList** que por sua vez atualiza a lista de usuários no state
local do componente forçando o template do mesmo a exibir a lista de usuários atualizada.

```javascript
store.on('removeUser', updateUserList)

const updateUserList = ({ userList }) => {
    state.set({ userList })
}      
```

Na sequência foi adicionado um evento de clique no usuário a ser removido e sempre que o botão
remover usuário é clicado a action **removeUser** é disparada o que executa a mutation de mesmo
nome, que remove o usuário identificado através do **id** fornecido.

```javascript

const hooks = ({on, queryAll}) => ({
    onClickToRemoveUser(on, queryAll)
})   

const onClickToRemoveUser = (on, queryAll) => {
    const buttonRemove = queryAll('button')
    on('click', buttonRemove, removeUser)
}    

const removeUser = ({ target })=> {
    const id = +target.dataset.id
    store.removeUser({ id })
}    

```

Dessa forma, sempre que um usuário for removido o componente será renderizado novamente e a lista 
de usuários será atualizada e exibida novamente.


## Navegação e Rotas

Atualmente a sistema de navegação de **lemejs** é baseado em **hash (#)** e expressões regulares
que havaliam o hash atual para decidir que view será exibida.

```javascript
//routes/index.js

import { routerFactory } from "lemejs"    

import {appHome} from './components/appHome'
import { appNotFound } from './components/appNotFound'
import { appOther } from './components/appOther'

const router = routerFactory()

router.add({
    hash: '/',
    validator: /^#\/$/,
    component: appHome,
    isInitial: true
})

router.add({
    hash: '/other',
    validator: /^#\/other$/,
    component: appOther,
})

router.add({
    hash: 'not-found',
    validator: /^#\/not-found$/,
    component: appNotFound,
    isDefault: true
})

export { router }    

```

Como pode ver acima, as rotas da aplicação são definidas através de um objeto ***router***.

* **Propriedades de rotas**
    - hash - O hash a ser definido na url
    - isInitial - Primeira rota a ser carregada na aplicação
    - isDefault - Rota carregada no caso de hash inválido ou rota inexistente
    - validator - Expressão regular usada para havaliar o hash
    - component - Componente/view a ser carregado caso a rota seja valido

>Observe que na lista de rotas acima, a segunda rota ***other*** não possui propriedades especiais como isDefault ou is Initial. Isso significa que será carregada apenas quando um hash correspondente for encontrado na url.

* **Expressão regular de rota**

```javascript
{ validator: /^#\/$/, component: appHome },
{ validator: /^#\/not-found$/, component: appNotFound },
```

No trecho acima, a primeira expressão regular ```/^#\/$/``` valida para **true** o hash ```#/``` que muitas vezes representa a página inicial de uma aplicação.

Já a expressão ```/^#\/not-found$/``` valida para **true** caso o hash seja ```#/not-found```.

No caso em que o hash atual difere de todas as rotas definidas, o roteador redireciona a aplicação para um hash definido como padrão.

O hash padrão é definido através da propriedade ***isDefault:true***.

O component fornecido para a rota com a propriedade **isDefault** é carregado após o redirecionamento para o hash padrão.

```javascript
{
    hash: 'not-found',
    validator: /^#\/not-found$/,
    component: appNotFound,
    isDefault: true
}
 ```

 * **Renderizando uma rota**

 Para rederizar uma rota é necessário utilizar o componente ```<router-view></router-view>``` do próprio sistema de rotas. No entanto não é mesmo nem necessário importá-lo. Basta inserir a tag do componente de rotas no local desejado e as rotas serão renderizadas.

 Você também pode aplicar estilos globais para controlar o comportamento visual desse componente já que a esse componente não é aplicado nenhum comportamento visual padrão definido através do css.

```javascript
//componentes/appUserCreate/index.js
  
const appMain = (element) => {

    const state = {
        title: 'Main'
    }

    const template = ({state, html}) => html`
        <h1>${state.title}</h1>
        <router-view></router-view>
    `
}
```

> É comum que as views definidas pelo sistema de rotas sejam renderizadas no componente principal como acima.

Isso é tudo que há para saber sobre o sistema de rotas de **Leme Js**.

## Próximos passos

Agora que já conhece mais sobre Leme JS hora de seguir o tutorial, por em prática todos os conceitos vistos até aqui e desenvolver uma aplicação reativa.

- [Seguir o tutorial](/tutorial)