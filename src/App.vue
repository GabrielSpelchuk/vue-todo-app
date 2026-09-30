<script setup>
import { computed, ref, onMounted } from "vue";
import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from "./components/TodoItem.vue";
import Message from "./components/Message.vue";
import * as todoApi from "./api/todos";

const todos = ref([]);
const title = ref("");
const errorMessage = ref("");
const status = ref("all");

onMounted(async () => {
  try {
    todos.value = await todoApi.getTodos();
  } catch (e) {
    errorMessage.value = "Unable to load todos";
  }
});

const activeTodos = computed(() => {
  return todos.value.filter((todo) => !todo.completed);
});

const visibleTodos = computed(() => {
  if (status.value === "active") {
    return activeTodos.value;
  }

  if (status.value === "completed") {
    return todos.value.filter((todo) => todo.completed);
  }

  return todos.value;
});

async function addTodo() {
  if (!title.value) {
    errorMessage.value = "Title should not be empty";

    return;
  }

  try {
    const newTodo = await todoApi.createTodo(title.value);

    todos.value.push(newTodo);
    title.value = "";
  } catch (e) {
    errorMessage.value = "Unable to add a todo";
  }
}

const deleteTodo = async (todoId) => {
  try {
    await todoApi.deleteTodo(todoId);

    todos.value = todos.value.filter((todo) => todo.id !== todoId);
  } catch (e) {
    errorMessage.value = "Unable to delete a todo";
  }
};

const updateTodo = async ({ id, title, completed }) => {
  try {
    const updatedTodo = await todoApi.updateTodo({ id, title, completed });
    const currentTodo = todos.value.find((todo) => todo.id === id);

    Object.assign(currentTodo, updatedTodo);
  } catch (e) {
    errorMessage.value = "Unable to update a todo";
  }
};
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button
          type="button"
          v-if="todos.length > 0"
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
        ></button>

        <form @submit.prevent="addTodo">
          <input
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
            @input="errorMessage = ''"
          />
        </form>
      </header>

      <TransitionGroup
        tag="section"
        name="todolist"
        class="todoapp__main"
        v-if="todos.length > 0"
      >
        <TodoItem
          v-for="todo in visibleTodos"
          :key="todo.id"
          :todo="todo"
          @delete="deleteTodo(todo.id)"
          @update="updateTodo($event)"
        />
      </TransitionGroup>

      <footer class="todoapp__footer">
        <span class="todo-count" data-cy="TodosCounter">
          {{ activeTodos.length }} items left
        </span>

        <StatusFilter v-model="status" />

        <button
          type="button"
          class="todoapp__clear-completed"
          :disabled="todos.length === activeTodos.length"
          @click="todos = activeTodos"
        >
          Clear completed
        </button>
      </footer>
    </div>

    <Message class="is-warning" :hidden="!errorMessage" @close="errorMessage = ''">
      <template #header>
        <p>Server Error</p>
      </template>

      <template #default>
        <p>{{ errorMessage }}</p>
      </template>
    </Message>
  </div>
</template>

<style scoped>
.todolist-enter-active,
.todolist-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.todolist-enter-from,
.todolist-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
