# A documentação

Nessa documentação você vai encontrar todas as informações que pode precisar sobre Leme JS.

Logo a seguir a documentação apresenta passo a passo cada um dos recursos que você pode querer utilizar na hora de construir suas aplicações SPA.

    
## Estrutura 

Aplicações construídas com LEME JS podem fazer uso de recursos como:

- Componentes
    - eventos
    - ganchos
    - métodos
    - diretivas

- Gerenciamento de estado local
    - state
    - compartilhando dados para baixo
    - compartilhando dados para cima

- Comunicação entre componentes
    - Por eventos
    - Por observadores

- Gerenciamento de estado global
    - store de dados

- Navegação
    - Rotas


### Componentes

Para LEME JS componentes são apenas funções que recebem parametros e retornam objetos.

Acompanhe abaixo a estrutura básica de um componente.

```javascript

export const appHelloWorld = (element) => {

    const template = ({ state, html }) => html`
        <h1>Hello, ${title}</h1>
    `

    const styles = ({ ctx, css }) => css`
        ${ctx} h1 { color: blue }
    `

    const state = observableFactory({
        title: 'this is a simple component.'
    })

    return {
        state,
        template,
        styles
    }
}

```

*Você pode estar pensando que essa quantidade de código é grande para um componente tão básico.* 

Bem, a verdade e que *LEME JS* precisa de mais código para criar componentes do que frameworks e biliotecas famosas. Mas, há um bom motivo.

> **LEME JS não faz mágica, não esconde nada de você**.

O componente poderia ser refatorado facilmente o que diminuiria a quantidade de código. Por agora, 
é importante entender a responsabilidade de cada um dos trechos do código acima. Mas, logo a frente
o componente será refatorado o que dará a impressão de que ele é bem menor.

#### 1 - Nome do componente

Componentes para *LEME JS* são apenas funções e por isso o nome de cada função de componente é muito importante, pois, é através desse
nome que a tag html do componente será relacionada a sua função manipuladora.

```javascript
export const appHelloWorld = (element) => { ... }
```

É como se o nome da função no trecho acima fosse utilizado para criar a tag html do componente abaixo.

```html
<app-hello-world> ...conteúdo... </app-hello-world>
```

Perceba que enquanto o nome das função manipuladora é definido em camelCase, o nome da tag do componente 
é definido utilizando kebab case que separadas por traço as palavras que formam a tag.

> **Esse comportamento é adotado para respeitar as regras do HTML5 sobre criação de tags personalizadas.**

#### 2 - Estado do componente

```javascript

export const appHelloWorld = (element) => {
    //código omitido...

    const state = observableFactory({
        title: 'this is a simple component.'
    })

    return {
        state
        //código omitido...
    }
}
```
> Através da variável **state** o estado do componente é definido. 

Alterações em qualquer uma das propriedades em ***state***, resultará em um efeito
colateral que atualizará o template exibindo a alteração realizada através do mesmo.

Graças a função fábrica ***observableFactory*** que fabrica um objeto observável e o retorna
a variável ***state*** armazenado esse objeto observável.

O ***state*** então é retornado pela função fabrica manipuladora ***appHelloWorld***  para que ***LEME JS*** 
se encarregue de atualizar o template com a informação nova inserida ou carregada no state.

#### 3 - Template do componente

O template do componente é responsável por exibir os dados armazenados nas propriedades do state do componente.

```javascript

export const appHelloWorld = (element) => {

    const template = ({ state, html }) => html`
        <h1>Hello, ${title}</h1>
    `
    //código omitido...

    return {
        template,
        //código omitido...
    }
}

```

Observe que a função template é um mero **literal template** que recebe os parâmetros state e html e
retorna um **tagged template** *html* com o valor da propriedade *title* presente no state.

