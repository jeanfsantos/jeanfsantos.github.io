---
title: Fazendo deploy de uma aplicação node.js na Heroku
layout: post
---
E ae galera, hoje vamos fazer deploy da nossa aplicação todo-list na Heroku quem quiser conferir clica [aqui](https://todo-list-javascript.herokuapp.com).

### Heroku

Primeiro antes de alterar nossa aplicação todo-list precisamos criar uma conta na [Heroku](https://www.heroku.com) e um novo app.

Vamos precisar instalar a ferramenta [Heroku Toolbelt](https://toolbelt.heroku.com): ferramenta CLI para criar e gerenciar aplicativos Heroku.

Depois que instalar o Toolbelt podemos executar o comando heroku para confirmar a instalação e visualizar todos os comandos disponíveis:

<pre>
  <code class="javascript">
  $ heroku
  </code>
</pre>

Precisamos fazer o login para autorizar o envio do nosso aplicativo local para a heroku:

<pre>
  <code class="javascript">
  $ heroku login
  </code>
</pre>

### todo-list

Vamos alterar a propriedade start e adicionar a propriedade main no arquivo `package.json`:

<pre>
  <code class="json">
  {
    "name": "todo-list",
    "main": "index.js",
    "description": "todo list",
    "scripts": {
      "start": "node ."
    },
    "author": "Jean Santos <jeanwfsantos@gmail.com>",
    "dependencies": {
      "http-server": "^0.9.0"
    }
  }
  </code>
</pre>

Vamos criar um arquivo `index.js` na raiz do nosso projeto que será responsável por instanciar o servidor http:

<pre>
  <code class="javascript">
  var httpServer = require('http-server');
  var port = (process.env.PORT || 5000);

  httpServer.createServer().listen(port);
  </code>
</pre>

Podemos testar nossa aplicação com linha de comando e acessar a url [http://127.0.0.1:5000](http://127.0.0.1:5000) ou a porta que voce configurou.

<pre>
  <code class="javascript">
  $ heroku local web
  </code>
</pre>

Se até aqui tudo estiver funcionando, vamos fazer um comite, definir nosso repositório remoto na Heroku e fazer um push.

<pre>
  <code class="javascript">
  $ git commit -m "deploy heroku"
  $ heroku git:remote -a testetttt
  $ git push heroku master
  </code>
</pre>

O nosso ambiente na heroku vai instalar todas dependencias e executar o script start do `package.json`.

Para acessar a nossa aplicação funcionando na heroku podemos acessar a url direto ou através da linha de comando:

<pre>
  <code class="javascript">
  $ heroku open
  </code>
</pre>

Caso ocorre algum erro voce pode executar o comando abaixo e ver o log:

<pre>
  <code class="javascript">
  $ heroku logs -t
  </code>
</pre>

Bom galera qualquer dúvida, crítica ou sugestão pode deixa um comentário ou criar uma issue no [repositório](https://github.com/jeanfsantos/jeanfsantos.github.io).
