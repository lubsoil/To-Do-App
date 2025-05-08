<script setup>
import { ref, computed, onMounted } from 'vue'

let id = 0

const addTodoFormVisible = ref(false)
const newTodoText = ref('')
const newTodoDeadline = ref('')
const selectedFilter = ref('all')
const todos = ref([])

const filteredTodos = computed(() => {
  if (selectedFilter.value === 'active') {
    return todos.value.filter((t) => !t.done)
  } else if (selectedFilter.value === 'completed') {
    return todos.value.filter((t) => t.done)
  }
  return todos.value;
})

function addTodo() {
  todos.value.push({ id: id++, text: newTodoText.value, done: false, deadline: newTodoDeadline.value, editing: false })
  newTodoText.value = ''
  newTodoDeadline.value = ''
  saveTodos();
  addTodoFormVisible.value = false;
}

function editTodo(todo) {
  todo.editing = !todo.editing;
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t !== todo)
  saveTodos();
}

function toogleFilter() {
  if (selectedFilter.value === 'all') {
    selectedFilter.value = 'active';
  } else if (selectedFilter.value == 'active') {
    selectedFilter.value = 'completed';
  } else if (selectedFilter.value == 'completed') {
    selectedFilter.value = 'all';
  }
}

function loadTodos() {
  let todosString = localStorage.getItem("todos");
  let todosObject = JSON.parse(todosString);
  if (todosObject != null) {
    todos.value = todosObject;
    if (todosObject.length > 0) {
      id = todosObject[todosObject.length - 1].id + 1;
    }
  }
}

function saveTodos() {
  const todosString = JSON.stringify(todos.value)
  localStorage.setItem("todos", todosString)
}

onMounted(() => {
  loadTodos();
})
</script>

<template>
  <div class="main">
    <div class="title">
      To-Do List
    </div>


    <div>
      <h4>Your tasks:</h4>
      <div class="filters">
        Filters:
        <a @click="selectedFilter = 'all'" :class="selectedFilter === 'all' ? 'button button-active' : 'button'">All</a>
        <a @click="selectedFilter = 'active'"
          :class="selectedFilter === 'active' ? 'button button-active' : 'button'">Active</a>
        <a @click="selectedFilter = 'completed'"
          :class="selectedFilter === 'completed' ? 'button button-active' : 'button'">Completed</a>
      </div>
      <ul>
        <li v-for="todo in filteredTodos" :key="todo.id" class="">
          <div v-if="!todo.editing" class="todo-element">
            <div>
              <input type="checkbox" :checked="todo.done" @click="todo.done = !todo.done; saveTodos();" >
              <span :class="{ done: todo.done }">{{ todo.text + (todo.deadline != "" && todo.deadline != null ? " (Deadline: "+todo.deadline+")" : "") }}</span>
            </div>
            <div>
              <button @click="editTodo(todo);" class="button button-save">Edit</button>
              <button @click="removeTodo(todo)" class="button button-cancel">Delete</button>
            </div>
          </div>
          <div v-else class="form">
            <label>Task Name</label> <input type="text" v-model="todo.text">
            <label>Deadline</label> <input type="date" v-model="todo.deadline">
            <button class="button button-save" @click="editTodo(todo); saveTodos();">Save</button>
          </div>
        </li>
        <h3 v-if="filteredTodos.length == 0">There are no available tasks</h3>
      </ul>
    </div>
    <div>
      <a v-show="!addTodoFormVisible" @click="addTodoFormVisible = !addTodoFormVisible" class="button button-save">Add
        Task</a>
      <div v-show="addTodoFormVisible">
        <form @submit.prevent="addTodo();" class="form">
          <label for="add-text">Task Name</label> <input v-model="newTodoText" name="add-text" type="text" required
            placeholder="Task Name">
          <label for="add-deadline">Deadline</label> <input type="date" name="add-deadline" v-model="newTodoDeadline">
          <button class="button button-save">Add Task</button>
          <button class="button button-cancel" @click="addTodoFormVisible = !addTodoFormVisible">Cancel</button>
        </form>
      </div>
    </div>
  </div>
</template>