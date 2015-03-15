---
layout:     post
title:      "Introdução a MEAN Stack"
subtitle:   "Construindo um servidor de páginas estáticas com NodeJS + Express"
date:       2015-03-14 23:11:48
author:     "Bruno Milhan"
header-img: "img/home-bg.jpg"
---

Olá;

Neste primeiro post sobre MEAN Stack veremos o básico necessário para construirmos um backend simples utilizando o [Node.js](https://nodejs.org/)  e o framework [Express](http://expressjs.com/). No final deste post você será capaz de criar um servidor web de páginas estáticas.

Sempre que possível postarei um novo tópico aprofundando neste assunto, o objetivo desta série de posts é apresetar a stack desenvolvendo uma aplicação para encontrar animais de estimação perdidos.

<a href="#">
    <img src="{{ site.baseurl }}/img/shutup-meme.jpg" alt="meme">
</a>
Vamos ao que interessa! :)


## Node.js ##

 Nada melhor para apresentar o Node que a própria documentação encontrada em seu website:

> "Node.js® is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices." - [Node.js](https://nodejs.org/) 

Trabalhei com o nodejs na Unify desenvolvendo o projeto Ansible ([Circuit](https://www.yourcircuit.com/)), por experiência posso dizer que ele é um backend muito poderoso e flexível ainda mais quando utilizado juntamente com o [Angular](https://angularjs.org/) para construir um [SPA](http://en.wikipedia.org/wiki/Single-page_application).

###Instalando o NodeJS###

Para instalar o node basta acessar o website [nodejs.org](https://nodejs.org/) e clicar em Install. Fácil! O próprio website se encarregará de baixar a versão correta para sua plataforma. Após a instalação abra o terminal e verifique se a instalação foi efetuado com sucesso através do comando:

    node -v

Caso esteja interessado em realizar um helloworld com o nodejs:

```javascript
var http = require('http');
http.createServer(function (req,res){
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Hello world nodejs');
}).listen(3000, '127.0.0.1');
```

Este pequeno trecho de código não tem muito segredo, mas já é interessante para observarmos como node funciona.
Primeiramente declaramos que iremos utilizar o módulo http do node e posteriormente utilizamos essa variável para criar um simples servidor de http que escutará em localhost na porta 3000.

Para testar acesse http://localhost:3000 de sua maquina, se a instalação foi efetuada com sucesso e você verá a mensagem *Hello world nodejs*.

Apesar do node por si só já possuir a capacidade de servir o conteúdo web, veremos adiante nos próximos posts que ele é muito limitado e para suprir esse problema utilizaremos o famework Express.

##Express##

Vamos ver o que o website do Express diz a respeito de si:

> "Fast, unopinionated, minimalist web framework for Node.js" -[Express](http://expressjs.com/)
>
**Web Applications**
Express is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.
>
**APIs**
With a myriad of HTTP utility methods and middleware at your disposal, creating a robust API is quick and easy.
>
**Performance**
Express provides a thin layer of fundamental web application features, without obscuring Node features that you know and love.

Preciso dizer algo a mais?  ...Vamos ao que interessa. :)

### Instalando o Express ###

O Express será o framework web que servirá o conteúdo web da aplicação, ele trabalha com middlewares, que serão responsáveis por lidar com as requisições.

Antes de instalar o express, que não é uma tarefa difícil, vamos criar a estrutura padrão do projeto, seu projeto deve conter as seguintes pastas:

    /app
     |public
     |config   
     

Posteriormente quando estivermos trabalhando com páginas dinâmicas adicionaremos mais pastas para estruturar a aplicação, nesta etapa inicial essas duas pastas já irão satisfazer as necessidades. 

Já temos uma simples estrutura, vamos instalar o express! Existem duas formas para fazer isso, o modo mais prático é executar o comando **npm init** dentro da pasta raiz do projeto. Ele se encarregará de criar o arquivo **package.json** que será o responsável pela configuração da aplicação e suas dependências. Algumas simples perguntas serão feitas, basta responde-las ou pressionar enter até o final.

    npm init

**Mas afinal o que é npm?**

NPM é o gerenciador de pacotes padrão do node que foi instalado automaticamente quando instalamos o node. Através da linha de comando ele busca pacotes em [npmjs.com](https://www.npmjs.com/) e nos auxilia a instalar as dependências da aplicação de uma maneira bastante simples.
Não existe muitos segredos quanto ao npm, com o tempo você verá o quanto ele simplifica o processo de desenvolvimento. :D

Agora que já temos o arquivo de configuração vamos executar o comando:

    npm install express --save

A opção **--save** faz com que a dependência seja adicionada no package.json, caso queira migrar o projeto para outra maquina basta rodar o comando **npm install** que todos os módulos necessários serão instalados na maquina.

Além disso podemos notar que foi criada a pasta **node_modules** no diretório da aplicação, é nesta pasta que ficarão todos os módulos necessários para a aplicação funcionar.

Com o express instalado vamos configurá-lo, para isso criaremos o arquivo **express.js** dentro da pasta **/config** e implementaremos o seguinte código:

```javascript
var express = require('express');

module.exports = function() {
        var app = express();

        // environment var
        app.set('port', 3000);

        // middleware
        app.use(express.static('./public'));

        return app;
};
```

O que fizemos acima foi instanciar o módulo express e depois utilizamos um recurso interessante que é o **module.exports** este será responsavel por retornar uma instância do express sempre que necessário.
Em nodejs tudo que utilizar module.exports ficará visível em todo escopo da aplicação.

Podemos notar também o uso de **app.set(*chave, valor*)**  que é utilizado para armazenar variáveis de ambiente no formato chave - valor que poderão ser recuperadas através da função **app.get(*valor*)**. 

Temos também o **app.use()** que tem uma utilização um pouco mais abrangente, neste exemplo estamos utilizando apenas para criar um middleware que sirvirá páginas estáticas através de **express.static(*./local*)**. Futuramente veremos que ele também servirá para efetuar as operações do http.

Já temos uma função que retornará uma instancia do express, vamos utiliza-la em **server.js** que criamos no inicio deste artigo, caso você já tenha criado basta copia-lo para a pasta raiz do projeto, entretanto precisamos realizar algumas alterações para informar ao node que estamos utilizando o express, o arquivo **server.js** ficará assim:


```javascript
var http = require('http');
var app = require('./config/express')();

http.createServer(app).listen(app.get('port'), function(){
        console.log('Express running... On port: '+app.get('port'));
});
```

O que fizemos basicamente foi passar a *var app* que tem uma instância do express como argumento da função createServer do http que recebe um listener e depois passamos a porta declarada na variável ambiente do express para o listen através da função **app.get('port')**.

Vamos criar um simples arquivo html dentro da pasta public para testar o servidor de páginas web estáticas.

```html
<!doctype html>
<html>
<head>
        <meta charset="UTF-8">
</head>
<body>
        <h1>Hello world Express</h1>
</body>
</html>
```

Pronto! :) Agora execute o comando **node server** dentro da pasta raiz da aplicação e tente acessar o endereço **http://localhost:3000** se você seguiu todos os passos descritos aqui, o navegador deverá apresentar a mensagem *Hello world Express*.

Observou como é simples criar um servidor http? :)

Você pode ver este exemplo completo no repositório do [GitHub](https://github.com/brunomilhan/example-express-node), futuramente postarei toda a aplicação neste reposotório. 

Quando penso no empenho que era configurar um servidor apache-tomcat ao trabalhar com java+jsf, configurar a IDE, instalar as dependências... E se precisar mudar de maquina? Com certeza terá um grande trabalho... Velhos tempos... :/

O node e seu poderoso gerenciador de pacotes **npm** se encarregá de fazer todo trabalho sujo para nós! Pensando dessa maneira, a cada estudo que faço sobre essas ferramentas, me afasto cada vez mais das ferramentas tradicionais.

No próximo post adicionaremos mais funcionalidades ao backend, respondendo com conteúdo dinâmico, veremos um pouco de views, template engine (ejs), routes...

Agradeço a você leitor, não sou um bom escritor, mas tentarei sempre ser o mais didático possível, foi uma grande experiência poder compartilhar esse básico sobre node e express, gostei dessa ideia de escrever e com certeza escreverei algo sempre que sobrar um tempo.

Até mais!