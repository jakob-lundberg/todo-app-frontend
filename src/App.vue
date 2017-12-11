<template>
  <div id="app">
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    </header>
    <section class="main" v-show="todos.length">
      <input class="toggle-all" type="checkbox" v-model="allDone">
      <ul class="todo-list">
        <li class="todo" v-for="todo in filteredTodos" :key="todo.id" :class="{completed: todo.completed, editing: todo == editedTodo}">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{todo.title}}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input class="edit" type="text" v-model="todo.title" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
        </li>
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
      <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
        Clear completed
      </button>
    </footer>
  </div>
</template>

<script>

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
  },
  data() {
    return {
      todos: this.fetch(),
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
    };
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
      return JSON.parse(localStorage.getItem(this.STORAGE_KEY) || '[]');
    },
    save(todos) {
      localStorage.setItem(this.STORAGE_KEY, JSON.stringify(todos));
    },
    pluralize(word, count) {
      return word + (count === 1 ? '' : 's');
    },

    addTodo() {
      const value = this.newTodo && this.newTodo.trim();
      if (!value) {
        return;
      }
      this.todos.push({ id: this.todos.length + 1, title: value, completed: false });
      this.newTodo = '';
    },

    removeTodo(todo) {
      const index = this.todos.indexOf(todo);
      this.todos.splice(index, 1);
    },

    editTodo(todo) {
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },

    doneEdit(todo) {
      if (!this.editedTodo) {
        return;
      }
      this.editedTodo = null;
      todo.title = todo.title.trim();
      if (!todo.title) {
        this.removeTodo(todo);
      }
    },

    cancelEdit(todo) {
      this.editedTodo = null;
      todo.title = this.beforeEditCache;
    },

    removeCompleted() {
      this.todos = filters.active(this.todos);
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
