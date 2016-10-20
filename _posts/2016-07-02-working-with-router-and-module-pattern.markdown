---
title: Trabalhando com router e module pattern
layout: post
---
E ae galera, hoje vamos adicionar rotas e modularizar todo nosso código do todo-list, criaremos uma classe para manipular as rotas, quem quiser ver como ficou clica [aqui](https://github.com/jeanfsantos/jeanfsantos.github.io).

Para começar separaremos a nossa aplicação em dois, uma para listagem das tarefas e outra para cadastro.

O nosso layout principal `index.html` possui dois links `#/index` e `#/add` essas `hash` serão utilizadas para navegar e carregar o conteúdo dinâmico na `div[data-js="main"]`.

<pre>
  <code class="html">
  &lt;div class="container"&gt;
    &lt;div class="row"&gt;
      &lt;div class="col-xs-12"&gt;
        &lt;h1&gt;Todo List&lt;/h1&gt;
      &lt;/div&gt;
    &lt;/div&gt;
    &lt;div class="row"&gt;
      &lt;div class="col-md-12"&gt;
        &lt;ul class="nav"&gt;
          &lt;li class="nav-item"&gt;
            &lt;a class="nav-link" href="#/index"&gt;List&lt;/a&gt;
          &lt;/li&gt;
          &lt;li class="nav-item"&gt;
            &lt;a class="nav-link" href="#/add"&gt;Add&lt;/a&gt;
          &lt;/li&gt;
        &lt;/ul&gt;
      &lt;/div&gt;
    &lt;/div&gt;
    &lt;div class="row"&gt;
      &lt;div class="col-md-12"&gt;
        &lt;div data-js="main"&gt;&lt;/div&gt;
      &lt;/div&gt;
    &lt;/div&gt;
  &lt;/div&gt;
  &lt;script src="src/js/route.js"&gt;&lt;/script&gt;
  &lt;script src="src/js/todo.js"&gt;&lt;/script&gt;
  &lt;script src="src/partials/index/index.js"&gt;&lt;/script&gt;
  &lt;script src="src/partials/add/index.js"&gt;&lt;/script&gt;
  </code>
</pre>

Quando instanciamos a classe `Route` passamos 3 parâmetros:

- `hash` que utilizaremos para identificar a página;
- `template` que passamos o caminho do nosso html;
- `fn` a função que executará nosso script da página;

<pre>
  <code class="javascript">
  Route('index', '/src/partials/index/index.html', app);
  </code>
</pre>

Após instanciar a classe `Route` executa dois ouvintes: `load` e `hashchange` os dois executam o mesmo método, que manipulará nossas rotas, verifica se possui `location.hash` caso não encontre atribuímos o `hash` da nossa primeira view e retornamos. Neste momento o ouvinte `hashchange` é executado e dessa vez o encontramos uma `location.hash` e verifica se conhecemos este `hash` caso conhecemos verificamos se o conteúdo está em cache e renderizamos o html ou fazemos uma requisição com ajax armazenamos o retorno no cache e renderizamos e for fim executamos a função do terceiro parâmetro.

<pre>
  <code class="javascript">
  (function(window, $){
    'use strict';

    function Route(hash, template, fn){
      if(!(this instanceof Route))
        return new Route(hash, template, fn);
      this.hash = hash;
      this.template = template;
      this.fn = fn;
      this.cache = null;
      this.initEvents();
    }

    Route.prototype.initEvents = function initEvents(){
      window.addEventListener(
        'load',
        this.handleRoute.bind(this),
        false
      );
      window.addEventListener(
        'hashchange',
        this.handleRoute.bind(this),
        false
      );
    };

    Route.prototype.handleRoute = function handleRoute(e){
      if(!location.hash){
        location.hash = '#/index';
        return;
      }
      if(location.hash.substring(2) !== this.hash)
        return;
      if(this.cache){
        this.renderHTML(this.cache);
        return;
      }
      this.handleAjax(this.template);
    };

    Route.prototype.handleAjax = function handleAjax(file){
      this.ajax = new XMLHttpRequest();
      this.ajax.open('GET', file);
      this.ajax.send();
      this.ajax.addEventListener(
        'readystatechange',
        this.handleReadyStateChange.bind(this),
        false
      );
    };

    Route.prototype.handleReadyStateChange =
      function handleReadyStateChange(){
      if(!this.isRequestOk())
        return;
      this.cache = this.ajax.responseText;
      this.renderHTML(this.ajax.responseText);
    };

    Route.prototype.isRequestOk = function isRequestOk(){
      return this.ajax.readyState === 4 && this.ajax.status === 200;
    };

    Route.prototype.renderHTML = function renderHTML(html){
      var $main = $.querySelector('[data-js="main"]');
      $main.innerHTML = html;
      this.fn();
    };

    window.Route = Route;
  })(window, document)
  </code>
</pre>

A classe `Todo` funciona como um serviço ela é responsável somente por manipular nossas tarefas nela foram implementados os métodos: `create`, `insert`, `read`, `update`, `remove` e `removeAll`.

<pre>
  <code class="javascript">
  (function(window){
    'use strict';

    function Todo(){
      if(!(this instanceof Todo))
        return new Todo();
      this.create();
    }

    Todo.prototype.create = function create(){
      this._todo = [];
      this._id = 1;
    };

    Todo.prototype.insert = function insert(obj){
      this._todo.push(Object.assign({}, {id: this._id}, obj));
      this._id++;
    };

    Todo.prototype.read = function read(){
      return this._todo;
    };

    Todo.prototype.update = function update(id, obj){
      this._todo.forEach(function(item){
        if(item.id === id){
          item.task = obj.task;
          item.status = obj.status;
        }
      });
    };

    Todo.prototype.remove = function remove(id){
      var _this = this;
      this._todo.forEach(function(item, index){
        if(item.id === id)
          _this._todo.splice(index, 1);
      });
    };

    Todo.prototype.removeAll = function removeAll(){
      this._todo = [];
    };

    window.Todo = Todo();
  })(window);
  </code>
</pre>

As páginas de listagem e cadastro que antes estavam juntas foram separadas e elas fazem o uso do serviço `Todo` portanto elas só precisam se preocupar na lógica de suas páginas.

public/src/partials/index/index.html:

<pre>
  <code class="html">
  &lt;div class="card"&gt;
    &lt;div class="card-block"&gt;
      &lt;table class="table table-bordered"&gt;
        &lt;thead&gt;
          &lt;tr&gt;
            &lt;th class="text-xs-center"&gt;#&lt;/th&gt;
            &lt;th class="text-xs-center"&gt;task&lt;/th&gt;
            &lt;th class="text-xs-center"&gt;status&lt;/th&gt;
            &lt;th class="text-xs-center"&gt;delete&lt;/th&gt;
          &lt;/tr&gt;
        &lt;/thead&gt;
        &lt;tbody data-js="tbody"&gt;
        &lt;/tbody&gt;
      &lt;/table&gt;
    &lt;/div&gt;
  &lt;/div&gt;
  </code>
</pre>

src/partials/index/index.js:

<pre>
  <code class="javascript">
  (function(Route, Todo, $){
    'use strict';

    Route('index', '/src/partials/index/index.html', app);

    function app(){
      /* ... */

      function handleContentTable(){
        $tbody.innerHTML = null;
        Todo.read().forEach(function(item){
          var frag = $.createDocumentFragment();
          var tr = $.createElement('tr');
          tr.appendChild(createCell(item.id));
          tr.appendChild(createCell(item.task));
          tr.appendChild(createCheckbox(item));
          tr.appendChild(createButtonRemove(item));
          frag.appendChild(tr);
          $tbody.appendChild(frag);
        });
        tableEvents();
      }

      /* ... */

      function updateTodo(){
        Todo.update(Number(this.getAttribute('data-id')), {
          task: this.getAttribute('data-task'),
          status: changeStatus.call(this)
        });
      }

      /* ... */

      function removeItem(){
        Todo.remove(Number(this.getAttribute('data-id')));
        $tbody.dispatchEvent(eventUpdate);
      }
    }
  })(Route, Todo, document);
  </code>
</pre>

src/partials/add/index.html:

<pre>
  <code class="html">
  &lt;div class="card"&gt;
    &lt;div class="card-block"&gt;
      &lt;form data-js="form"&gt;
        &lt;div class="form-group"&gt;
          &lt;label&gt;task&lt;/label&gt;
          &lt;input type="text" class="form-control" data-js="input-task"&gt;
        &lt;/div&gt;
        &lt;div class="clearfix"&gt;
          &lt;div class="pull-xs-right"&gt;
            &lt;button type="button" class="btn btn-danger"
              data-js="btn-remove-all"&gt;Remove All&lt;/button&gt;
            &lt;button type="submit" class="btn btn-primary"&gt;Add&lt;/button&gt;
          &lt;/div&gt;
        &lt;/div&gt;
      &lt;/form&gt;
    &lt;/div&gt;
  &lt;/div&gt;
  </code>
</pre>

src/partials/add/index.js:

<pre>
  <code class="javascript">
  (function(Route, Todo, $){
    'use strict';

    Route('add', '/src/partials/add/index.html', app);

    function app(){
      /* ... */

      function handleForm(e){
        e.preventDefault();
        var $inputTask = $.querySelector('[data-js="input-task"]');
        Todo.insert({
          task: $inputTask.value,
          status: 0
        });
        $inputTask.value = '';
      }

      function removeAll(){
        Todo.removeAll();
      }
    }
  })(Route, Todo, document);

  </code>
</pre>

Bom galera qualquer dúvida, crítica ou sugestão pode deixa um comentário ou criar uma issue no [repositório](https://github.com/jeanfsantos/jeanfsantos.github.io).
