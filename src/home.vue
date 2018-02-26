<template>
<div>
  <section class="todoapp" v-cloak>
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    </header>
    <section class="main" v-show="todos.length">
      <input class="toggle-all" type="checkbox" v-model="allDone">
      <ul class="todo-list">
        <li class="todo" v-for="todo in filteredTodos" :key="todo.id" :class="{completed: todo.completed, editing: todo === editedTodo}">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{todo.title}}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input class="edit" type="text" v-model="todo.title" v-todo-focus="todo === editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length">
      <span class="todo-count">
        <strong v-text="remaining"></strong> {{ pluralize('item', remaining) }} left
      </span>
      <ul class="filters">
        <li><a @click="visibility = 'all'" :class="{selected: visibility === 'all'}">All</a></li>
        <li><a @click="visibility = 'active'" :class="{selected: visibility === 'active'}">Active</a></li>
        <li><a @click="visibility = 'completed'" :class="{selected: visibility === 'completed'}">Completed</a></li>
      </ul>
      <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <p>Based on <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>
</div>
</template>

<script>
const STORAGE_KEY = 'todos-vuejs';

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
  name: 'todos',
  info: {
    path: '/',
  },
  head: {
    title: 'Basys • TodoMVC',
  },

  data() {
    return {
      todos: JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'),
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
    };
  },

  watch: {
    todos: {
      deep: true,
      handler(todos) {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
      },
    },
  },

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
        this.todos.forEach(todo => {
          todo.completed = value;
        });
      },
    },
  },

  methods: {
    pluralize(word, count) {
      return word + (count === 1 ? '' : 's');
    },

    addTodo() {
      const title = this.newTodo && this.newTodo.trim();
      if (!title) return;
      const id = this.todos.length > 0 ? Math.max.apply(Math, this.todos.map(todo => todo.id)) + 1 : 1;
      this.todos.push({id, title, completed: false});
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
      if (!this.editedTodo) return;
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

  // A custom directive to wait for the DOM to be updated before focusing on the input field
  directives: {
    'todo-focus'(el, binding) {
      if (binding.value) el.focus();
    },
  },
};
</script>

<style>
[v-cloak] { display: none; }
.filters li { cursor: pointer; }
</style>