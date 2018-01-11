<template>
  <div id="app">
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    </header>
    <section class="main" v-show="todos.length">
      <input class="toggle-all" type="checkbox" v-model="allDone">
      <ul class="todo-list">
        <Todo class="todo" :todo="todo" v-for="todo in filteredTodos" :key="todo.updatedAt" :class="{completed: todo.completed, editing: todo == editedTodo}">
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
      api: 'https://grdao5tka1.execute-api.eu-central-1.amazonaws.com/dev/todos',
    };
  },
  created() {
    this.fetch();
  },

  // watch todos change for localStorage persistence
  watch: {
    todos: {
      deep: true,
      handler(todos) {
        this.save(todos);
      },
    },
  },

  // computed properties
  // http://vuejs.org/guide/computed.html
  computed: {
    filteredTodos() {
      return filters[this.visibility](this.todos);
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
      axios.get(this.api)
        .then((response) => {
          this.todos = response.data.items;
        })
        .catch((e) => {
          this.errors.push(e);
        });
    },
    save(todos) {
      localStorage.setItem(this.STORAGE_KEY, JSON.stringify(todos));
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
      console.log(this.newTodo);
      axios.post(this.api, data)
        .then((results) => {
          this.todos.push(results.data);
        })
        .catch((e) => {
          this.errors.push(e);
        });
      this.newTodo = '';
    },

    removeTodo(todo) {
      const index = this.todos.indexOf(todo);
      this.todos.splice(index, 1);
      axios.delete(`${this.api}/${todo.todo_id}`)
        .then()
        .catch((e) => {
          this.errors.push(e);
        });
    },

    editTodo(todo) {
      this.beforeEditCache = todo.text;
      this.editedTodo = todo;
    },

    doneEdit(todo) {
      if (!this.editedTodo) {
        return;
      }
      this.editedTodo = null;
      todo.text = todo.text.trim();
      axios.put(`${this.api}/${todo.todo_id}`, todo)
        .then()
        .catch((e) => {
          this.errors.push(e);
        });
      if (!todo.text) {
        this.removeTodo(todo);
      }
    },

    cancelEdit(todo) {
      this.editedTodo = null;
      todo.text = this.beforeEditCache;
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

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
<style lang="scss">
@import 'assets/base.css';
@import 'assets/index.css';
</style>
