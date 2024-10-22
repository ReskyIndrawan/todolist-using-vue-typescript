<script setup lang="ts">
import { ref, onMounted, computed, watch } from "vue";

// Definisikan tipe untuk todo item
interface Todo {
  content: string;
  category: string;
  done: boolean;
  createdAt: number;
}

const todos = ref<Todo[]>([]);
const name = ref<string>("");

const input_content = ref<string>("");
const input_category = ref<string | null>(null);

const todos_asc = computed((): Todo[] =>
  todos.value.slice().sort((a, b) => {
    return b.createdAt - a.createdAt;
  })
);

const addTodo = (): void => {
  if (
    input_content.value.trim() === "" ||
    input_category.value === null
  ) {
    return;
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date().getTime(),
  });

  // Reset form inputs
  input_category.value = null;
  input_content.value = "";
};

const removeTodo = (todo: Todo): void => {
  todos.value = todos.value.filter((t) => t !== todo);
};

watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});

onMounted((): void => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos") || "[]");
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up
        <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE TODO</h3>

      <form @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input
          type="text"
          placeholder="e.g. make a video"
          v-model="input_content" />

        <h4>Pick a category</h4>

        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="business"
              v-model="input_category" />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              value="personal"
              v-model="input_category" />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>
        <input type="submit" value="add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <div class="list">
        <div
          v-for="todo in todos_asc"
          :class="`todo-item ${todo.done && 'done'}`">
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">
              Delete
            </button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
