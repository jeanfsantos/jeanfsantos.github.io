---
title: A simple todo list using javascript es5
layout: post
---
E ae galera, hoje quero mostra como construir um simples todo list usando somente javascript sem precisar de um framework angular, react, e outros.

Se quiser ver como ficou segue o link: [todo list](http://jeanfsantos.github.io/todo-list-javascript/public/).

Utilizei o modulo `http-server` para levantar o server e começa a trabalha.

<pre>
  <code class="javascript">
    http-server public -p 3000
  </code>
</pre>

Uma boa prática e colocar seu código dentro de uma IIFE e utilizar o use strict, dessa forma você evita de poluir o escopo global.

<pre>
  <code class="javascript">
    (function($){
      'use strict';
      ...
    })(document)
  </code>
</pre>


No começo do código foi declarada 4 variáveis:

`_todo` armazena uma lista de objetos que são construidos com id(numérico), task(string) e status(boolean);

`_id` armazenar o índice que incrementa conforme adiciona item a lista;

`updateToDo` é um evento update que faço dispach toda vez que preciso atualizar a tabela.

`$tbody` seletor do tbody.

<pre>
  <code class="javascript">
    var _todo = [];
    var _id = 1;
    var updateToDo = new CustomEvent('update');
    var $tbody = $.querySelector('[data-js="tbody"]');
  </code>
</pre>

A primeira função que executa é o `init` que executa dois listeners:

O primeiro registra a espera do evento submit do form e executa a função `handleForm`.

O segundo registra a espera do evento update e executa a função `handleContentTable`.

<pre>
  <code class="javascript">
    function init(){
      var $form = $.querySelector('[data-js="form"]');
      $form.addEventListener('submit', handleForm, false);
      $tbody.addEventListener('update', handleContentTable, false);
    }
  </code>
</pre>

A função `handleForm` adiciona um novo item na lista, limpa o input e dispara o evento update.

<pre>
  <code class="javascript">
    function handleForm(e){
      e.preventDefault();
      var $inputTask = $.querySelector('[data-js="input-task"]');
      var todo = {
        id: _id,
        task: $inputTask.value,
        status: false
      };
      _todo.push(todo);
      _id++;
      $inputTask.value = '';
      $tbody.dispatchEvent(updateToDo);
    }
  </code>
</pre>

Quando a função `handleContentTable` é executado, e feito um limpo no conteúdo do tbody, e em seguida cria as tds com ajuda das funções `createCell`, `createCheckbox`, `createButtonRemove`, que são incluídas em uma tr e depois no tbody. E no final chama a função `tableEvents` que registra a espera dos eventos `changeStatus` e `removeItem`.

<pre>
  <code class="javascript">
    function handleContentTable(){
      $tbody.innerHTML = null;
      _todo.forEach(function(item){
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
  </code>
</pre>

As funções `createCell`, `createCheckbox`, `createButtonRemove` todas elas retorna um elemento td com seu conteúdo.

`createCell` cria uma célula e adiciona um texto.

<pre>
  <code class="javascript">
    function createCell(text){
      var td = $.createElement('td');
      td.classList.add('text-xs-center');
      td.textContent = text;
      return td;
    }
  </code>
</pre>

`createCheckbox` cria uma célula e adiciona um elemento input checkbox.

<pre>
  <code class="javascript">
    function createCheckbox(item){
      var td = $.createElement('td');
      var label = $.createElement('label');
      var input = $.createElement('input');
      td.classList.add('text-xs-center');
      input.setAttribute('type', 'checkbox');
      input.setAttribute('data-js', 'input-checkbox');
      input.setAttribute('value', item.id);
      if(item.status)
        input.setAttribute('checked', 'checked');
      label.appendChild(input);
      td.appendChild(label);
      return td;
    }
  </code>
</pre>

`createButtonRemove` cria uma célula e adiciona um botão remover.

<pre>
  <code class="javascript">
    function createButtonRemove(item){
      var td = $.createElement('td');
      var button = $.createElement('button');
      var span = $.createElement('span');
      td.classList.add('text-xs-center');
      button.setAttribute('class', 'btn btn-danger btn-sm');
      button.setAttribute('data-js', 'btn-remove');
      button.setAttribute('data-value', item.id);
      span.innerHTML = '&times';
      span.setAttribute('aria-hidden', 'true');
      button.appendChild(span);
      td.appendChild(button);
      return td;
    }
  </code>
</pre>

`tableEvents` executa duas funções `changeStatus` e `removeItem`.

<pre>
  <code class="javascript">
    function tableEvents(){
      Array.prototype.forEach.call(
        $.querySelectorAll('[data-js="input-checkbox"]'),
        function(item){
          item.removeEventListener('click', changeStatus, false);
          item.addEventListener('click', changeStatus, false);
        }
      );

      Array.prototype.forEach.call(
        $.querySelectorAll('[data-js="btn-remove"]'),
        function(item){
          item.removeEventListener('click', removeItem, false);
          item.addEventListener('click', removeItem, false);
        }
      );
    }
  </code>
</pre>

`changeStatus` é executado toda vez que marca um checkbox. Ele encontra o item marcado na lista e altera o status.

<pre>
  <code class="javascript">
    function changeStatus(){
      var _this = this;
      _todo.forEach(function(item, index){
        if(item.id === Number(_this.getAttribute('value')))
          item.status = !item.status;
      });
    }
  </code>
</pre>

`removeItem` é executado quando clica no botão remover. Ele remove o objeto da lista e dispara o evento update.

<pre>
  <code class="javascript">
    function removeItem(){
      if(!confirm('do you want delete?'))
        return;
      var _this = this;
      _todo.forEach(function(item, index){
        if(item.id === Number(_this.getAttribute('data-value')))
          delete _todo[index];
      });
      $tbody.dispatchEvent(updateToDo);
    }
  </code>
</pre>

Bom galera esse foi meu primeiro post. Qualquer dúvida, crítica ou sugestão pode deixa um comentário ou criar uma issue no [repositório](https://github.com/jeanfsantos/jeanfsantos.github.io).
