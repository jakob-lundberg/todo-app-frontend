<template>
  <div id="app" class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    </header>
    <section class="main" v-show="todos.length">
      <input class="toggle-all" type="checkbox" v-model="allDone">
      <ul class="todo-list">
        <Todo class="todo"
              :class="{completed: todo.completed, editing: todo == editedTodo}"
              :key="todo.updatedAt"
              :todo="todo"
              v-for="todo in filteredTodos"
              v-on:remove-todo='removeTodo'
              >
        </Todo>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length">
      <span class="todo-count">
        <strong v-text="remaining"></strong> {{pluralize('item', remaining)}} left
      </span>
      <ul class="filters">
        <li><a href="#/all"
               :class="{selected: visibility == 'all'}"
               @click="visibility = 'all'"
               >All</a></li>
        <li><a href="#/active"
               :class="{selected: visibility == 'active'}"
               @click="visibility = 'active'"
               >Active</a></li>
        <li><a href="#/completed"
               :class="{selected: visibility == 'completed'}"
               @click="visibility = 'completed'"
               >Completed</a></li>
      </ul>
    </footer>
  </div>
</template>

<script>

import axios from 'axios';
import Todo from './components/Todo';
import ENDPOINT from './constants';


const filters = {
  all(todos) {
    return todos;
  },
  active(todos) {
    return todos.filter(todo => !todo.completed);
  },
  completed(todos) {
    return todos.filter(todo => todo.completed);
  },
};

export default {
  name: 'app',
  components: {
    Todo,
  },
  data() {
    return {
      todos: [],
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
    };
  },
  created() {
    this.fetch();
  },

  // computed properties
  // http://vuejs.org/guide/computed.html
  computed: {
    filteredTodos() {
      let items = this.todos;
      items = filters[this.visibility](items);
      items = items.sort((a, b) => new Date(a.createdAt) > new Date(b.createdAt));
      return items;
    },
    remaining() {
      return filters.active(this.todos).length;
    },
    allDone: {
      get() {
        return this.remaining === 0;
      },
      set(value) {
        this.todos.forEach(
          (todo) => { todo.completed = value; },
        );
      },
    },
  },

  methods: {
    fetch() {
      axios.get(ENDPOINT)
        .then((response) => {
          this.todos = response.data.items;
        })
        .catch((e) => {
          this.errors.push(e);
        });
    },
    pluralize(word, count) {
      return word + (count === 1 ? '' : 's');
    },

    addTodo() {
      const value = this.newTodo && this.newTodo.trim();
      const data = { text: value, completed: false };
      if (!value) {
        return;
      }
      axios.post(ENDPOINT, data)
        .then((results) => {
          this.todos.push(results.data);
        })
        .catch((e) => {
          this.errors.push(e);
        });
      this.newTodo = '';
    },
    removeTodo(todo) {
      console.log('removing todo');
      const index = this.todos.indexOf(todo);
      this.todos.splice(index, 1);
      axios.delete(`${ENDPOINT}/${todo.todo_id}`)
        .then()
        .catch((e) => {
          this.errors.push(e);
        });
    },


  },

  // a custom directive to wait for the DOM to be updated
  // before focusing on the input field.
  // http://vuejs.org/guide/custom-directive.html
  directives: {
    'todo-focus': function focus(el, binding) {
      if (binding.value) {
        el.focus();
      }
    },
  },
};
</script>

<style lang="scss">
@import 'assets/base.css';
@import 'assets/index.css';
</style>
