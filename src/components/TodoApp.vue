<script setup lang="ts">
import { ref, shallowRef, computed, onMounted, onBeforeUnmount } from 'vue'
import type { Ref } from 'vue'

// Identify Todo Type
interface Todo {
  id: number
  text: string
  completed: boolean
}

const todos: Ref<Todo[]> = ref([])
const todoInput: Ref<string> = ref('')
const editedTodo: Ref<Todo | null> = shallowRef(null);
const cate: Ref<string> = ref('all')

const filter: {[key: string]: (todos: Todo[]) => Todo[]} = {
  all: (todos: Todo[]): Todo[] => todos,
  active: (todos: Todo[]): Todo[] => todos.filter((todo) => !todo.completed),
  completed: (todos: Todo[]): Todo[] => todos.filter((todo) => todo.completed)
}

const filteredTodos = computed<Todo[]>(() => filter[cate.value](todos.value))
const remaining = computed<number>(() => filter.active(todos.value).length)

function addTodo(e: KeyboardEvent): void {
  const target = e.target as HTMLInputElement
  if (target && target.value) {
    const value: string = target.value.trim()
    const newTodo: Todo = {
      id: Date.now(),
      text: value,
      completed: false
    }
    todos.value.push(newTodo)
    todoInput.value = ''
  }
}

function removeTodo(todo: Todo): void {
  // todos.value = todos.value.filter(v => v.id !== todo.id);
  todos.value.splice(todos.value.indexOf(todo), 1); // Performance better!
}

let beforeEditCache: string = '';
function editTodo(todo: Todo): void {
  beforeEditCache = todo.text;
  editedTodo.value = todo;
}

function doneTodo(todo: Todo): void {
  if (editedTodo.value) {
    editedTodo.value = null;
    todo.text = todo.text.trim();
    if (!todo.text) removeTodo(todo);
  }
}

function cancelTodo(todo: Todo): void {
  editedTodo.value = null;
  todo.text = beforeEditCache;
}

function editInputMounted({ el }: { el: HTMLElement }): void {
  el.focus();
}

function removeCompleted(): void {
  todos.value = filter.active(todos.value)
}

function toggleAll(e: Event): void {
  const target = e.target as HTMLInputElement;
  todos.value.forEach((todo) => todo.completed = target.checked)
}

function onHashChange(): void {
  const route = window.location.hash.replace(/#\/?/, '');
  if (filter[route]) {
    cate.value = route;
  } else {
    window.location.hash = ''
    cate.value = 'all'
  }
}

onMounted(() => {
  window.addEventListener('hashchange', onHashChange)
  onHashChange()
})

onBeforeUnmount(() => {
  window.removeEventListener('hashchange', onHashChange)
})

</script>

<template>
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autofocus
        v-model="todoInput"
        @keyup.enter="addTodo"
      />
    </header>
    <section class="main">
      <input id="toggle-all" class="toggle-all" type="checkbox"
        @click="toggleAll"
        :checked="!remaining"
      />
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
          class="todo"
          v-for="todo in filteredTodos"
          :key="todo.id"
          :class="{
            'completed': todo.completed,
            'editing': editedTodo === todo
          }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed" />
            <label @dblclick="editTodo(todo)">{{ todo.text }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input class="edit" type="text"
            v-if="editedTodo === todo"
            v-model="todo.text"
            @vue:mounted="editInputMounted"
            @blur="doneTodo(todo)"
            @keyup.enter="doneTodo(todo)"
            @keyup.escape="cancelTodo(todo)"
          />
        </li>
      </ul>
    </section>
    <footer class="footer">
      <span class="todo-count" v-if="!!remaining">{{ remaining }} item left</span>
      <ul class="filters">
        <li>
          <a href="#/" :class="{ 'selected': cate === 'all' }">All</a>
        </li>
        <li>
          <a href="#/active" :class="{ 'selected': cate === 'active' }">Active</a>
        </li>
        <li>
          <a href="#/completed" :class="{ 'selected': cate === 'completed' }">Completed</a>
        </li>
      </ul>
      <button class="clear-completed" @click="removeCompleted">Clear completed</button>
    </footer>
  </section>
</template>