> Acima html é um tagged function, se você não sabe sobre tagged functions ou literal templates vai gostar
> de ler mais sobre o assunto [aqui](https://css-tricks.com/template-literals/).

#### 4 - Estilos CSS

A função **styles** é reponsável por definir e retornar os estilos css do componente.

```javascript
export const appHelloWorld = (element) => {

    //código omitido
    const styles = ({ ctx, css }) => css`
        ${ctx} h1 { color: blue }
    `

    return {
        //código omitido
        styles
    }
}
```

Observe que a função styles acima extrai 2 parâmetros sendo o primeiro **ctx** o escopo
do componente e o segundo **css** um tagged function.

A variável **ctx** deve ser utilizada para aplicar escopo ao css evitando o vazamento do 
mesmo para outros componentes.

> É recomendado o uso de **ctx** como boa prática.

A variável **css** tem a função de formatar corretamente o código oferecendo melhor experiência
de desenvolvimento e recursos como highlight syntax.

No fim das contas será renderizado algo como:

```html
    <style>
        app-hello-world h1 { color: blue }
    </style>
```

> Agora que você já compreende a estrutura básica de um componente LEME, o componente será separado
em 3 partes, template, style e controller.

O template -  *./template.js*
```javascript

    export default ({ state, html }) => html`
        <h1>Hello, ${title}</h1>
    `
```
    
Os estilos CSS - *./styles.js*
```javascript

    export default ({ ctx, css }) => css`
        ${ctx} h1 { color: blue; }
    `
    
```

O controller - *./index.js*
```javascript

import template from './template'
import styles from './styles'

export const appHelloWorld = ( element ) => {

    const state = observableFactory({
        title: 'this is a simple component.'
    })

    return { state, template,  styles  }
}
```

> Com esse novo formato, o componente parece bem menor e cada arquivo trata de uma única responsabilidade.


#### eventos

O arquivo index.js que contém o controllador do componente abaixo foi refatorado para incluir eventos
de interação.

O controller - *./index.js*
```javascript

import template from './template'
import styles from './styles'

export const appHelloWorld = ( element ) => {

    const state = observableFactory({
        title: 'this is a simple component.'
    })

    const events = ({on, queryOnce, queryAll}) => ({
        onClickTitle () {
            const h1 = queryOnce('h1')
            on('click', [h1], ({target}) => console.log(target))
        }
    })

    return { 
        state, 
        template,  
        styles,
        events  
    }
}
```

Observe acique que a fábrica de eventos **events** tem acesso aos manipuladores DOM **on, queryOnce e queryAll**. Cada um dos 
manipuladores possuem responsabilidades específicas:

**queryOnce**

- Encontra um elemento html especifico no componente
- retorna um element html

**queryAll** 

- Encontra todas as ocorrencias de um tipo de elemento html dentro do componente.
- returna um array de elementos html

**on** 

- Adiciona um evento a um elemento html para executar uma função ou método do componente.
- O primeiro parâmetro recebido deve ser o nome do evento
- O segundo parâmetro deve ser o array de elementos para receber um *event listener*
- O terceiro parâmetro deve ser a função ou método a ser executado

> A função **on** ainda consegue acessar o parâmetro **methods** e é através desse 
parâmetro que os métodos do componente são disponibilizados dentro da propriedade 
**events** do componente.

#### eventos em campos de formulário

É recomendo a utlização de debouceTime nos eventos adicionados em campos de formulário.

No caso de campos de texto por exemplo, isso é necessário porque quando o input altera o
estado do componente e o template é re-renderizado, o input deve ser mantido em fócu. 
Veja abaixo: 

O controller - *./index.js*
```javascript

import template from './template'
import styles from './styles'

export const appHelloWorld = ( element ) => {

    //codigo omitido

    const events = ({on, queryOnce, methods}) => ({
        onClickTitle () {
            const h1 = queryOnce('h1')

            const debounceOptions = {
                focusEnd: () => ['input', element]
                debounceTime: 300
                useDebounce: true                
            }

            on('click', [h1], ({target}) => {
                methods.log(target)
            }, debounceOptions)
        }
    })

    return { 
        //códio omitido
        events  
    }
}
```
A variável debouceOptions foi declarada contendo um objeto que define as seguintes propriedades:

- foucsEnd - Uma função anônima que retorna o seletor e o contexto do elemento respectivamente
- debounceTime - O intervalo em que novas ocorrências do mesmo evento serão canceladas
- useDebouce - Flag Boolean que define se debouceTime está ativo ou inativo.

```javascript
    const debounceOptions = {
        focusEnd: () => ['input', element]
        debounceTime: 300
        useDebounce: true                
    }

    on('click', [h1], ({target}) => {
        methods.log(target)
    }, debounceOptions)
```

> Veja acima que depois de declarada a variável debouceOptions foi passada como **quarto parametro** para 
a função **on**.


#### Methods

Os métodos do componente são apenas funções simples que executam alguma operação e podem retornar algum resultado.

O componente pode mesclar o uso dos métodos com eventos ou hooks para manter o código organizado e ainda
assim executar todas as tarefas pertinentes.

 Veja que  **methods.log** foi passada como handler para a função **on** que a adiciona como listener no evento
 de **"click"**.

```javascript
    //código omitido

    on('click', [h1], methods.log, debounceOptions)
``` 

Só é possível acessar o método log do componente nos eventos por que ele foi declarado como no expemplo abaixo e é
assim que um método sempre será declarado.

```javascript

    const appHelloWorld = (element) => {
        //código omitido
        const methods = () => ({
            log ({target}) {
                console.log(target.value)
            }
        })

        return {
            methods
        }
    }
```

Logo após a declaração da propriedade **events** do componente, **methods** é declaro e events
passa a ter acesso às funções da propriedade methods do componente.

```javascript
    const events = ({ methods }) => ({  
        //Acesso aos métodos declarados
       anyEvent() {
            methods.any('show any value')
       }
    })   
    
    const methods = () => ({ 
        //código omitido
        any (value) { 
            //any action
        }
    })
```    

#### Hooks

Atualmente os hooks são quatros funções que podem ser executadas, cada uma, em um momento específico do ciclo de vida do componente.

- beforeOnInit - Executada um única vez antes da inicialização do componente
- afterOnInit - Executada um única vez depois da inicialização do componente
- beforeOnRender - Executada antes da renderização do componente e toda vez que é renderizado
- afterOnRender - Executada depois da renderização do componente e toda vez que é renderizado

```javascript
const appHelloWorld = (element) => {

    //código omitido
    const hooks = ({ methods }) => ({
        beforeOnInit () {},
        afterOnInit () {},
        beforeOnRender () {},
        afterOnRender () {},
    })

    return {
        hooks
        //código omitido
    }
}
```

Como os hooks também tem acesso aos métodos do componente, pode ser interessante a partir desses hooks, executar operações
assincronas como requisições HTTP antes ou após a inicialização do componente.

Também pode ser interessante executar validadores e filtros de dados sempre que o componente é renderizado através dos hooks beforeOnRender e
afterOnRender.

Apesar de poucos, os hooks combinados a objetos observáveis são mais que suficiente para tratar a reatividade de qualquer
aplicação.

> Os **hooks** se combinados com **objetos observáveis** possibilitarão que uma infinidade de ações possam acontecer de 
forma simples e clara.



### Gerenciamento de estado local

O controller - *./index.js*
```javascript

import template from './template'
import styles from './styles'

export const appHelloWorld = ( element ) => {

    const state = observableFactory({
        title: 'this is a simple component.'
    })

    return { state, template,  styles  }
}
```
Na definicação de componente acima, o gerenciamento de estado local é definido pelo trecho em destaque:

```javascript
    const state = observableFactory({
        title: 'this is a simple component.'
    })
```

É dessa forma que deve ser definido o gerenciamento de estado local. Pois, *observableFactory* retorna
um objeto observável com propriedades especiais que podem ser utilizadas tando para atualizar o *state*
do componente como para observar atualizações e reagir a elas de alguma maneira.

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

Observe que acima a função **set** que é uma propriedade especial do **state** recebe como
parâmetro um objeto contendo o novo valor da propriedade **title** presente no state. É dessa
forma que o state local deve ser atualizado.

#### recuperando informações do state

```javascript

    const state = observableFactory({
        title: 'Default title value'
    })

    // Retorna um objeto estático com as propriedades do state
    const newStaticState = state.get() 

    // Recupera uma propriedade estática do state
    const { title } = state.get() 

```

Observe que acima a função **get** que é uma propriedade especial do **state** é utilizada para recuperar
o estado do componente completo mas estático. Ou seja, caso alterações nas propriedades de *newStaticState* ocorram,
essas alterações não serão refletidas no estado original e não desencadearão efeitos colaterais que atualizam a o template
do componente.

A função **get** pode ser utilzada para recuperar propriedades estáticas específicas do state, bastando informar o nome das
propriedades dentro de chaves como no exemplo acima onde a propriedade estática **title** foi extraída do *state* através
da função *get*.

As propriedades específicas recuperadas também não desencadeiam efeitos colaterais caso seus valores sejam alterados.

#### Observando alterações no state

```javascript

    const state = observableFactory({
        title: 'Default title value'
    })

    const onStateChange = state.on((newPayload) => {
        console.log(newPayload)
    })

    const onUpdateState = state.on(methods.logger)

    const methods = () => ({
        logger (value) { console.log(value) }
    })

```

No exemplo acima, **on** foi utilizada para adicionar funções como observadoras para mudanças no estado. 
Quando uma alteração ocorrer em uma das propriedades do estado, as funções observadoras serão executadas.

No primeiro caso uma função anônima foi adicionada como observadora. Já no segundo caso um método do componente
foi adicionado.

Observe que para ambos os casos, a referência para o observador adicionado é retornado para que posteiormente 
possa ser removidos caso necessário.

> referências: onStateChange e OnUpdateState

#### Removendo observadores do state

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

### Compartilhando o state

Uma maneira simples e eficaz para compartilhar o state entre componentes é através de objetos observáveis.

Por exemplo, no caso em que um componente pai quer enviar informações em um determinado momento para o componente
filho.

* **Compartilhado para baixo**

Primeiro um serviço mensageiro pode ser criado como no exemplo abaixo onde messengerDown envia informações as componentes
filhos e messengerUp ao componente pai.

```javascript
//messenger.service.js

import { observableFactory } from "r9x_js";
export const messengerUp = observableFactory({})
export const messengerDown = observableFactory({})
```

No componente pai então o serviço mensageiro é importado e messengerDown define as informações a serem enviadas aos
componentes filhos.

Ainda no componente pai, messengerUp é configurado para escutar alterações que devem gerar efeitos colaterias no 
componente pai através da função **messengerUp.on** que adiciona uma função como handler a ser executado quando
uma mensagem de dados for enviada ao componente pai.

> Receber informações de componentes filhos pode ser interessante por exemplo para validação de formulários onde
> uma operação só pode ser executada caso todos os campos do formulário estejam validados.

```javascript
//appHelloWorld/index.js

import { messengerDown, messengerUp } from '../services/messenger.service.js'
import { appName } from '../componentes/appName'

const appHelloWorld = ( element ) => {
    //código omitido

    const state = observableFactory({
        name: 'Jhon'
    })

    const template = ({ html }) => html`
        <h1>
            Hello, <app-name></app-name>
        </h1>
    `

    const children = () => ([
        appName
    ])

    hooks () {
        beforeOnInit () {
            messengerDown.set({ ...state.get() })
            messengerUp.on((payload) => console.log(payload))
        }
    }

    return {
        state,
        element,
        template,
        children,
        hooks
    }
}
```

O exemplo acima é de como se parece um **componente pai** compartilhando o state com o componente filho. Mas,
o componente filho se parece muito com o componente pai. Veja abaixo:

* **compartilhando para cima**

```javascript
//appName/index.js

import { messengerDown, messengerUp } from '../services/messenger.service.js'

const appName = ( element ) => {
    //código omitido

    const state = observableFactory({
        name: ''
    })

    const template = ({ state, html }) => html`
        <span>${state.name}!</span>
    `

    hooks () {
        beforeOnInit ({ methods }) {
            messengerUp.set({ anyData: ''})
            messengerDown.on((payload) => methods.updateState(payload))
        }
    }

    const methods = () => ({
        updateState (payload) {
            state.set({ ...payload })
        }
    })

    return {
        state,
        element,
        template,
        hooks,
        methods,
    }
}
```

A semelhança é perceptível. No entando, enquanto o componente pai utiliza o método messengerUp.on 
para escutar notificações e reagir a elas, o componente filho utiliza messengerUp.set para enviar 
notificações de dados ao componente pai.

No caso em que o componente pai utiliza o método messengerDown.set para enviar notificações contendo
dados ao componente filho, o componente filho utiliza o método messegerDown.on para escutar essas notificações de
dados e reagir a elas.

* No componente pai

```javascript
/*appHelloWorld/index.js*/

    hooks () {
        beforeOnInit ({ methods }) {
            messengerDown.set({ ...state.get() })
            messengerUp.on((payload) => console.log(payload))
        }
    }
```

* No componente filho:

```javascript
/*appName/index.js*/

    hooks () {
        beforeOnInit ({ methods }) {
            messengerUp.set({ anyData: ''})
            messengerDown.on((payload) => methods.updateState(payload))
        }
    }
```

### Comunicação entre componentes

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

    hooks () {
        beforeOnInit ({ methods }) {
            messengerB.set({ anyData: ''})
            messengerA.on((payload) => methods.updateState(payload))
        }
    }

```
* Componente B

```javascript

    import { messengerA, messengerB } from '../services/messenger'

    hooks () {
        beforeOnInit ({ methods }) {
            messengerA.set({ anyData: ''})
            messengerB.on((payload) => methods.updateState(payload))
        }
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

Observe que no trecho de código abaixo está acontecendo exatamente assim:

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

#### Compartilhamento por eventos customizados

Os componentes também podem se comunicar através de eventos customizados. Esses eventos são
baseados no padrão pubsub e sua implementação se parece bastante com a implementação de objetos
observáveis.

1. **Criando o serviço de mensagem**

Para criar o serviço de gerenciamento de eventos é necessário importar a fábrica do objeto pubsub, criar um objeto com a fábrica e exportar esse objeto, exatamente como no código abaixo.

```javascript
//services/eventDrive/index.js

    import { pubsubFactory } from 'lemeJS'

    export const eventDrive = pubsubFactory()

```

2. **Usando o serviço nos compoentes**

Como pode observar, para usar o seviço, basta importá-lo.

Uma boa maneira para começar a usá-lo é se inscrevendo para ouvir eventos disparados por outros componentes combinando o uso do ***eventDrive*** com o hook ***beforeOnInit*** exatamente como no código abaixo.

```javascript
//component/anyCompoent/index.js

    import { eventDrive } from '../../services/eventDrive'

//código omitido

    const hooks = () => ({
        beforeOnInit () {
            eventDrive.on('onIncrement', (payload) => {
                console.log(payload)
            })            
        }
    })

//código omitido
```

No trecho em destaque abaixo, ***eventDrive.on*** faz com que o componente se registre para ser
notificado de quando um evento ***onIncremente*** é disparado por outro componente. Dessa forma,
quando esse evento ocorrer o componente que se registrou executa o callback/handler passado como segundo parâmetro para eventDrive.on e pode realizar as operações necessárias com os dados recebidos através do callback fornecido ao evento.

> Para adicionar um listener em eventos, **eventDrive.on** recebe 2 parâmetros. Sendo o primeiro uma string contendo o nome do evento e o segundo um callback/handler que recebe um único parâmetro.

3. **Nomes de eventos**

Você pode criar quaisquer nomes para eventos, mas, é recomendado a convenção de prefixar o nome
do evento com **on** e utilizar **camelCase** na junção de direntes palavras.

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

Aplicações maiores podem precisar de um formato mais flexível e efetivo na comunicação entre componentes
de diferentes níveis. Nesse caso uma **store** de dados pode ser implementada para gerir o estado da
aplicação tornando a comunicação entre os componentes mais simples.

Implementar uma **store** traz diversos benefícios:

- Simplifica a comunicação entre componentes.
- Diminui a necessidade de objetos observaveis (messengerUp, messengerDown).
- Diminui a necessidade de usar emissores de eventos (eventDrive).
- Permite navegar na linha do tempo das modificações do state da aplicação.
- Permite organizar o código por módulos.
- Diminui e extrai a lógica de modificação de dados nos componentes.
- Redução da quantidade de código da aplicação de forma geral.

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

O state nesse caso é global, por tanto, deve concentrar todos os dados que serão compartilhados entre
os componentes da aplicação.

Considere um aplicativo de gerenenciamento de usuários formada ao menos por 2 componentes:
    * Cadastro de usuários
    * Listagem de usuários

A store de dados deve ser estruturada com base nos dados consumidos por componentes da aplicação. Logo o state se deve se parecer como no código abaixo:

```javascript
//./src/store/state/index.js

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

Uma **mutation** se parece assim:

```javascript
//./src/store/mutations/user/index.js

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

Observe que cada uma das mutations recebem 2 parâmetros, sendo o primeito o state global e o segundo
a carga de dados a ser criada, alterada, ou removida no state.

```javascript
//./src/store/mutations/user/index.js

    const addUser = (state, payload) => {
        const userList = [...state.userList, payload.newUser]
        return { ...state, userList }
    }

    const removeUser = (state, payload) => {
        const userList = state.userList.filter( user => {
            if(user.id !== payload.id) return user
        })
        return { ...state, userList }
    }

    export const userMutations = { 
        addUser, 
        removeUser 
    }
```

Acima o código completo das mutatio para adicionar e remover usuários da store.

Oberserve que cada **mutation** retorna uma mutação do estado já com todas as alterações
necessárias. Ou seja, elas modifica o estado e retornam a nova representação do mesmo.

Uma vez que já estão definidos o state global e as mutations que o modificarão é chegada a 
hora de completar a store.

3. **Fabricando a store**

O arquivo index.js na raiz da store deve se parecer com isso:

```javascript
//./src/store/index.js

    import { state } from './state'
    import { userMutations } from './mutations/user'    
```

Primeiro, importa-se state e o módulo de mutações. Na sequência, as mutações podem ser agrupadas 
como no trecho em destaque:

```javascript
//./src/store/index.js

    import { userMutations } from './mutations/user'

    const mutations = {
        ...userMutations,
    }
```

Por fim, a store é fabricada através da função **storeFactory** que completa se parece com o código abaixo:

```javascript
//./src/store/index.js

    import { storeFactory } from 'lemeJS'

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

#### Integrando a store em componentes

Para utilizar a store de dados em componentes basta importá-la para na sequência definir o
state do componente.

```javascript
    //componentes/appUserRegistration/index.js
    import { store } from '../../store

    const appUserRegistrations = (element) => {

        const state = {
            title: 'User Registration',
            user: {}
        }
    }
```

Na sequência pode ser necessário implementar alguma ação para executar a mutation apropriada e realizar a operação necessária.

No caso do cadastro de usuários, a mutation a ser executada deve ser **addUser**.

```javascript
    //componentes/appUserRegistration/index.js
    import { store } from '../../store

    const appUserRegistrations = (element) => {

        const state = {
            title: 'User Registration',
            user: {}
        }

        const events = ({on, queryOnce, methods}) => ({
            onClickToAddUser () {
                const buttonSave = queryOnce('button')
                on('click', [buttonSave], methods.addUser)
            }
        })

        const methods = () => ({
            addUser () {
                const newUser = {userId: 4, userName: 'Eric', userEmail: 'eric@email.com'}
                store.emit('addUser', newUser)
            }
        })
    }
```

Conforme o código acima, quando o botão adicionar usuário for clicado, a função addUser será executada e a store emit uma ação **addUser** que executa a mutation de mesmo nome realizando
a adição no novo usuário na lista de usuários.

Aproveitando o evento de adição é possível fazer com que o formulário do componente de cadastro
seja apagado por completo para facilitar um novo cadastro e também é possível fazer com que a
listagem de usuários seja atualizada no outro componente.

* **Limpando o formulário de cadastro**

Para limpar o formulário de cadastro basta zerar as informações do usuário no state local do componente de cadastro.

É completamente possível tirar proveito dos hooks para ouvir ações da store e reagir a elas. Então,
tirando proveito disso o state local pode ser apagado.

```javascript
    //componentes/appUserRegistration/index.js
    import { store } from '../../store

    const appUserRegistrations = (element) => {

        const state = {
            title: 'User Registration',
            user: {}
        }

        const hooks = () => ({
            beforeOnInit () {
                store.on('addUser', (payload) => {
                    const user = {}
                    state.set({ user })
                })
            }
        })

        const events = ({on, queryOnce, methods}) => ({
            onClickToAddUser () {
                const buttonSave = queryOnce('button')
                on('click', [buttonSave], methods.addUser)
            }
        })

        const methods = () => ({
            addUser () {
                const newUser = {userId: 4, userName: 'Eric', userEmail: 'eric@email.com'}
                store.emit('addUser', newUser)
            }
        })
    }
```

Observe o trecho em destaque:


```javascript
    const hooks = () => ({
        beforeOnInit () {
            store.on('addUser', (payload) => {
                const user = {}
                state.set({ user })
            })
        }
    })
```

No trecho em destaque, através do hook **beforeOnInit** que é executado uma única vez antes
do componente inicializar, foi adicionado um ouvinte da store que será executado sempre que
a action addUser for disparada. Assim, toda vez que um novo usuário for adicionado o state
local do componente cadastro de usuários será apagado.

* **Atualizando a lista de usuários**

```javascript
    //componentes/appUserRegistration/index.js
    import { store } from '../../store

    const appUserRegistrations = (element) => {

        const { userList } = store.get()

        const state = {
            title: 'User list',
            userList
        }

        const hooks = ({ methods }) => ({
            beforeOnInit () {
                store.on('addUser', methods.addUser)
            }
        })    

        const methods = () => ({
            addUser ({ user }) {
                state.set({ user })
            }
        })

    }
```

No código acima, através do hook beforeOnInit foi adicionado um ouvinte da store através do método
***store.on*** e sempre que a action **addUser** for executada a função **methods.addUser**
também é executada atualizando o state local do componente o que força o template do mesmo a ser
atualizado com a nova lista de usuários.

Ainda resta uma última ação necessária. A remoção de usuários.

```javascript
    //componentes/appUserRegistration/index.js
    import { store } from '../../store

    const appUserRegistrations = (element) => {

        const { userList } = store.get()

        const state = {
            title: 'User list',
            userList
        }

        const hooks = ({ methods }) => ({
            beforeOnInit () {
                store.on('addUser', methods.addUser)
                store.on('removeUser', methods.updateUserList)
            }
        })

        const events = ({on, queryOnce, methods}) => ({
            onClickToRemoveUser () {
                const buttonRemove = queryAll('button')
                on('click', [buttonRemove], methods.removeUser)
            }
        })            

        const methods = () => ({

            addUser ({ user }) {
                state.set({ user })
            },

            removeUser ({ target }) {
                const id = +target.dataset.id
                store.removeUser({ id })
            },

            updateUserList ({ userList }) {
                state.set({ userList })
            }   

        })

    }
```

Primeiro um ouvinte para a action **removeUser** da store foi definido e esse ouvinte executa o
método do componente **updateUserList** que por sua vez atualiza a lista de usuários no state
local do componente forçando o template do mesmo a exibir a lista de usuários atualizada.

```javascript
    store.on('removeUser', methods.updateUserList)

    const methods = () => ({
        updateUserList ({ userList }) {
            state.set({ userList })
        }      
    })
```

Na sequência um evento de clique no usuário a ser removido foi adicionado e sempre que um botão
de remover usuário é clicar a action **removeUser** é disparada o que executa a mutation de mesmo
nome removendo o usuário através do **id** fornecido.

```javascript

    const events = ({on, queryOnce, methods}) => ({
        onClickToRemoveUser () {
            const buttonRemove = queryAll('button')
            on('click', [buttonRemove], methods.removeUser)
        }
    })     

    const methdos = () => ({
        removeUser ({ target }) {
            const id = +target.dataset.id
            store.removeUser({ id })
        }    
    })

```

Dessa forma, sempre que um usuário for removido o componente será renderizado novamente e a lista 
de usuários será exibida corretamente.


## Navegação e Rotas

Atualmente a sistema de navegação de **LemeJs** é baseado em **hash (#)** e expressões regulares
que havaliam o hash atual para decidir que view será exibida.

```javascript
    //routes/index.js

    import {appHome} from './components/appHome';
    import { appNotFound } from "./components/appNotFound";


    export const routes = {
    firstRoute: { hash: "#/", component: appHome },
    defaultRoute: { hash: "#/404", component: appNotFound },
    otherRoutes: [
        { hashExp: /^#\/$/, component: appHome },
        { hashExp: /^#\/other$/, component: appNotFound },
    ],
    };

```

Como pode ver acima, as rotas da aplicação são definidas através de um objeto.

* **Propriedades de rotas**
    - firstRoute - Primeira rota a ser carregada na aplicação
    - defaultRoute - Rota carregada no caso de hash inválido ou rota inexistente
    - otherRoutes - Lista de rotas baseadas em regex que havalia o hash atual para carregar uma view
    - hashExp - Expressão regular usada para havaliar o hash
    - component - Componente/view a ser carregado caso a rota seja valido

* **Expressão regular de rota**

```javascript
    { hashExp: /^#\/$/, component: appHome },
    { hashExp: /^#\/other$/, component: appNotFound },
```

No trecho acima, a primeira expressão regular ```/^#\/$/``` valida para **true** o hash ```#/``` que muitas vezes representa a página inicial de uma aplicação.

Já a expressão ```/^#\/other$/``` valida para **true** caso o hash seja ```#/other```.

No caso em que o hash atual difere de todas as rotas definidas, o component/view fornecido para **defaultRoute** é carregado no hash fornecido como padrão.

```javascript
    defaultRoute: { hash: "#/404", component: appNotFound }
 ```

 * **Renderizando uma rota**

 Para rederizar uma rota é necessário utilizar o componente ```<router-view></router-view>``` do próprio sistema de rotas. No entanto não é mesmo nem necessário importá-lo. Basta inserir a tag do componente
 de rotas no local desejado e as rotas serão renderizadas.

 ```javascript
    //componentes/appUserRegistration/index.js
  

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

Agora que já conhece mais sobre Leme JS hora de seguir o tutorial, por em prática todos os conceitos vistos até aqui e
desenvolver uma aplicação reativa completa.

- [Seguir o tutorial](/tutorial)