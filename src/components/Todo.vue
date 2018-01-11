<template>
  <li :class="{completed: todo.completed, editing: todo == editedTodo}">
    <div class="view">
      <input class="toggle" type="checkbox" v-model="todo.completed">
      <label @dblclick="editTodo(todo)">{{todo.text}}</label>
      <button class="destroy" @click="removeTodo(todo)"></button>
    </div>
    <input class="edit" type="text" v-model="todo.text" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
  </li>
</template>


<script type = "text/javascript" >

export default {
  props: ['todo'],
  components: {
  },
  data() {
    return {
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
      api: 'https://grdao5tka1.execute-api.eu-central-1.amazonaws.com/dev/todos',
    };
  },
  methods: {
    editTodo(todo) {
      this.beforeEditCache = todo.text;
      this.editedTodo = todo;
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
