---
layout:     post
title:      "MEAN Stack - Parte 2"
subtitle:   "Aprimorando o backend NodeJS + Express"
date:       2015-03-16 23:23:53
author:     "Bruno Milhan"
header-img: "img/home-bg.jpg"
---

Olá;

Este será o segundo post da série sobre **MEAN Stack**, caso você não tenha visto o primeiro poderá encontrá-lo [aqui](http://brunomilhan.com.br/2015/03/14/meanStackIntro/). Desta vez implementaremos mais funcionalidades no backend que utiliza **NodeJS + Express** já criado, logo é altamente recomendável que você de uma olhada no primeiro post. Caso não esteja com tempo suficiente poderá fazer clone do [repositório GitHub](https://github.com/brunomilhan/example-express-node) e executar o comando **npm install** para instalar as dependências, considerando que você já tenha NodeJS instalado em sua máquina.

Uma breve retrospectiva da primeira parte vimos como configurar um backend com NodeJS e Express e implementamos a funcionalidade de servir páginas estáticas, entretanto sabemos que apenas isso não basta para construir uma aplicação.

No final deste post você será capaz de criar views dinâmicas no Express utilizando EJS, verá o que é e como implementar uma rota e aprenderá boas práticas adotando a arquitetura MVC para a aplicação.

<img src="{{ site.baseurl }}/img/nodejs_express.jpg" alt="NodeJS+Express" height="200" width="140">

## EJS - Template Engine ##

Quem já trabalhou com javascript talvez em algum momento tenha se deparado com templates engine, se é algo novo para você não se preocupe, pois não é algo difícil de entender.

**Templates Engine são utilizados para adicionar dados dinâmicos nas views.** Simples? Pois é...

Utilizaremos o Template Engine [EJS](http://www.embeddedjs.com/). O EJS manipula as views através da sintaxe **<%= %>**.

Veremos como funciona na prática. Ainda não temos o EJS instalado, para instalar execute no terminal o seguinte comando dentro da pasta raiz:

    npm install ejs --save

Pronto! Agora você já tem o EJS instalado em seu projeto. Mais uma vez o npm fazendo o trabalho sujo para nós. :)

Falta declarar para o Express que utilizaremos o EJS no projeto, abra o arquivo **/config/express.js** e adicione a seguinte linha abaixo do comentário *environment var*:

```javascript
//environment var
app.set('view engine', 'ejs');
app.set('views', './views');
```

Não existe segredos no código acima, apenas declaramos para o express que estamos utilizando o EJS,  além disso informamos ao express que nossas views estão dentro da pasta views, logo crie uma pasta chamada views.

Vamos criar a view que usará o EJS, faremos algo simples somente para exemplificar, dentro da pasta views crie o arquivo **index.ejs**: 

```html
<!doctype html>
<html>
<head>
        <meta charset="UTF-8">
</head>
<body>
        <h1>Animal desaparecido: <%=nome %></h1>
</body>
</html>
```

Legal! O *nome* será subsistido pelo nome do animal quando a página for renderizada, o backend será responsável por preenche-lo, mas primeiro aprenderemos outro assunto: **Routes**.

## Routes##

Precisamos acessar o arquivo **index.ejs** dentro do diretório views, Como faremos isso? Através de rotas do Express.

Uma rota é uma combinação de [URI](http://pt.wikipedia.org/wiki/URI) e um método request do http. Simples, você verá na prática.

Crie uma pasta chamada **routes**. Por enquanto necessitamos criar apenas uma rota para a view index que acabamos de criar, então crie um arquivo chamado **home.js** dentro de routes e implemente o código abaixo:

```javascript
var controller = require('../controllers/home');

module.exports = function(app) {
	app.get('/', controller.index);
};
```

Não assuste. Vou explicar...

Para configurar uma rota é necessário ter uma instância do Express, não temos ela então o jeito é recebe-la por parâmetro. Se você leu o primeiro post com atenção lembrará que temos a instância do express dentro do arquivo **/config/express.js**, lembre-se disso no final voltaremos nesse assunto.

Note que utilizamos a função get do express no código acima, não há nada a temer, se trata do função **get** do http, o que fizemos foi declarar para o Express que para as requisições a partir de / efetuaremos uma determinada ação, como eu disse no inicio para configurar uma rota combinamos um **URI + método http**. 

Você já deve ter percebido que o controller será o responsável pelo **método http**. Isso é uma boa prática, pois estamos separando as responsabilidades da aplicação, implantaremos o app baseado na [arquitetura MVC](http://pt.wikipedia.org/wiki/MVC) e o controller é o responsavel por ligar a view ao model, porém ainda não temos esse arquivo, o próximo passo será cria-lo.

Crie uma pasta chamada **controllers** e um arquivo chamado **home.js**, implemente um array que guardará todas as ações deste controller, lembre-se que teremos que acessar esse método dentro de outro arquivo, então utilize o **module.exports**:

```javascript
module.exports = function() {
	var controller = {};
	//ações do controller...

	return controller;
}
```

Precisamos renderizar a view index que criamos, além disso, lembra do template ejs? Precisamos também preenche-lo, faremos isso *hardcode* desta vez, através da associação **chave : valor**. Posteriormente utilizaremos o **MongoDB** para preencher essas informações, mas este é assunto para outros posts. Implemente uma função para representar essa ação como um item do array controller:

```javascript
...
//ações
controller.index = function(req, res) {
	res.render('index', {nome: 'Gato Louco'});
};
...
```

Perceba que a ação é uma requisição http, logo, temos dois parâmetros **req** e **res**, neste exemplo apenas a '**res** posta' que enviaremos ao usuário importa, no caso acima enviamos a view index renderizando o template ejs através da função **render**.

### Últimos passos... :) ###

Você deve ter percebido que está quase tudo pronto, o que fizemos até aqui foi:

 1. Instalamos o EJS;
 2. Configuramos o Template Engine no Express;
 3. Criamos uma view com uma tag ejs para ser renderizada;
 4. Criamos uma Rota;
 5. Adotamos boas práticas e criamos um controller para a ação de renderizar a view.

O 6º e último passo é 'avisar ao express' que implementamos as etapas acima. Como? Fácil! Lembra que disse no começo que precisamos de uma instância do Express para a rota funcionar?

Abra o arquivo **/config/express.js** e adicione a dependência da rota criada através da função **require**, em seguida abaixo do comentário *middlewares* chame a função enviando app, **home(app)**. Seu arquivo express.js deverá ficar desta maneira:

```javascript
var express = require('express');
var home = require('../routes/home');

module.exports = function() {
    var app = express();

    // environment vars
    app.set('port', 3000);
    app.set('view engine', 'ejs');
    app.set('views', './views');

    // middlewares
    //app.use(express.static('./public'));
    home(app);

    return app;
};
```

### Pronto! :) ###
Inicie o servidor através do comando **node server** dentro da pasta raiz, você deverá ver que o Gato Louco está desaparecido. hehe

Talvez você esteja decepcionado por não ter implementado alguma funcionalidade da ideia de recuperar animais de estimação desaparecidos. 

Pensei melhor sobre o assunto e dividirei essa série de posts em duas etapas, primeiro tentarei ensinar e depois aplicarei os estudos a um problema. Cheguei a esta conclusão pois minha maior dificuldade quando leio tutoriais na internet é que as vezes os exemplos são tão aplicados que fica difícil extrair algum conhecimento.

Para cumprir com minha palavra no próximo post colocarei em prática os estudos destes dois posts construindo um backend para o problema proposto! :)

Obrigado!