<template>
  <li :class="{completed: todo.completed, editing: todo == editedTodo}">
    <div class="view">
      <input class="toggle" type="checkbox" @click="completeTodo(todo)" v-model="todo.completed">
      <label @dblclick="editTodo(todo)">{{todo.text}}</label>
      <button class="destroy" @click="removeTodo(todo)"></button>
    </div>
    <input class="edit" type="text" v-model="todo.text" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
  </li>
</template>


<script type = "text/javascript" >
import axios from 'axios';
import ENDPOINT from '../constants';

export default {
  props: ['todo'],
  components: {
  },
  data() {
    return {
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
    };
  },
  methods: {
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
      axios.put(`${ENDPOINT}/${todo.todo_id}`, todo)
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

    completeTodo(todo) {
      console.log('completeTodo');
      todo.completed = !todo.completed;
      console.log(todo.completed);
      axios.put(`${ENDPOINT}/${todo.todo_id}`, todo)
        .then((results) => {
          console.log(results.data.body);
          todo = results.data.body;
        })
        .catch((e) => {
          this.errors.push(e);
        });
    },

    removeTodo(todo) {
      this.$emit('remove-todo', todo);
    },


  },
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
</style>
