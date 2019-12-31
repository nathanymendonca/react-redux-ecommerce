# Arquitetura Flux um Ecommerce

## Vamos construir um ecommerce para aprender o Flux com a implementação do Redux, usando Redux Saga para fazer o side effects das funcionalidades assíncronas

Vamos construir um ecommerce do zero bem bonito com Create React App utilizando o Redux, veja as imagens no final do post.

### Utilizaremos o Redux para explicar o Flux.

## Aula 01 - Conceitos de Redux

- Biblioteca que implementa Arquitetura Flux;
  - Ela pode ser utilizada com qualquer framework javascript (Angulas, Vue, React), ou até mesmo com JS puro
  - Redux implementa o Flux, [Flux](https://facebook.github.io/flux/) é uma arquitetura, um conceito que facilita a comunicação entre os elementos em tela.
- Controle de estados globais da aplicação, um estado global é quando não tem um dono específico, ele é pode ser usado em qualquer componente ou estrutura de código para renderizar uma tela;
- Quando utilizar o Redux?
  - Meu estado tem mais de um “dono”? Quando o estado é exibindo em outros lugares, não apenas em um componente específico.
  - Meu estado é manipulado por mais componentes? Se sim, podemos utilizar o Redux.
  - As ações do usuário causam efeitos colaterais nos dados? Exemplo: adicionar um produto no carrinho, deveria disparar uma mensagem para o usuário ou mudar um contador em outro componente da tela.

Redux é utilizado para controlar estados globais, um bom exemplo para utilizar Redux: Carrinho de compras, dados do usuário logado com as permissões, player de música, etc;

Quando o estado não ter um dono específico ou tiver que ser exibido em outros lugares, então o Redux é uma boa solução, geralmente para médio a grande projetos onde os estados se espalham na aplicação.

## Arquitetura Flux

Sempre que queremos acessar ou atualizar um estado nós disparamos uma Ação (Action)

Em um e-commerce, se o usuário clica em catálogo de produtos, um action é disparada (a action é disparada não apenas quando o usuário faz uma ação, pode ser o próprio sistema disparando uma ação, por exemplo quando o componente carrega no `componentDidMount` do React, então podemos disparar um `action` para fazer algo)

A Action tem uma estrutura:

```json
{
  type: ADD_TO_CART,
  product: { ... } 
}
```

Que é um tipo e o payload, o valor que ela carrega ou retorna.

Então no nosso caso, estamos enviando uma ação de Adicionar ao carrinho, um produto que o usuário gostou e escolheu.

Essa ação vai para o `Redux Store`, que contém os `reducers` (termo que designa a separação de estados no Redux Store) no redux podemos ter vários estados, e cada reducers separa o estado por funcionalidade, podemos ter um Store com vários reducers. Podemos ter um Cart Reducer, User Reducer, onde Cart armazena os valores de compra e User armazena os valores do usuário logado, etc. No exemplo acima, o Cart Reducer recebe a action ADD*TO*CART, ele faz uma mutação no estado, alterando o valor, incluindo no Carrinho um novo produto.

E agora o componente Carrinho no header da aplicação escuta a alteração, e ele mesmo pode disparar uma action para atualizar o valor de quantidade inserida no carrinho:

```json
{
  type: UPDATE_QUANTITY,
  product: { ... },
  quantity: 5
}
```

Um reducer recebe essa action e faz a alteração no estado novamente e esse valor é replicado para quem ouvir esse estado.

## Princípios

- Toda action deve possuir um "type", informando o tipo da ação e deve ser uma string única.
- O estado do Redux é o único ponto de verdade, se o estado do carrinho estiver no Redux todo os dados referente ao carrinho tem que estar no Redux, não pode ficar no componente e no Redux ao mesmo tempo, isso ocorrer gera inconsistência e fica ruim para manter.
- Não podemos mudar o estado do Redux sem um action; O estado é imutável, apenas o Reducer pode alterar de acordo com as chamadas da Action.
- As actions e reducers são funções puras, ou seja, não lidam com side-effects assíncrons, isso quer dizer que, elas não vão acessar banco de dados, chamar api, etc. Isso ajuda muito com testes unitários, pois uma função pura, sempre que receber o mesmo parâmetro sempre vai devolver o mesmo resultado. Para lidar com side-effects assíncronos usamos Redux Saga que veremos mais pra frente.
- Qualquer lógica síncrona para regras de negócio deve ficar no reducer e nunca na action, a action não altera dados, ela só carrega os valores, o reducer faz a mutação do estado
- Nem toda aplicação precisa Redux, inicie sem ele e sinta a necessidade depois, é uma das forms mais fáceis de entender Redux, pq colocar Redux de cara, vc vai escrever mais código e não vai entender porque está sendo útil, só é legal colocar de cara se você souber que o projeto vai crescer, se os requisitos estiverem bem claros;

## Exemplo do Carrinho

Cart Reducer, sempre vai iniciar com o estado vazio, um `[ ]` vazio por exemplo.

O Cart Reducer sempre vai ouvir as actions que a aplicação disparar, se ele ouvir o tipo da action, ele faz o que precisa fazer, então o Cart Reducer ouve a action do Type: ADD*TO*CART:

```json
{
  type: "ADD_TO_CART",
  product: {
    id: 1,
    title: "Novo produto",
    price: 129.9
  }
}
```

E Agora o Cart Reducer colocar o produto no estado:

```json
[
  {
    id: 1,
    title: "Novo produto",
    price: 129.9,
    amount: 1,
    priceFormatted: "R$129,90"
  }
]
```

O reducer é capaz de adicionar campos mesmo sem a action tem enviado os campos, veja o `amount` e `priceFormatted`. O reducer sabe que vou precisar do preço formatado então já criou isso pra gente e como é o primeiro produto que coloco no carrinho, logo a quantidade é 1. E essa quantidade vai ser usado em algum lugar, então temos esses dados a nossa disposição.

E outro momento o usuário pode escolher o mesmo produto de ID 1, e então apenas atualizamos o valor de quantidade.

Chamando outra action do type: UPDATE_AMOUNT, com produto de ID 1 e com quantidade 5.

```json
{
    type: “UPDATE_AMOUNT”,
    product: 1,
    amount: 5,
}
```

O Cart Reducer ouve essa actions e manipula o estado da maneira necessidade:

```json
[{
    id: 1,
    title: "Novo produto",
    price: 129.9,
    amount: 5,
    priceFormatted: "R$129,90"
}]
```

Pronto, agora atualizou apenas o valor.

Redux é fácil, basta entender o conceito de Actions, Store e Reducers, e principalmente quando utilizar o Redux na aplicação.

## Aula 02 - Estrutura do Projeto

Vamos criar uma loja virtual de calçados (Rocketshoes) para aprender a implementação do Redux.

Utilizaremos o Create React App para criar o frontend da aplicação em React:

```shell
npx create-react-app rocketshoes
```

Executei no terminal:

```shell
yarn & yarn start
```

Pronto, tudo rodando!

## Aula 03 - Configurando Rotas

Vamos criar a configuração de navegação do projeto.

Não importei o `BrowserRouter` de dentro do `routes.js` pois iremos criar um componente Header que também precisará de ter acesso aos dados da rota.

Então os componentes que precisarão de Rotas vão ficar no App.js

## Aula 04 - Estilos Globais

- Instalamos a lib styled-components;
- Criamos o arquivo globals.js com os estilos globais;
- Importamos o estilo para App.js

## Aula 05 - Criando o Header

- Crio o componente de estilização do header
- Crio o componente Header que tem uma logo e link para a home e o carrinho e o link para o carrinho
- Coloco o Header no topo do App.js

## Aula 06 - Estilização da Home

- Criamos a estilização no arquivo styled.js e aplicamos no index.js da Home
- Destaque para a lib [polished](https://polished.js.org/) que instalamos, com `yarn add polished`. Agora podemos manipular as cores com essa lib.

https://www.youtube.com/watch?v=CbFMwHvHK1Y

## Aula 07 - Estilização do Carrinho

- Criamos a estilização no arquivo styled.js e aplicamos no index.js do Cart
- Utilizamos novamente a lib do polished para gerenciar a cor do hover do botão finalizar pedido.

## Aula 08 - Configurando a API

Configurar a API para consumir produtos.

Utilizamos a API [json-server](https://github.com/typicode/json-server) para criar uma API Fake, que simula o que aconteceria em uma API Real.

Só criar um json com os dados fake e rodar o json-server que vai simular a API, inclusive dá para configurar tempo de resposta e outras coisas mais.

Para instalar de forma global na máquina:

```text
yarn add json-server -D
```

Depois criar um arquivo `server.json` na raiz do projeto e colocar os dados contendo o estoque (stock) e os produtos (products), inclusive fazendo os relacionamentos com id.

Teremos uma rota stock que retorna o estoque dos produtos, e a rota products que retorna os produtos.

Vamos configurar o axios para consumir essa api.

```shell
yarn add axios
```

Veja o arquivo `api.js` no código fonte.

E para rodar a api fake, basta rodar:

```shell
yarn json-server server.json -p 3333 -w
```

E podemos também ter um script no package.json:

```json
...
"server":  "json-server server.json -p 3333 -w"
...
```

E rodar `yarn server` para executar a api.

- json-server é o nome da lib
- server.json é o nome do arquivo que tem a api fake e está na raiz do projeto, por isso não passo o caminho, apenas o nome do arquivo. Está no mesmo nível do package.json
- -p 3333 é a porta que defini para rodar essa api
- -w é para ficar `watching` cada alteração que eu fizer na api, se eu mudar alguma coisa lá dentro não preciso rodar o comando novamente.

Pronto agora para acessar, só chamar a rota que deseja:

```text
http://localhost:3333/stock
```

e

```text
http://localhost:3333/products
```

Ambas as rotas vão mostrar um array com seus respectivos objetos.

Legal, que se eu chamar o produto e passando o id, ele me traz apenas o produto com seu respectivo id informado:

```
[http://localhost:3333/products/4](http://localhost:3333/products/4)
```

Se eu passar um ID q não exisite ele retorna um objeto vazio.

Dá até para deletar e atualizar valores dentro dessa api, é muito legal!

Excelente para usar em modo de desenvolvimento no frontend quando um backend não foi feito ainda com a API.

## Aula 09 - Buscando produtos da API

Nessa aula vamos buscar os dados da API e renderizar na tela do usuário, seguindo algumas boas práticas de código.

- Transformar o componente stateless Home em uma classe statefull para poder armazenar o estado dos products que vieram da API.
- Criei um arquivo `format.js` dentro de `src/util` que exporta uma função que formata valores em moeda brasileira, utilizando a instância de `Intl.NumberFormat` do Javascript.
- Buscamos os dados da API, armazenamos no estado products, e populamos a lista de produtos na tela
- Assim que buscamos os dados da API fizemos um map para devolver um array de produtos com o preço formatado, ao invés de usar a função `formatPrice()` de dentro do render na variável price do product. Não é recomendado colocar funções que manipulam variáveis de dentro do render, pois a cada alteração no estado, essa função é chamada também, fazendo com que perca performance, a ideia é sempre entregar para o render os valores prontos para serem renderizados e qualquer formatação deve ser feita antes.

## Aula 10 - Configurando o Redux

Nessa Aula vamos configurar o Redux na aplicação.

```text
yarn add redux react-redux
```

Instalamos o redux em si e a integração do React com Redux.

Vamos criar uma pasta `store` de dentro da `src` com o arquivo `index.js`:

```react
import { createStore } from 'redux';

const store = createStore();

export default store;
```

Agora no App.js vamos importar o `Provider` do `react-redux` para poder deixar disponível o `store` em toda a aplicação, portanto o `Provider` tem que envolver todos os componentes. E o provider recebe uma prop chamada store, que é a store que criamos, no código acima.

```react
import React from 'react';
import { BrowserRouter } from 'react-router-dom';
import { Provider } from 'react-redux';
import Routes from './routes';
import GlobalStyle from './styles/globals';
import Header from './components/Header';
import store from './store';

function App() {
  return (
    <Provider store={store}>
      <BrowserRouter>
        <Header />
        <GlobalStyle />
        <Routes />
      </BrowserRouter>
    </Provider>
  );
}

export default App;
```

Pronto agora o store está disponível na aplicação, e o projeto *crashou*.

```text
Error: Expected the reducer to be a function.
```

Isso ocorreu porque sempre que criamos um store, temos que passar uma função que é **reducer**.

Vamos criar o nosso primeiro reducer para deixar disponível no store.

```react
import { createStore } from 'redux';

function cart() {
  return [];
}

const store = createStore(cart);

export default store;
```

Pronto agora o app volta a funcionar.

Porém se aplicação crescer demais, é interessante separar os reducers em outro arquivo, por módulo e funcionalidade.

Então vamos criar uma pasta `models` e dentro dela outra pasta `cart` com o arquivo `reducers.js`:

E dentro dela colocar o reducer de carrinho.

```react
export  default  function  cart() {
	return [];
}
```

E importamos no index.js do store:

```react
import { createStore } from 'redux';
import reducer from './models/cart/reducer';

const store = createStore(reducer);

export default store;
```

Mas como vamos ter vários reducers vamos ter que mudar o store novamente, vamos criar um arquivo `rootReducer.js` dentro de `models`, e esse arquivo vai conter todas as funções dos reducers e vamos combiná-las para que o store tenha acesso em um único estado.

`rootReducer.js`:

```react
import { combineReducers } from 'redux';

import cart from './cart/reducer';

export default combineReducers({
  cart,
});
```

e no `index.js` da store, importamos o `rootReducer` que contém os reducers:

```react
import { createStore } from 'redux';
import rootReducer from './models/rootReducer';

const store = createStore(rootReducer);

export default store;
```

Até aqui a aplicação está funcionando, mas ainda não estamos utilizando os reducers, apenas configuramos o store e o reducers, ainda falta as actions e o acesso aos stores.

## Aula 11 - Adicionando ao carrinho

Vamos adicionar o produto ao Carrinho, todos os dados da tela serão passados para o reducer de carrinho que vamos criar.

Vamos utilizar o Redux agora no component da Home

Para conectar o componente com Redux vamos utilizar a função `connect` do `react-redux`.

`Home/index.js`:

```shell
import { connect } from 'react-redux';
```

O connect retorna outra função, e com essa função de retorno, nós chamamos ela passando o componente como parâmetro:

`Home/index.js`:

```react
...
class  Home  extends  Component { ... }
...
export  default  connect()(Home);
```

connect() recebe alguns parametros, que vamos falar pra frente:

Para disparar uma action para o redux, precisamos de uma função, então quando o usuário clicar no botão de adicionar ao carrinho, vamos disparar um action.

```react
 <button
   type="button"
   onClick={() => this.handleAddProduct(product)}  
>
```

Quando usamos o `connect`, ele passa para o componente um `prop` chamada `dispatch` que serve para chamar uma **ação**, ou seja, disparar uma ação.

```text
this.props.dispatch
```

**Dispatch** é uma função que recebe uma action e a action tem um tipo e o valor que ela carrega com ela, o tipo (*type*) é obrigatório

Exemplo de uma action:

```json
// action
{
	type: 'ADD_TO_CART',
	product,
}
```

Então quando o usuário clicar no botão vai ser chamado essa função `handleAddProduct` que vai utilizar a prop `dispatch` para executar a função com parâmetro informando o type: 'ADD*TO*CART' que algum reducer (cart reducer) vai ouvir e tratar o valor `product` que está sendo enviado como parâmetro:

```react
...
  handleAddProduct = product => {
    const { dispatch } = this.props;
    dispatch({
      type: 'ADD_TO_CART',
      product,
    });
  };
...
```

Agora vamos testar a chamada dessa função, adicionando um log no `cart/reducer.js`:

```react
export default function cart() {
	console.log('teste');
	return [];
}
```

Agora abrindo console do navegador, vamos ver que apareceu ***teste\*** três vezes, ou seja, a função foi chamada mesmo sem termos clicado no botão adicionar carrinho, vou explicar depois, e logo quando clicamos no adicionar carrinho a função dispatch é disparada e o log ***teste\*** aparece no navegador.

Toda vez que um *dispatch* é disparado todos os reducers escutam e executam a função. Fique com isso em mente. Agora qual reducer vai lidar com a chamada, depende do type e do reducer que escuta esse type, já vamos ver isso melhor.

E para receber os valores do parâmetro enviado no dispatch?

Todo o reducer, recebe por padrão uma variável `state` e outra variável chamada `action`.

```react
export default function cart(state, action) {
  console.log(action);
  return [];
}
```

E agora se executarmos essa função, vamos ver as actions sendo logadas no console:

```text
{type: "@@redux/INITp.i.5.b.k.v"}type: "@@redux/INITp.i.5.b.k.v"__proto__: Object
reducer.js:2 {type: "@@redux/PROBE_UNKNOWN_ACTIONv.t.j.3.7"}type: "@@redux/PROBE_UNKNOWN_ACTIONv.t.j.3.7"__proto__: Object
reducer.js:2 {type: "@@redux/INITp.i.5.b.k.v"}
reducer.js:2 {type: "ADD_TO_CART", product: {…}}
reducer.js:2 {type: "ADD_TO_CART", product: {…}}
reducer.js:2 {type: "ADD_TO_CART", product: {…}}
```

E olha que legal, os três primeiros logs é a inicialização e integração do React com Redux. E os outros três sou eu clicando no botão adicionar ao carrinho.

Veja que a **action** trouxe nosso objeto com o **type** e o **product**.

O state é o estado anterior à chamada do dispatch, então sempre que está vindo alguma action, provavelmente vai ser alterado o estado de alguma variável, então o reducer recebe o estado atual e a action traz um estado que irá mudar o estado atual, essa alteração é feita de forma que respeite a imutabilidade do store, onde temos que recriar o estado passando o novo valor para ele.

Agora vamos ver a cara completa de um reducer:

```react
export default function cart(state, action) {
  switch (action.type) {
    case 'ADD_TO_CART':
      return [];
    default:
      return state;
  }
}
```

Esse é o template de um reducer, ele é uma função, que recebe o estado (state) atual e uma action, ele possui um if ( que no caso por padrão é um switch mesmo) com o valor do type da action. Caso o type da action é igual ao case do switch do reducer, então ele faz algo com o state e retorna o novo state. Se não tiver nenhum um type para ele, ele retorna o state atual.

Como eu disse acima, todo os reducers ouvem a chamada do dispatch, então o que faz um reducer alterar o estado é ele ter o mesmo type do action.type no switch, caso contrário não faz nada, simplesmente devolver o estado atual. Sim, se eu disparar uma action do type: DELETE*CATEGORY*PRODUCT, que provavelmente outro reducer de categoryProduct estaria na aplicação iria ouvir e tratar essa action, mas ela seria ouvida pelo nosso reducer cart também mas ele só devolveria o state atual, pois ele não lida com essa action.

Agora vou mostrar a alteração do estado:

```react
export default function cart(state = [], action) {
  console.log(state);

  switch (action.type) {
    case 'ADD_TO_CART':
      return [...state, action.product];
    default:
      return state;
  }
}
```

Sempre o estado começa com um array vazio, por isso passamos um array vazio como default para o state.

Olha que legal, quando eu clico em adicionar ao carrinho, na primeira vez loga um array vazio, pq estou logando o estado atual, antes de alterar o store, mas quando eu clico novamente, ele me mostra o store com produto dentro dele:

```text
[]
reducer.js:2 [{…}]0: {id: 1, title: "Tênis de Caminhada Leve Confortável", price: 179.9, image: "https://rocketseat-cdn.s3-sa-east-1.amazonaws.com/modulo-redux/tenis1.jpg", priceFormatted: "R$ 179,90"}length: 1__proto__: Array(0)
```

E conforme eu vou clicando ele vai adicionando ao carrinho.

E agora como acessar os dados do carrinho, da store de cart no componente de **Header**:

Como queremos acessar os reducers, então vamos importar o connect dentro do `Header/index.js`:

```react
...
import { connect } from 'react-redux';
...
```

Ok, agora queremos acessar o reducer, o **connect** recebe dois parâmetros, o primeiro é o estado e o segundo vou falar depois.

O primeiro parâmetro é uma função que recebe o state e retorna o state do reducer que queremos para dentro de uma variável que passamos para o React.

```text
export  default  connect(state  => ({
	cart: state.cart,
}))(Header);
```

Veja que passamos uma função com parâmetro state, que retorna um objeto com a propriedade cart, que possui o estado do reducer cart. **cart** foi o nome que colocamos lá no `rootReducer.js`

Só para lembrar:

```react
import { combineReducers } from 'redux';
import cart from './cart/reducer';

export default combineReducers({
  cart,
});
```

Agora vamos ver na prática o código completo:

```react
import React from 'react';
import { Link } from 'react-router-dom';
import { MdShoppingBasket } from 'react-icons/md';
import { connect } from 'react-redux';
import { Container, Cart } from './styles';
import logo from '../../assets/images/logo.svg';

function Header({ cart }) {
  console.log(cart);

  return (
    <Container>
      <Link to="/">
        <img src={logo} alt="Rocketshoes" />
      </Link>

      <Cart to="/cart">
        <div>
          <strong>Meu carrinho</strong>
          <span>3 items</span>
        </div>
        <MdShoppingBasket size={36} color="#FFF" />
      </Cart>
    </Container>
  );
}

export default connect(state => ({
  cart: state.cart,
}))(Header);
```

O **Header** recebe como `props` uma prop chamada **cart** que possui todos os valores de `state.cart`.

Agora quando clicamos em **adicionar ao carrinho**, vemos no *console.log* o estado atual, a cada clique. Legal que isso está em outro componente diferente do Cart, lembra, estamos no Header, onde podemos mostrar a quantidade de produtos no carrinho de compras.

Então todo componente que tem o **connect** ouvindo os reducers, e quando um reducer altera o componente é recriado com o novo valor do reducer, lembrando que no React a cada alteração no estado é chamado o ***render\*** que recria todo o componente do zero.

Mas no Header eu não preciso do carrinho todo, apenas da quantidade de itens no array de carrinho.

Então eu mudo um pouco:

```text
export  default  connect(state  => ({
	cartSize: state.cart.length,
}))(Header);
```

Em vez de pegar o carrinho todo, pego apenas o tamanho do carrinho e adiciono na prop `cartSize`.

```react
...
function  Header({ cartSize }) {
console.log(cartSize);
...
<div>
	<strong>Meu carrinho</strong>
	<span>{cartSize} items</span>
</div>
...
}
```

E utilizo a variável `cartSize` para mostrar a quantidade de itens no carrinho, e agora a cada alteração no carrinho, adicionando ou removendo itens do carrinho, o **Header** é recriado com o novo valor de quantidade total no carrinho.

Então vamos recapitular sintetizando tudo que aconteceu:

Tudo começou na index.js da Home, a gente conectou a Home com redux, usando o **connect**, e quando fazemos isso, o componente tem acesso ao `dispatch` nas `props`.

**dispatch** serve para disparar as actions do redux, actions são responsáveis para dizer ao redux que queremos alterar o estado, e ela carrega um tipo de ação e um valor. O type é obrigatório, e passamos o product para o carrinho.

E no arquivo `reducer.js` ouve todas as actions, e ele ouve e trata a action do type: 'ADD*TO*CART' e recria um estado com o valor que a action traz para ele, que é o produto. A função cart recebe um estado se não tiver nada pode ser um array vazio por padrão, e recebe também as actions.

Quando é feito a alteração, o Redux avisou todos os componentes que possuem o connect, e que estão necessitando do state que está no cart que essa informação foi atualizada, e ele executa novamente a função que está no connect passando para o componente o novo valor do estado que ele precisa renderizar na tela.

Então, nosso componente dispara uma action, a action avisa o reducer, o reducer faz as alterações, o redux avisa todos os componentes que possuem connect, e os componentes se atualizem com essa alteração.

## Aula 12 - Reactotron + Redux

Vamos configurar o Reactotron para mostrar os logs do Redux, vai ajudar bastante a debugar o Redux.

Para fazer isso na web em projetos com React e Redux adicionaremos as libs:

```shell
yarn add reactotron-react-js reactotron-redux
```

Depois criamos um arquivo `src/config/Reactotron.js` que conterá as configurações.

Embora ela seja uma ferramenta de fazer debug ela tem que ser instalada com dependência do projeto mesmo, pois o código de configuração dela é executado em produção também, porém o efetivo funcionamento dela só funciona em desenvolvimento, você vai ver agora porque:

```react
import Reactotron from 'reactotron-react-js';
import { reactotronRedux } from 'reactotron-redux';

if (process.env.NODE_ENV === 'development') {
  const tron = Reactotron.configure()
    .use(reactotronRedux())
    .connect();

  tron.clear();

  console.tron = tron;
}
```

Importamos as duas libs instaladas, o Create React App (CRA) coloca nas variáveis de ambiente, o NODE_ENV, se estiver em ambiente de desenvolvimento o valor é 'development' se não é 'production' quando o site está no ar. E se estiver em desenvolvimento, então a configuração é realizada.

Declaramos uma variável tron que recebe a configuração do reactotron o qual usa um plugin do redux para logar as actions e reducers.

Eu limpo o console toda vez que a aplicação reinicia e atribui a configuração de tron para a variável global console na propriedade tron também. Com isso não preciso importar o tron para usar o reactotron em todos os arquivos que quero logar, basta invocar console.tron.log('meu log').

Depois só importar a configuração na raiz da aplicação, que pode ser na primeira linha do componente App.js:

```react
import  './config/ReactotronConfig';
...
```

e por fim agora podemos usar o `console.tron.log('loguei');`.

Apenas com isso não é possível logar as actions e reducers, precisamos configurar no store:

```react
import { createStore } from 'redux';
import rootReducer from './models/rootReducer';

const enhancer =
  process.env.NODE_ENV === 'development' ? console.tron.createEnhancer() : null;

const store = createStore(rootReducer, enhancer);

export default store;
```

Criaremos um middleware no redux que intercepta as chamadas das actions pra os reducers, então com isso o reactotron irá logar todas as actions para nós.

Portanto, se o enhancer estiver com valor de configuração, ele é passado para o createStore e a integração é realizada com sucesso!

Nesse caso nem precisa de colocar um console.tron.log(''), só com a integração o Reactotron vai ouvir as actions através do middleware.

## Aula 13 - Listando no carrinho

Agora vamos listar os produtos do carrinho na listagem de produtos no carrinho. Para fazer isso vamos conectar o carrinho `cart/index.js` nosso componente Cart com o Redux, portanto, vamos importar connect e exportar o Cart com esse high order function envolvendo-o.

Única coisa nova aqui é que vamos criar uma função e passar para dentro do connect a referência dela.

Quando usamos o connect do Redux, por convenção nós criamos uma função chamada `mapStateToProps` e passamos a referência como primeiro parâmetro da função `connect(mapStateToProps)`.

`mapStateToProps`: mapear o estado do reducer para uma prop do componente.

```react
...
const  mapStateToProps  =  state  => ({
	cart: state.cart,
});

export  default  connect(mapStateToProps)(Cart);
```

A partir de agora continua a mesma coisa, o componente Cart terá uma prop `cart` com o estado do reducer cart que está na store do Redux.

E agora podemos alimentar o componente Cart com esses valores.

```react
import React from 'react';
import { connect } from 'react-redux';
import {
  MdRemoveCircleOutline,
  MdAddCircleOutline,
  MdDelete,
} from 'react-icons/md';
import { Container, ProductTable, Total } from './styles';

function Cart({ cart }) {
  return (
    <Container>
      <ProductTable>
        <thead>
          <tr>
            <th />
            <th>PRODUTO</th>
            <th>QTD</th>
            <th>SUBTOTAL</th>
            <th />
          </tr>
        </thead>
        <tbody>
          {cart.map(product => (
            <tr>
              <td>
                <img src={product.image} alt={product.title} />
              </td>
              <td>
                <strong>{product.title}</strong>
                <span>{product.price}</span>
              </td>
              <td>
                <div>
                  <button type="button">
                    <MdRemoveCircleOutline size={20} color="#7169c1" />
                  </button>
                  <input type="number" readOnly value={1} />
                  <button type="button">
                    <MdAddCircleOutline size={20} color="#7169c1" />
                  </button>
                </div>
              </td>
              <td>
                <strong>R$ 258,80</strong>
              </td>
              <td>
                <div>
                  <button type="button">
                    <MdDelete size={20} color="#7169c1" />
                  </button>
                </div>
              </td>
            </tr>
          ))}
        </tbody>
      </ProductTable>

      <footer>
        <button type="button">Finalizar Pedido</button>

        <Total>
          <span>TOTAL</span>
          <strong>R$ 1.920,28</strong>
        </Total>
      </footer>
    </Container>
  );
}

const mapStateToProps = state => ({
  cart: state.cart,
});

export default connect(mapStateToProps)(Cart);
```

Falta fazer funcionar a atualização da quantidade, remover item do carrinho, e deixar o preço formatado e calcular o valor total.

Para isso vamos alterar o reducer do carrinho: `cart/reducer.js`.

Primeiro vamos colocar uma propriedade de quantidade do produto no carrinho, que é o `amount`.

```react
export default function cart(state = [], action) {
  switch (action.type) {
    case 'ADD_TO_CART':
      return [...state, { ...action.product, amount: 1 }];
    default:
      return state;
  }
}
```

Pronto, toda vez que for incluído o produto, vai ser criado um estado com os dados do produto e o valor do amount será 1.

Na próxima aula iremos resolver o problema de produto duplicado. Pois o comportamento esperado é atualizar a quantidade de produto e não duplicar o mesmo produto quando adicionamos ele mais de uma vez no carrinho.

## Aula 14 - Produto duplicado

Quando o usuário adicionar o mesmo produto no carrinho, vamos somar a quantidade em vez de duplicar.

Vamos utilizar uma lib [immerjs](https://github.com/immerjs/immer) para lidar com objetos e arrays que são imutáveis.

```shell
yarn add immer
```

Com o immer nós importamos a função `produce` do `immer`, e essa função recebe o estado (state) atual e um rascunho (draftState) que podemos fazer qualquer coisa, programar sem utilizar os principios de imutabilidade, podemos fazer push no array, setar valor na mão mesmo, conforme o exemplo abaixo, ele vai pegar o rascunho e fazer as alterações do jeito certo (imutável) e disponibilizar no `nextState`.

> The basic idea is that you will apply all your changes to a temporary *draftState*, which is a proxy of the *currentState*. Once all your mutations are completed, Immer will produce the *nextState* based on the mutations to the draft state. This means that you can interact with your data by simply modifying it while keeping all the benefits of immutable data.

Exemplo:

```react
import produce from "immer"

const baseState = [
    {
        todo: "Learn typescript",
        done: true
    },
    {
        todo: "Try immer",
        done: false
    }
]

const nextState = produce(baseState, draftState => {
    draftState.push({todo: "Tweet about it"})
    draftState[1].done = true
})
```

Vamos ver na prática no nosso projeto.

Precisamos agora retornar o resultado próximo estado que o produce irá retornar, ele recebeu o state atual e o draft que é uma cópia do estado, com isso fizemos um verificação no array de carrinho para verificar se o produto já estava inserido, retornando a posição dele no array.

Se tem o produto, o productIndex recebe o id dele.

Verifico se é maior que zero, isso é, ele se ele tiver valor maior que zero então ele achou o produto.

Portanto ele altera o valor do amount acrescentando mais um ao valor.

Se não tivesse o produto, ele colocaria amount igual a 1 mesmo e das próximas vezes que adicionasse iria só incrementando esse valor.

```react
import produce from 'immer';

export default function cart(state = [], action) {
  switch (action.type) {
    case 'ADD_TO_CART':
      return produce(state, draft => {
        const productIndex = draft.findIndex(p => p.id === action.product.id);
        if (productIndex >= 0) {
          draft[productIndex].amount += 1;
        } else {
          draft.push({ ...action.product, amount: 1 });
        }
      });
    default:
      return state;
  }
}
```

Pronto, agora é só testar! O produto não duplica, mas aumenta o valor da quantidade.

## Aula 15 - Remover produto

Quando o usuário clicar no botão com ícone de lixeira para remover produto vamos remover do array do reducer do cart.

O carrinho já está conectado com o Redux, então só passar a função dispatch para disparar uma action para deletar o produto do carrinho informando o id do produto.

```react
...
function  Cart({ cart, dispatch }) { ... }
...
<MdDelete
   size={20}
   color="#7169c1"
   onClick={() => dispatch({ type: 'REMOVE_FROM_CART', id: product.id })}
/>
```

Pronto, agora a action vai ser disparada pelo dispatch, agora só falta fazer o reducer tratar essa ação, pois ouvir ele já está ouvindo. Pode testar no reactotron e ver que o log já aparece.

Para isso no cart/reducer.js:

```react
case 'REMOVE_FROM_CART':
  return produce(state, draft => {
    const productIndex = draft.findIndex(p => p.id === action.id);
    if (productIndex >= 0) {
      draft.splice(productIndex, 1);
    }
 });
```

Busco a posição do produto o produto pelo id, se tiver presente no array, removo um item do array usando slice, informando a posição e quantos itens a partir da posição, no nosso caso apenas 1, o próprio produto.

Pronto, agora é só testar.

## Aula 16 - Refatorando as actions

Uma prática muito legal é separar as actions em um único arquivo por funcionalidade, e não deixar elas espalhadas no código como estão, e também criar uma variável onde armazenaremos o nome da action, para evitar erros de digitação e facilitar a manunteção dos nomes dessas actions.

Para fazer isso, vamos criar um arquivo `actions.js` dentro da pasta `store/models/cart`:

```react
export function addToCart(product) {
  return {
    type: 'ADD_TO_CART',
    product,
  };
}

export function removeFromCart(id) {
  return {
    type: 'REMOVE_FROM_CART',
    id,
  };
}
```

Agora vamos usar essas funções nos componentes das telas Home e no Cart.

```react
import  *  as CartActions from  '../../store/models/cart/actions';

 handleAddProduct = product => {
    const { dispatch } = this.props;

    dispatch(CartActions.addToCart(product));
  };
```

Pronto, dessa forma já podemos continuar adicionando produtos ao carrinho, mas podemos mudar um pouco mais para melhorar o código.

Importamos o bindActionCreators do Redux:

```react
import { bindActionCreators } from  'redux';
```

E assim como criamos o mapStateToProps, vamos criar o mapDispatchToProps:

```react
const mapDispatchToProps = dispatch =>
  bindActionCreators(CartActions, dispatch);
```

ele recebe o dispatch do redux, e o bindActionCreators faz a combinação de actions. Agora além de podermos ter o state nas props teremos também as actions.

E passamos como segundo parâmetro da função connect a função `mapDispatchToProps`.

Foi setado `null` na primeira posição pois esse componente não lida com estado.

```react
export default connect(
  null,
  mapDispatchToProps
)(Home);
```

Pronto! Agora está funcionando, e o mesmo passo a passo vou fazer no carrinho também.

Legal também manter as actions com o nome da funcionalidade + ação, então vou alterar:

```react
export function addToCart(product) {
  return {
    type: '@cart/ADD',
    product,
  };
}

export function removeFromCart(id) {
  return {
    type: '@cart/REMOVE',
    id,
  };
}
```

E alterei também no reducer para ouvir corretamente, veja detalhes no código.

## Aula 17 - Alterando quantidade

Vamos adicionar ou remover a quantidade de itens do produto.

Primeiro criamos uma action para atualizar a quantidade:

```react
export function updateAmount(id, amount) {
  return {
    type: '@cart/UPDATE_AMOUNT',
    id,
    amount,
  };
}
```

A action recebe o id do produto e a quantidade que deverá ser atualizada.

Depois conectamos essa função no componente do Carrinho:

```react
import React from 'react';
import { connect } from 'react-redux';
import {
  MdRemoveCircleOutline,
  MdAddCircleOutline,
  MdDelete,
} from 'react-icons/md';
import { bindActionCreators } from 'redux';
import { Container, ProductTable, Total } from './styles';
import * as CartActions from '../../store/models/cart/actions';

function Cart({ cart, removeFromCart, updateAmount }) {
  function increment(product) {
    updateAmount(product.id, product.amount + 1);
  }
  function decrement(product) {
    updateAmount(product.id, product.amount - 1);
  }

  return (
    <Container>
      <ProductTable>
        <thead>
          <tr>
            <th />
            <th>PRODUTO</th>
            <th>QTD</th>
            <th>SUBTOTAL</th>
            <th />
          </tr>
        </thead>
        <tbody>
          {cart.map(product => (
            <tr>
              <td>
                <img src={product.image} alt={product.title} />
              </td>
              <td>
                <strong>{product.title}</strong>
                <span>{product.price}</span>
              </td>
              <td>
                <div>
                  <button type="button">
                    <MdRemoveCircleOutline
                      size={20}
                      color="#7169c1"
                      onClick={() => decrement(product)}
                    />
                  </button>
                  <input type="number" readOnly value={product.amount} />
                  <button type="button">
                    <MdAddCircleOutline
                      size={20}
                      color="#7169c1"
                      onClick={() => increment(product)}
                    />
                  </button>
                </div>
              </td>
              <td>
                <strong>R$ 258,80</strong>
              </td>
              <td>
                <div>
                  <button type="button">
                    <MdDelete
                      size={20}
                      color="#7169c1"
                      onClick={() => removeFromCart(product.id)}
                    />
                  </button>
                </div>
              </td>
            </tr>
          ))}
        </tbody>
      </ProductTable>

      <footer>
        <button type="button">Finalizar Pedido</button>

        <Total>
          <span>TOTAL</span>
          <strong>R$ 1.920,28</strong>
        </Total>
      </footer>
    </Container>
  );
}

const mapDispatchToProps = dispatch =>
  bindActionCreators(CartActions, dispatch);

const mapStateToProps = state => ({
  cart: state.cart,
});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Cart);
```

Foi criado duas funções que foram utilizadas nos botões de incrementar e decrementar a quantidade de produto no carrinho:

```react
  function increment(product) {
    updateAmount(product.id, product.amount + 1);
  }
  function decrement(product) {
    updateAmount(product.id, product.amount - 1);
  }
```

Por fim criamos mais um case para tratar a chamada da action de atualizar quantidade:

```react
...
    case '@cart/UPDATE_AMOUNT': {
      if (action.amount <= 0) {
        return state;
      }
      return produce(state, draft => {
        const productIndex = draft.findIndex(p => p.id === action.id);
        if (productIndex >= 0) {
          draft[productIndex].amount = Number(action.amount);
        }
      });
 ...
```

Pronto, só testar.