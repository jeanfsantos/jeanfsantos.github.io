---
title: Persisting the data in localStorage
layout: post
---
E ae galera, hoje quero mostrar como persistir dados no localStorage dando continuidade no [todo list](http://jeanfsantos.github.io/todo-list-javascript/public/) que implementamos.

Criei duas funções para manipular a API localStorage. Um método setter e outro getter.

Como estou utilizando uma lista de objetos preciso converter para string e gravar os dados. E para recuperar preciso parsear.

<pre>
  <code class="javascript">
    function setStorage(key, value){
      return localStorage.setItem(key, JSON.stringify(value));
    }

    function getStorage(key){
      return JSON.parse(localStorage.getItem(key));
    }
  </code>
</pre>

Na declaração das variáveis `_todo` e `_id` foi alterado para recuperar esses dados do localStorage e caso não tenha atribuímos os valores padrões.

<pre>
  <code class="javascript">
    var _todo = getStorage('todo') || [];
    var _id = getStorage('id') || 1;
  </code>
</pre>

E toda vez que atualizo alguns desses dados preciso atualizar o localStorage.

<pre>
  <code class="javascript">
    setStorage('todo', _todo);
    setStorage('id', _id);
  </code>
</pre>

E por fim criei um botão para limpar o localStorage todo e atualizar a tabela.

<pre>
  <code class="javascript">
    function init(){
      /* ... */
      var $btnRemoveAll = $.querySelector('[data-js="btn-remove-all"]');
      $btnRemoveAll.addEventListener('click', removeAll, false);
      /* ... */
    }

    function removeAll(){
      _todo = [];
      localStorage.removeItem('todo');
      $tbody.dispatchEvent(updateToDo);
    }
  </code>
</pre>

Para quem quiser saber mais sobre essa API segue o link: [Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage)

Bom galera qualquer dúvida, crítica ou sugestão pode deixa um comentário ou criar uma issue no [repositório](https://github.com/jeanfsantos/jeanfsantos.github.io).
